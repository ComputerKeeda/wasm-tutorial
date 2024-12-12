# Guide to Building a Cosmos Chain Binary with Static Linking to Resolve Wasm VM Compatibility Issues

This guide outlines the process of building a Cosmos chain binary with a **statically linked library**, ensuring compatibility and portability across different environments.

> [Note]
>
> ## What is a Static-Linked Library?
> 
> When building a binary using standard build commands like the one below:
> 
> ```shell
> go build -tags "netgo" -ldflags '-X github.com/cosmos/cosmos-sdk/version.Name=junction -X github.com/cosmos/cosmos-sdk/version.AppName=junctiond -X github.com/cosmos/cosmos-sdk/version.Version=0.2.0-17-g43c85e1 -X github.com/cosmos/cosmos-sdk/version.Commit=43c85e16f0271a44442b71b4b64e27a29f7048a5 -X "github.com/cosmos/cosmos-sdk/version.BuildTags=netgo" -X github.com/tendermint/tendermint/version.TMCoreSemVer=v0.38.6' -o ./build/junctiond ./cmd/junctiond
> ```
> 
> The resulting binary depends on shared libraries. You can verify this by running:
> 
> ```shell
> ldd ./build/junctiond
> ```
> 
> This command lists all shared libraries the binary depends on. If you transfer this binary to another system, it won’t run unless those shared libraries are also installed.
> 
> **Solution:** Use static linking to build a binary with all dependencies included.

---

## Steps to Build a Static Binary for `junctiond`

### Step 1: Clone the WasmVM Repository

Clone the WasmVM repository and navigate to the desired version:

```bash
git clone https://github.com/CosmWasm/wasmvm.git
cd wasmvm
# Checkout the version used in your project
git checkout v2.1.4
```

---

### Step 2: Build WasmVM

#### Install Rust and Cargo

Ensure Rust and Cargo are installed. Follow the [Rust Installation Guide](https://www.rust-lang.org/tools/install) if necessary.

#### Modify `Cargo.toml`

Navigate to the `libwasmvm` directory and update the `[lib]` section of `Cargo.toml` to include the `staticlib` crate type:

```toml
[lib]
crate-type = ["cdylib", "rlib", "staticlib"]
```

#### Build the Static Library

Run the following commands to build the static library:

```bash
cd libwasmvm
make build-rust
```

This generates a `libwasmvm.a` file in the `libwasmvm/target/release/` directory. Rename the file:

```bash
mv libwasmvm.a libwasmvm.x86_64.a
```

---

### Step 3: Configure Your Cosmos Chain Project

#### Update the Makefile

Modify your Cosmos chain project’s Makefile to use the `libwasmvm.x86_64.a` file for static linking.

#### Install Required Libraries

Install the following libraries:

```bash
sudo apt install musl musl-dev musl-tools libc6-dev
```

#### Add Static Linking Flags

Update the `ldflags` in your Makefile to include:

```makefile
-extldflags "-static -L/path/to/your/wasmvm/libwasmvm/target/release -lwasmvm -lm" -linkmode external
```

Replace `/path/to/your/wasmvm/` with the actual path to your `libwasmvm.x86_64.a` file.

---

### Step 4: Build the Cosmos Chain Binary

Run the build command to create the static binary.
