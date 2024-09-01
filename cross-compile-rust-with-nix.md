# cross compiling rust binaries on nix

This worked for me to build from a linux aarch64 host -> x86_64 target. The build
machine was nixos, the target was a traditional distro. Note the rustflags setting
the dynamic-linker to the traditional linker location for the non-nix distro.

devshell.nix:
```
{ pkgs ? import <nixpkgs> { crossSystem = { config = "x86_64-unknown-linux-gnu"; };  } }:
pkgs.mkShell {
  nativeBuildInputs = with pkgs.buildPackages; [
    pkg-config
    openssl
    clang
    rustc
    gcc
    stdenv.cc
  ];
}
```

~/.cargo/config.toml
```
[target.x86_64-unknown-linux-gnu]
linker = "x86_64-unknown-linux-gnu-gcc"
# for targeting a traditional distro, override the linker location here:
rustflags = ["-C", "link-arg=-Wl,-dynamic-linker,/lib64/ld-linux-x86-64.so.2"]
```

cargo build --target=x86_64-unknown-linux-gnu --release
