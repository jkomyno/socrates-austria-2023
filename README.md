# WebAssembly for Backend Devs: Rust & TypeScript edition

> Accompanying slides for the talk I presented at SoCraTes Austria 2023 @ Linz.

<p>
  <a href="https://github.com/jkomyno/socrates-austria-2023/actions/workflows/ci.yml">
    <img alt="Github Actions" src="https://github.com/jkomyno/socrates-austria-2023/actions/workflows/ci.yml/badge.svg?branch=main" target="_blank" />
  </a>

  <a href="https://github.com/jkomyno/socrates-austria-2023/blob/main/LICENSE">
    <img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-yellow.svg" target="_blank" />
  </a>
  
</p>

Slides for this talk are also available [here](https://jkomyno-slides-socrates-austria-2023.vercel.app/).

## Abstract

This talk will teach you how to run performance critical native code in a JavaScript runtime with Rust, without the burden of distributing platform-dependent artifacts. You will learn how to smoothly integrate WebAssembly (Wasm) into your Node.js application, while also automatically generating idiomatic TypeScript definitions. And while Wasm looks like a clear winner thanks to its portability, it's not a silver bullet. That's why we'll also solve the typical Wasm data serialization issues, addressing limitations and escape hatches. Finally, we will present some lesser-known tricks to let WebAssembly interact with the outside world without resorting to WASI - by defining the I/O logic in Node.js and executing it within the Wasm context.

## Inspirations

This repository and talk wouldn't have been possible without the following projects:

- [`js_sys`](https://github.com/rustwasm/wasm-bindgen/tree/main/crates/js-sys)
- [`wasm-bindgen`](https://github.com/rustwasm/wasm-bindgen)
- [`wasm-bindgen-futures`](https://github.com/rustwasm/wasm-bindgen/tree/main/crates/futures)
- [`serde`](https://github.com/serde-rs/serde)
- [`serde_json`](https://github.com/serde-rs/json)
- [`serde-wasm-bindgen`](https://github.com/cloudflare/serde-wasm-bindgen)
- [`tsify`](https://github.com/madonoharu/tsify)

Please consider starring, supporting, and contributing to them.

## Get Started

### Requirements

- [`nodejs@18.12.1`](https://nodejs.org/en/download/) or superior*
- [`pnpm@7.20.0`](https://pnpm.io/installation) or superior*

(*) These are the versions used to develop this repository. Older versions might work as well, but they haven't been tested.

### Install Dependencies

- Install dependencies:
  ```sh
  pnpm i
  ```

In [`./rust`](./rust):

- Install the Rust toolchain via [Rustup](https://rustup.rs/):
  ```sh
  curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
  ```

- Add suppport for the `wasm32-unknown-unknown` compilation target for Rust:
  ```sh
  rustup target add wasm32-unknown-unknown
  ```

- Install `wasm-bindgen`:
  ```sh
  cargo install -f wasm-bindgen-cli@0.2.87
  ```
  
  (the specific version is important, as `wasm-bindgen-cli` doesn't yet follow semantic versioning. This version needs to match the version of the `wasm-bindgen` dependency in the `Cargo.toml` files of the Rust crates)

### Build & Test

With Docker:

  - Build and run the local Docker image:

    ```sh
    ./build.sh
    ```

Without Docker:

  - Run Rust unit tests and build the WebAssembly artifacts:

    ```sh
    pnpm build:wasm
    ```

  - Run Node.js unit tests:

    ```sh
    pnpm test:ci
    ```

## Example Playgrounds

### `playground-wasm-bindgen`

The local [`playground-wasm-bindgen`](./rust/playground-wasm-bindgen/src/lib.rs) crate demonstrates how to use `wasm-bindgen` to export Rust functions and types (in the form of structs / enums) to TypeScript.

The [`functions::unsupported`](./rust/playground-wasm-bindgen/src/functions/unsupported.rs) module (respectively, the [`types::unsupported`](./rust/playground-wasm-bindgen/src/types/unsupported.rs) module) contains a set of functions (respectively, types) that are not supported by `wasm-bindgen` by default, with comments showing the compilation errors that are thrown when trying to export them.

For instance, trying to compile the [following code](https://github.com/jkomyno/socrates-austria-2023/blob/main/rust/playground-wasm-bindgen/src/functions/unsupported.rs#L4-L30)

```rust
/// Given a Vec<Vec<i32>> vector, return its length.
#[wasm_bindgen]
pub fn get_nested_array_length(x: Vec<Vec<i32>>) -> usize {
  x.iter().flatten().count()
}
```

would result in a compilation error like the following:

```console
error[E0277]: the trait bound `Vec<i32>: JsObject` is not satisfied
  --> wasm-bindgen-playground/src/functions/unsupported.rs:27:1
   |
27 | #[wasm_bindgen]
   | ^^^^^^^^^^^^^^^ the trait `JsObject` is not implemented for `Vec<i32>`
   |
   = help: the following other types implement trait `FromWasmAbi`:
             Box<[JsValue]>
             Box<[T]>
             Box<[f32]>
             Box<[f64]>
             Box<[i16]>
             Box<[i32]>
             Box<[i64]>
             Box<[i8]>
           and 6 others
   = note: required for `Box<[Vec<i32>]>` to implement `FromWasmAbi`
   = note: this error originates in the attribute macro `wasm_bindgen`
```

- You can find the TypeScript tests for the `playground-wasm-bindgen` crate in [`./nodejs/demo/__tests__playground-wasm-bindgen.test.ts`](./nodejs/demo/__tests__/playground-wasm-bindgen.test.ts) and [`./nodejs/demo/__tests__/playground-wasm-bindgen.test-d.ts`](./nodejs/demo/__tests__/playground-wasm-bindgen.test-d.ts)
- You can find the TypeScript bindings for the `playground-wasm-bindgen` crate in [`./nodejs/demo/wasm/playground_wasm_bindgen.d.ts`](./nodejs/demo/wasm/playground_wasm_bindgen.d.ts)

### `playground-serde-wasm-bindgen`

The local [`playground-serde-wasm-bindgen`](./rust/playground-serde-wasm-bindgen/src/lib.rs) crate demonstrates how to use `wasm-bindgen` combined with `serde` and `serde-wasm-bindgen` to export Rust functions and types to TypeScript. This allows you to deal with complex types both in Wasm arguments and return types, at the cost of loosing strongly typed TypeScript bindings.

- You can find the TypeScript tests for the `playground-serde-wasm-bindgen` crate in [`./nodejs/demo/__tests__/playground-serde-wasm-bindgen.test.ts`](./nodejs/demo/__tests__/playground-serde-wasm-bindgen.test.ts)
- You can find the TypeScript bindings for the `playground-serde-wasm-bindgen` crate in [`./nodejs/demo/wasm/playground_serde_wasm_bindgen.d.ts`](./nodejs/demo/wasm/playground_serde_wasm_bindgen.d.ts)

### `playground-wasm-tsify`

The local [`playground-wasm-tsify`](./rust/playground-wasm-tsify/src/lib.rs) crate demonstrates how to use `wasm-bindgen` combined with `serde` and `tsify` to export Rust functions and types to TypeScript. This allows you to deal with complex types both in Wasm arguments and return types, as well as obtaining strongly typed and idiomatic TypeScript bindings.

- You can find the TypeScript tests for the `playground-wasm-tsify` crate in [`./nodejs/demo/__tests__/playground-wasm-tsify.test.ts`](./nodejs/demo/__tests__/playground-wasm-tsify.test.ts)
- You can find the TypeScript bindings for the `playground-wasm-tsify` crate in [`./nodejs/demo/wasm/tsify.d.ts`](./nodejs/demo/wasm/playground_wasm_tsify.d.ts)

## Demo: I/O in WebAssembly

**Note**: for this demo, open a terminal into the [`./nodejs/demo`](./nodejs/demo) folder.

One can define Node.js functions that perform I/O operations (normally forbidden in WebAssembly) and forward them to WebAssembly by using the `js_sys` crate. Moreover, one can use the `wasm_bindgen_futures` crate to await JavaScript functions that return a Promise.

We can ask WebAssembly to read an example [./nodejs/demo/io.txt](./nodejs/demo/io.txt) file and print its content to the console via:

```bash
npx ts-node ./src/io.ts
```

We expect the following output:

```bash
[rust] Calling async fn from Rust...
[rust] Async fn is running
[rust] Awaiting promise...
[rust] Promise resolved with: JsValue("You are reading I/O from WebAssembly!
$ € 𐐷 𤭢
")

[node] File content: You are reading I/O from WebAssembly!
$ € 𐐷 𤭢
```

## 👤 Author

**Alberto Schiabel**

* Twitter: [@jkomyno](https://twitter.com/jkomyno)
* Github: [@jkomyno](https://github.com/jkomyno)

Please consider supporting my work by following me on Twitter and starring my projects on GitHub.
I mostly post about TypeScript, Rust, and WebAssembly. Thanks!

## 📝 License

Built with ❤️ by [Alberto Schiabel](https://github.com/jkomyno).
This project is [MIT](https://github.com/jkomyno/socrates-austria-2023/blob/main/LICENSE) licensed.
