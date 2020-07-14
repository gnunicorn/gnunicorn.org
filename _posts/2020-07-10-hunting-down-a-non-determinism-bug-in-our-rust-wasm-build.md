---
layout: post
title: "Hunting down a non-determinism-bug in our Rust Wasm build"
category:
- tech
tags:
  - rust
image: "https://res.cloudinary.com/practicaldev/image/fetch/s--WdUM_SfK--/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://dev-to-uploads.s3.amazonaws.com/i/jkkxjur9ex8egrpachcm.jpg"
---

_[This is a cross-post from dev.to, where I published this first](https://dev.to/gnunicorn/hunting-down-a-non-determinism-bug-in-our-rust-wasm-build-4fk1)_

> _Note: Together with a few colleagues, I will be hosting [an AMA on the Rust subreddit](https://www.reddit.com/r/rust/) Wednesday, July 15th (next week). Come join us, if you have questions on this or any other part of our code base or how we handle things at [Parity Technologies](https://www.parity.io/)._

We recently learned that the WebAssembly build in our system isn't deterministic any longer. This is a short summary of what we did to chase down the bug in the hope that this helps others facing similar issues, give some help and guidance on what to try or how this kind of thing works.


## A bit of background

### Substrate / our stack
At [Parity](https://parity.io) we are building software to run the next generation of the Web: a trust-less secure Web 3.0. Most notably we are building the [Polkadot Network](https://polkadot.network/) based on [Substrate](https://substrate.dev/). Substrate is natively compatible with Polkadot, thus making it simple to secure your blockchain and communicate with Polkadot’s network. We have built everything using Rust which means no legacy codebase and it's open source. 

#### Wasm Runtimes
A key architecture feature of Substrate is that the chain-specific state transition function (STF) is separated from the rest of the node. The binary WebAssembly (wasm) blob is stored on chain (at the key`:code`). The separation allows for network-wide upgrades of the state transition function independently from the rest of the client. These events happen through on-chain governance mechanisms. The client utilizes a wasm runtime, executing updated wasm blobs as the community decides to implement an upgrade. These mechanisms allow Substrate-based blockchains to evolve without the complicated aspects of Hard Forks. We call this `forkless-upgrades`, as participants can stay in sync with the network without relying on upgrading their client.

The way we achieve this is through building our the Rust-runtime-code in a separate crate with the `wasm32-unknown-unknown` target into a Wasm blob and store it on chain.

#### Deterministic builds
Wasm is overall a great invention providing a thin multi-plattform abstraction over the machine language target. Thus the wasm output from a compiler can easily be translated into the actual machine code needed for execution. However, WebAssemblz is not human readable, which makes it hard to audit for the people who should vote upon the chain upgrade. Upgrading a blockchain state transition function can't be taken back, so voting on an opaque block is not particularly instilling confidence in its users.

The way we approach this problem is through "deterministic builds". The rust compiler itself does not, as of now, guarantee that builds are deterministic – meaning that compiling the same code twice will yield the same resulting binary. However, Parity has  managed to solve this problem for our code. So _at least_ within the same environment (OS, Compiler, libs installed) it is reasonably deterministic for a set of targets to be consistent and `wasm32-unknown-unknown` is among them. This allows us to produce a docker-image (and its build description) confirming that the Rust source files indeed result in the binary blob proposed as the next chain runtime. Using the Docker image, auditors can look at the source code directly and do not have to wade through the wasm32 bytecode.


----

Whilein the past they did, we recently discovered that two consecutive builds of the runtime code did not yield the same wasm blobs anymore. This was the starting point when we realized that our build process had been broken.

Unfortunately we didn't have yet tooling in place to alert us about the problem, so we didn't know what introduced the bug. However we know when it last worked for sure: 2020-03-21T16:55:27Z . Here is what I did with that information to track down the problem:

## 0. The test

The first step towards the fix was devising a test that reproducibly displays the problem. In our case this was rather simple: build a specific package in the project (`node-runtime`) twice and compare the resulting wasm. We used the SHA256 hash of the compiler output to do that. 
The specific step then are:

```bash
cargo build --release -p node-runtime   # build the first time
sha2sum target/release/wbuild/target/wasm32-unknown-unknown/release/*.wasm > checksum.sha256      # store the hash
cargo clean     # clean up the artifacts
cargo build --release -p node-runtime   # build again
sha2sum -c checksum.sha256    # are the build identical?
```

## 1. Nightly builds

`wasm32-unknown-unknown` is not yet a fully supported target by the Rust Compiler Team (aka Tier 1), so you need the nightly version to use it. As nightly is moving fast, features are added left and right and often times have unexpected side effects affecting determinism. It is not unlikely that upgrading nightly broke our build.

Luckily, all old rust nightly builds are available. With `rustup` and `cargo`, the amazing tooling Rust provides, it is easy to use and compile with any compiler version. So we first installed the old version via:

```
rustup install nightly-2020-03-19 # install nightly
rustup toolchain add nightly-2020-03-19 wasm32-unknown-unknown # the wasm target toolchain for that compiler version
```

This will install the last nightly version and the target toolchain we know to produce deterministic builds (Note: despite the name, it isn't released _every_ night, often the compiler or some of the components fail to build and the version is skipped).

Now when building the crate we just add `+nightly-2020-03-19` to the cargo command to tell it which version to use:

```bash
cargo +nightly-2020-03-19 build --release -p node-runtime   # build the first time
sha2sum target/release/wbuild/target/wasm32-unknown-unknown/release/*.wasm > checksum.sha256      # store the checksum
cargo clean     # clean up the build
cargo +nightly-2020-03-19 build --release -p node-runtime   # build again
sha2sum -c checksum.sha256    # are the build identical?
```

Unfortunately, that wasn't it: The nightly from back then doesn't produce the same wasm on the current version of our code either. But what about on the older version?

**Nightly builds on old version**

Checking out the old version of the code and building it with the old compiler was indeed deterministic. So what changed? The compiler.

Building the old code with the latest compiler also yields a deterministic build. 

So we know it isn't a change in the compiler that was causing the determinism to break. This doesn't mean it isn't a compiler bug, but only that whatever it was, it wasn't introduced on their side but by changes from us. 

## 2. `git bisect`

One of the great things about using popular (among developers) tools is that others add tooling on top that makes your lives easier. A lesser known but super useful of these tools that made it into mainline `git` already a few years ago, is `git bisect`. It helps you identify which changeset introduced a bug by applying a binary search on the commit history.

Essentially you tell `git bisect` which checkout you know to be `git bisect good` and `git bisect bad`, it then checks out the changeset in the middle of the two. You then perform your test and indicate the status via the same `git bisect good` or `git bisect bad` command. It then jumps in the middle between the latest known good and bad one and checks that out. And so you go until `git bisect` tells you `this is the first change with the bug`.

`git bisect run` even allows you to specify a command it should run that either succeeds or fails and lets `git` perform all the steps automatically. (Unfortunately I didn't have that script ready at the time and was actually building through a more complicated rsync-remote-build system – but that it can run that by itself is pretty awesome.).

Git bisect identified [a pretty large PR](https://github.com/paritytech/polkadot/commit/b361171329213ce41c75ef55d93fb952a4f6c034) to be the culprit . "Pretty large" not only because it adds a significant amount of code itself, it is also the first that activated a few features in the default runtime we are testing against, namely `session_historical`, which we already suspected to be related to this issue.

Unfortunately, this also didn't yield that _one line_ that was the cause on our side. It could still be any number of aspects that cause it. One way forward could be to activate one feature after another to move closer and closer to the actual source. But that isn't as easy as it sounds, so we opted for wasm-introspection first.

Git bisect is a powerful tool and it often is crucial to help narrow down the scope of the search for a bug. Like this episode illustrates, it seldom pinpoints a specific line of code by itself.

## 3. Introspecting the Wasm

The wasm output from the rust compiler is a binary blob, but as any modern standard, it has a few introspection features built in. Most importantly, there is a 1-to-1 text-representation – called WAT – that it can the converted back and forth to without problem. And the default wasm toolchain already includes the handy `wasm2wat` tool.

Running `wasm2wat` on the wasm output of the first build and then again on the second build give us a human-parsable representation of the binary. Then we can diff the two wat version to identify what changed between them (omitted some lines marked as `...` for legibility):

```diff
--- node_runtime_1_1.wat	2020-07-06 09:43:09.122043953 +0200
+++ node_runtime_1_2.wat	2020-07-06 09:43:14.178717225 +0200
@@ -791052,10 +791052,10 @@
                                             i32.add
                                             call $_ZN86_$LT$sp_trie..node_header..NodeHeader$u20$as$u20$parity_scale_codec..codec..Encode$GT$9encode_to17h7b42619bd7dc163dE
                                             local.get 15
-                                            br_if 11 (;@9;)
+                                            br_if 12 (;@8;)
                                             i32.const 0
                                             local.set 1
-                                            br 14 (;@6;)
+                                            br 13 (;@7;)
                                           end
                                           local.get 0
                                           i32.const 1
@@ -791092,10 +791092,10 @@
                                           i32.add
                                           call $_ZN86_$LT$sp_trie..node_header..NodeHeader$u20$as$u20$parity_scale_codec..codec..Encode$GT$9encode_to17h7b42619bd7dc163dE
                                           local.get 15
-                                          br_if 11 (;@8;)
+                                          br_if 10 (;@9;)
                                           i32.const 0
                                           local.set 1
-                                          br 12 (;@7;)
+                                          br 13 (;@6;)
                                         end
                                         i32.const 1
                                         local.set 17
@@ -966366,6 +966366,6 @@
   (export "__data_end" (global 1))
   (export "__heap_base" (global 2))
   ...
-  (data (;0;) (i32.const 1048576) "\04\80\e9\1b\80\14\10\00\80\14\10\00")
+  (data (;0;) (i32.const 1048576) "R\8a\11v\80\14\10\00\80\14\10\00")
   (data (;1;) (i32.const 1048592) " \00\10\00\17\00\00\00\ee\02\00\00\05\00\00\00src/liballoc/raw_vec.rs\00\c7\00\10\00F\00\00\00b\01\00\00\13\00\00\00J\00\00\00\04\00\00\00\04\00\00\00K\00\00\00L\00\00\00M\00\00\00a formatting trait implementation returned an error\00J\00\00\00\00\00\00\00\01\00\00\00N\00\00\00\b4\00\10\00\13\00\00\00J\02\00\00\05\00\00\00src/liballoc/fmt.rs/rustc/15812785344d913d779d9738fe3cca8de56f71d5/src/libcore/fmt/mod.
```

The diff is rather short. With just a bit of scrolling through [the great Mozilla Wasm Explainer](https://developer.mozilla.org/en-US/docs/WebAssembly/Understanding_the_text_format) and the [extensive and great wasm documentation by the original working group](https://webassembly.github.io/spec/core/text/index.html), we quickly learn that the first four changes are just labels that might just be having a different numbering because of compiler internals caused by the last change.

That last one is odd, though: It is a global memory address (at location `1048576`), that is filled with differently prefixed values. What makes this particuarly odd is that if we searched for the address-number in the original `wat` file, we find it is marked as `mutable`.

```wat
  (table (;0;) 343 343 funcref)
  (global (;0;) (mut i32) (i32.const 1048576))
  (global (;1;) i32 (i32.const 1284452))
  (global (;2;) i32 (i32.const 1284452))
```

That is pretty weird, considering the context in which we are running this blob. Remember that the blob is stored on-chain, transparent to everyone. For the execution of each block, the wasm memory is reset and a new instance is created. In general, having mutable globals is pointless for the wasm we are producing, because it would only be live for the duration of the execution of a single block. If you want to mutate state from the runtime to be persisted between blocks, you'd have to call into the external database-storage functions. Because of this we generally don't have any mutable `lazy_statics` or alike in our code.

That doesn't mean that having globals is pointless for our wasm. It can be used for compiler optimisation. For example, every time we create a new `Vec`, it is reading the default values from a global address. This is quicker and needs less storage than pasting the values everywhere.

But, what is causing this memory address to be allocated here? Why would it be mutuable and why with a different value for every build?

***Digging deeper***
Well, we had to track down where it is being used. The compiler wouldn't mark it as mutable if it wasn't changed at least once. When searching we find it is being read from 8 times, but then once, we also see it being written to:

```wat

(func $_ZN14pallet_session10historical20ProvingTrie$LT$T$GT$12generate_for17hb9e80633994986a6E (type 2) (param i32 i32)
    (local i32 i32 i64 i64 i32 i32 i32 i32 i32 i32 i64 i64 i32 i32 i32 i32 i32 i32 i32 i32 i32 i32 i32 i32 i32 i32 i32 i32 i32 i32 i32 i32 i32 i32)
    global.get 0
    i32.const 928
    i32.sub
    local.tee 2
    global.set 0
    block  ;; label = @1
      block  ;; label = @2
        block  ;; label = @3
          block  ;; label = @4
            block  ;; label = @5
              block  ;; label = @6
                block  ;; label = @7
                  block  ;; label = @8
                    block  ;; label = @9
                      i32.const 1
                      call $__rust_alloc
                      local.tee 3
                      i32.eqz
                      br_if 0 (;@9;)
                      local.get 3
                      i32.const 0
                      i32.store8
                      i32.const 0
                      i32.const 0
                      i64.load32_u offset=1048576
                      i64.const 6364136223846793005
                      i64.mul
                      local.get 2
                      i32.const 640
                      i32.add
                      i64.extend_i32_u
                      i64.add
                      i64.const 31
                      i64.rotl
                      local.tee 4
                      i64.store32 offset=1048576
                      local.get 2
                      i32.const 8
```

Even without knowing too much WebAssembly ourselves, we have a few interesting hints in here: 

1. the function name is `_ZN14pallet_session10historical20ProvingTrie$LT$T$GT$12generate_for17hb9e80633994986a6E`, [which translates to `ProvingTrie::generate_for` in the `session::historical` module of our code base](https://github.com/paritytech/substrate/blob/802a0d0b0ade796a3b2d4663212518315923fe8a/frame/session/src/historical/mod.rs#L171-L209).
2. the indentation and many labels indicate that this is within some loop of loops.

Looking at the code we can identify three objects that are mutable – but only at that time, none of them is global. Or at least, as far as we can see, because in Wasm it clearly is.

**Finding the source**
Now that we have a precise point of focus, we can begin troubleshooting. Unfortunately, `git` won't help us here. We could trace back changes to that code base, but as it was only activated in the specific change set we've already identified, there is little that will help identify the source of the error.

```rust
impl<T: Trait> ProvingTrie<T> {
	fn generate_for<I>(validators: I) -> Result<Self, &'static str>
		where I: IntoIterator<Item=(T::ValidatorId, T::FullIdentification)>
	{
		let mut db = MemoryDB::default();
		let mut root = Default::default();

		{
			let mut trie = TrieDBMut::new(&mut db, &mut root);
			for (i, (validator, full_id)) in validators.into_iter().enumerate() {
				let i = i as u32;
				let keys = match <SessionModule<T>>::load_keys(&validator) {
					None => continue,
					Some(k) => k,
				};

				let full_id = (validator, full_id);

				// map each key to the owner index.
				for key_id in T::Keys::key_ids() {
					let key = keys.get_raw(*key_id);
					let res = (key_id, key).using_encoded(|k|
						i.using_encoded(|v| trie.insert(k, v))
					);

					let _ = res.map_err(|_| "failed to insert into trie")?;
				}

				// map each owner index to the full identification.
				let _ = i.using_encoded(|k| full_id.using_encoded(|v| trie.insert(k, v)))
					.map_err(|_| "failed to insert into trie")?;
			}
		}

		Ok(ProvingTrie {
			db,
			root,
		})
	}
```

When looking at the code, there are three main objects involved. It is also helpful to have a minimal understanding of the code base. We see two objects, both the `root` and the `MemoryDB` are passed to the trie. This is likely calculating a new trie root when the `trie.insert` (`let _ = i.using_encoded(|k| full_id.using_encoded(|v| trie.insert(k, v)))` is being called – the only thing we can actually see mutatating state here – and then passes the element through to `MemoryDB`.

[So then, how is memory DB implemented](https://github.com/paritytech/trie/blob/95fd3b5d73a147e357fbb49222e5500309c08d56/memory-db/src/lib.rs#L110-L119)?

```rust
pub struct MemoryDB<H, KF, T>
	where
	H: KeyHasher,
	KF: KeyFunction<H>,
{
	data: HashMap<KF::Key, (T, i32)>,
	hashed_null_node: H::Out,
	null_node_data: T,
	_kf: PhantomData<KF>,
}
```

It uses a `HashMap`.

`HashMap`s are tricky in a fixed-memory environment like wasm. I will not go into the details here, but most HashMaps use simplistic fast hashing functions that can cause collisions. This can degrade performance of a HashMap to the point of resulting in a DoS of the whole process. Any decent modern `HashMap` therefore adds some randomness when relying on hashing keys. Rusts `std`-implementation does this for example. However, a source of randomness doesn't exist within a `wasm32-unknown-unknown` environment, making generic (and in particular the `std`) `HashMap`s unsafe for user-controlled input. 

Well, this isn't input users could easily control and use to bomb our hashmap, so that is fine. But the fact that modern implementations require a source of randomness is something to investigate. For a `no_std` like the wasm environment, the `MemoryDB` implementation takes the [great implementation provided by the `hashbrown` crate](https://github.com/paritytech/trie/blob/95fd3b5d73a147e357fbb49222e5500309c08d56/memory-db/src/lib.rs#L39-L43) – at [version 0.6.3. with default features disabled as the `Cargo.toml` reveals](https://github.com/paritytech/trie/blob/memory-db-v0.21.0/memory-db/Cargo.toml#L14).

It is important to note that `default-features` are disabled because otherwise [hashbrown would activate the `compile-time-rng` feature in ahash](https://github.com/rust-lang/hashbrown/blob/4e7acb5aed8ecdd93fc6f4dc9fcf6d9b8cede39d/Cargo.toml#L42). This is the hasher it utilizes internally. If the `default-features` is activated, the [hasher would include `const-random`](https://github.com/tkaitchuck/aHash/blob/6cf0438e39cd429b78faa98c63784946abad0043/Cargo.toml#L29) to generate a random seed for the hasher _at compile time_ for _some_ randomness.

This is similar to the problem we are debugging: a global constant, updated when we add a new key, that is slightly different on every compile.

**Feature leaking**
With the default features deactivated, how could it have snuck into our build still?

Well, looking just one line lower in the `Cargo.toml` reveals the answer. In order to fix a different compatibilty bug the `MemoryDB` attempts to pin the `ahash` crate to a non-broken version. In doing so, it activates the default features. And cargo features are additive, meaning that if one activates a features, all instances of the crate within the build have that feature activated. Thus leaking into our build.

Boom.

### 4. Fixing it

As you might have noticed, I was linking to specific older commits of these crates. On one side to give long-term correct links, secondly because those are the specific versions in use, but also because the problem is already partially address in newer version. 
The latest `ahash` doesn't have the feature as part of the defaults anymore, it needs to be activated explictly. And by [just removing the dependency pin in MemoryDB](https://github.com/paritytech/trie/commit/117f9efcd7dcbf36137977f53760721b8986b433) (which isn't needed anymore) and releasing a new version, the problem is gone here, too.

The test from step 0 is there to prove it. That very same test [is now added as a proper CI-check](https://github.com/paritytech/substrate/pull/6597) that every PR must pass, so we notice early on if we broke it again.

### 5. Down the label-rabbit-hole

But it doesn't pass. The PR fixes the issue, yet the CI-check complains. Running the script directly and diffing the `.wat`'s tells us why:

```diff
--- node_runtime_1_1.wat	2020-07-06 09:43:09.122043953 +0200
+++ node_runtime_1_2.wat	2020-07-06 09:43:14.178717225 +0200
@@ -791052,10 +791052,10 @@
                                             i32.add
                                             call $_ZN86_$LT$sp_trie..node_header..NodeHeader$u20$as$u20$parity_scale_codec..codec..Encode$GT$9encode_to17h7b42619bd7dc163dE
                                             local.get 15
-                                            br_if 11 (;@9;)
+                                            br_if 12 (;@8;)
                                             i32.const 0
                                             local.set 1
-                                            br 14 (;@6;)
+                                            br 13 (;@7;)
                                           end
                                           local.get 0
                                           i32.const 1
@@ -791092,10 +791092,10 @@
                                           i32.add
                                           call $_ZN86_$LT$sp_trie..node_header..NodeHeader$u20$as$u20$parity_scale_codec..codec..Encode$GT$9encode_to17h7b42619bd7dc163dE
                                           local.get 15
-                                          br_if 11 (;@8;)
+                                          br_if 10 (;@9;)
                                           i32.const 0
                                           local.set 1
-                                          br 12 (;@7;)
+                                          br 13 (;@6;)
                                         end
                                         i32.const 1
                                         local.set 17
```

Although our bug is fixed, the label issues are _not_ and are persistent after. Other than we assumed, they are not caused by our fixed bug, they are their own bug.

Darn.

`br` and `br_if` are control flow conditions. This means they are marking the respective blocks stopping point, in addition to where the execution should continue after. Considering that there are no other diffs, the code jump seems to not differ, it just caused by the ordering of these labels by the compiler. Most likely this marking occurs by the method in which the labels are sorted within the compiler's final processing – specifcially, storing items by a non-determinstic collection internally (e.g. hashmap etc).

The most common place were this happens is in the optimization steps. If we build the `wasm/wat` in debug mode (without the `--release` flag) the diff doesn't show any changes. Thus strongly indicating that this was introduced somewhere in the optimization phase. 

### 6. How the sausage is made

Rust, and the rust compiler in particular, is not a full re-implementation of compilers, but is built on a compiler toolchain called LLVM. This allows Rust to focus on implementing its own syntax and features, while leaving most of the heavy lifting to the impressive suite of LLVM compiler tooling. 

`rustc -vV` even tells you as much:
```
$ rustc +nightly -vV
rustc 1.46.0-nightly (8ac1525e0 2020-07-07)
binary: rustc
commit-hash: 8ac1525e091d3db28e67adcbbd6db1e1deaa37fb
commit-date: 2020-07-07
host: x86_64-unknown-linux-gnu
release: 1.46.0-nightly
LLVM version: 10.0
```
LLVM is a modern and very flexible compiler framework with a huge range of supported platforms and compile targets. The way this is achieved, is by compiling your own language into an abstract in-between representation called `llvm-ir` that you then feed to the llvm compiler. This can then be compiled to machine code for the requested target. As a result, Rust is one of the first languages to directly support compiling to wasm. 

In addition, most release-level optimizations don't actually happen within the rust compiler but in the llvm framework. Though llvm is mostly hidden from sight, for cases such as ours, rust allows us to pass through various arguments to get more data to inspect. Swapping `build` with the specific `rustc` command, we can append compiler flags to the compiler after`--`. For our case we were interested in learning about the different steps taken after each optimization run so `-- -C llvm-args=-print-after-all` is added before piping the entire output into a file.

We quickly noticed that the output _is too big to work with, and running it in parallel on multiple threads resulted in a unparsable output. Adding ` -Z no-parallel-llvm` fixes this but our example is still unwieldy to deal with. Fortunately, the random relabeling happens in the same funtion call everytime. Thus isolating the exact call results in a very thin test case for the compiler bug, too.

### 7. Digging into the llvm output

Looking again at the `.wat` and tracing back from the differing output we find it located in
```wat
(func $_ZN7trie_db9triedbmut18TrieDBMut$LT$L$GT$12commit_child17h1a851bf4aa72b1bdE (type 4) (param i32 i32 i32 i32)
  
```
In order to create a reproducible test, we need a version that tells us what the actual inputs use. Here, the compiler internals can help; by passing `-Zsymbol-mangling-version=v0`. As a result, we get a more comprehensive compiler symbol in our `.wat`

``` wat
(func $_RINvNtCs2KrsVm9iLGa_4core3ptr13drop_in_placeINtNtCsfzX9CsLwTkO_7trie_db9triedbmut9TrieDBMutINtCs7x3GksGMAjA_7sp_trie6LayoutNtNtCs42nTExHBz75_10sp_runtime6traits11BlakeTwo256EEECsdbcsBhSPY2w_12node_runtime (type 3) (param i32)
```

which we can then pass to `rustfilt` (`cargo install rustfilt`) to make it human readable again:

```
$ rustfilt _RINvNtCs2KrsVm9iLGa_4core3ptr13drop_in_placeINtNtCsfzX9CsLwTkO_7trie_db9triedbmut9TrieDBMutINtCs7x3GksGMAjA_7sp_trie6LayoutNtNtCs42nTExHBz75_10sp_runtime6traits11BlakeTwo256EEECsdbcsBhSPY2w_12node_runtime
core::ptr::drop_in_place::<trie_db::triedbmut::TrieDBMut<sp_trie::Layout<sp_runtime::traits::BlakeTwo256>>>
```

Alrighty! This is specific enough for us to create a build, and have something we can analyze. Replacing our main `lib.rs` with a static reference and adding the relevant dependencies in the` Cargo.toml`:

```rust
#![cfg_attr(not(feature = "std"), no_std)]

#[no_mangle] pub static FOO: unsafe fn(
    *mut trie_db::triedbmut::TrieDBMut<'static, 
        sp_trie::Layout<sp_runtime::traits::BlakeTwo256>,
    >,
) = core::ptr::drop_in_place;
```

We now have a short crate that triggers the bug in our code path. 

Now, to analyzing the `llvm` output, we compile this with:

```bash
cargo rustc -p tiny-package --release --target=wasm32-unknown-unknown -- -C llvm-args=-print-after-all -Z no-parallel-llvm 2> llvm-log-1
```

With a clean example, we can now look at the `llvm` output and the diffs between two runs again, revealing the first change to be:

```
-  successors: %bb.55(0x80000000); %bb.55(100.00%)
+  successors: %bb.58(0x80000000); %bb.58(100.00%
```

As assumed the internal order of processing changes between runs. This happens in the step `# *** IR Dump After WebAssembly Fix Irreducible Control Flow ***:`. [We find in the llvm code here](https://github.com/llvm/llvm-project/blob/a6d8a055e92eb4853805d1ad1be0b1a6523524ef/llvm/lib/Target/WebAssembly/WebAssemblyFixIrreducibleControlFlow.cpp#L236). Generally speaking, this has to be a compiler bug, because from the name of it, this step is supposed to prevent exactly that problem we are experincing. 

Once you've found a bug in an external repo, the first thing to do is see if the issue is already reported or patched. And indeed, we can find a [commit to master](https://github.com/llvm/llvm-project/commit/3648370a79235ddc7a26c2db5b968725c320f6aa), not yet released.

As expected, `llvm` also stores some of its own internal state in non-order-persistent ways (e.g. HashMaps with randomized keys). To ensure the order in which they are processed and yielded is deterministic, they are sorted before processing. Before this patch, under certain conditions, namely "fixing up a set of mutual loop entries", this wasn't reliably done. It appears that with our code we triggered this bug within the `llvm-ir` representation. 

After patching a local `llvm` in `rustc` and doing multiple runs, we can confirm: **this is the bug and this commit fixes it**. Although the bug was probably present in older versions of the compiler, only a new code base did actually triggered the path that would lead to it.

### 8. Backporting the compiler fix

Though the fix was merged into llvm back in February, it wasn't yet part of llvm10 nor any of the `10.0.1` release candidates, but will probably be part of the upcoming llvm 11 release. Unfortunately, rust is rather slow in picking up and porting to the latest version of llvm. Regardless, the Rust Team is very open for backports to the llvm and these migrate rather quickly into the nightly version of rust, which we are most interested in. As a result, we will can get a fixed wasm32 nightlt compiler pretty soon.

As of the time of writing the [backport patch has been accepted and merged](https://github.com/rust-lang/llvm-project/pull/68) and we are just waiting for [rustc to update its submodule](https://github.com/rust-lang/rust/blob/e59b08e62ea691916d2f063cac5aab4634128022/.gitmodules#L37). 🤞

---

_Credits_: Gratitude to [eddyb](https://github.com/eddyb) for their tremendous help, especially on the compiler part of things, knowing off and fiddling around the llvm args and pin down the bug in llvm ❤️!. Also a thanks to the [Museums of Victoria for sharing their Arctic Expedition Photos for free for reuse on unsplash](https://unsplash.com/collections/9338870/antarctic-expeditions-), which I used as the header picture.
