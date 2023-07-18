# PureScript Overlay

[![daily-update](https://github.com/thomashoneyman/purescript-nix/actions/workflows/daily-update.yaml/badge.svg)](https://github.com/thomashoneyman/purescript-nix/actions/workflows/daily-update.yaml)
[![darwin-support](https://github.com/thomashoneyman/purescript-nix/actions/workflows/darwin-support.yaml/badge.svg)](https://github.com/thomashoneyman/purescript-nix/actions/workflows/darwin-support.yaml)
[![nix-unit-tests](https://github.com/thomashoneyman/purescript-nix/actions/workflows/nix-unit-tests.yaml/badge.svg)](https://github.com/thomashoneyman/purescript-nix/actions/workflows/nix-unit-tests.yaml)

Pure and reproducible overlay for the standard PureScript toolchain, including support for Nix flakes. The toolchain is auto-updated every day. Currently supported tools:

- `purs`, the compiler
- `spago`, the package manager
- `purs-tidy`, the code formatter
- `purs-backend-es`, the optimizer

> :warning: This library is unstable and may be reorganized. Use at your own risk!

The overlay is tested on the following architectures:

- x86_64-linux
- x86_64-darwin (Intel Mac)
- aarch64-darwin (M1 Mac)
- aarch64-linux

The included overlay inserts the latest stable and unstable executables into your packages (ie. `purs`, `purs-unstable`, and so on). You can see all specific versions in the [named.json](./manifests/named.json) file. It also provides many versions of each executable under a `-bin` namespace (ie. `purs-bin`, `spago-bin`, and so on) so you can access specific versions of a tool. For example, you can use `purs-bin.purs-0_15_8` to get the 0.15.8 PureScript compiler. These are tracked in the [manifests](./manifests/) directory.

There is also an included library named `purix` which provides helper functions for building PureScript packages, namely:

- `buildSpagoLock`: Build `output` directories for any package or workspace listed in a spago.lock file

## Usage

In a Nix flake, use the provided overlay when importing nixpkgs to get access to tools like `purs` and `spago` and functions like `buildSpagoLock`. For example, the below flake creates a development shell with recent versions of the PureScript compiler and Spago package manager:

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-23.05";
    flake-utils.url  = "github:numtide/flake-utils";
    purescript-overlay.url = "github:thomashoneyman/purescript-overlay";
    purescript-overlay.inputs.nixpkgs.follows = "nixpkgs";
  };

  outputs = { self, nixpkgs, flake-utils, purescript-overlay }:
    flake-utils.lib.eachDefaultSystem (system:
      let
        overlays = [ purescript-overlay.overlays.default ];
        pkgs = import nixpkgs {
          inherit system overlays;
        };
      in {
        devShells.default = pkgs.mkShell {
          # You now have access to the standard PureScript toolchain in pkgs
          buildInputs = [
            pkgs.purs
            pkgs.spago-unstable
            pkgs.purs-tidy-bin.purs-tidy-0_10_0
            pkgs.purs-backend-es
          ];
        };
      }
    );
}
```

You can also run individual packages from the flake, e.g. to format your `src` directory:

```console
nix run github:thomashoneyman/purescript-overlay#purs-tidy format-in-place src
```

## Tests

You can run the repository tests using a combination of `nix eval .#lib` (to run the unit tests in Nix) and `nix flake check` (to run the derivation-based tests). Both are executed in CI.
