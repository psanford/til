# NixOS cheatsheet

List system generations:

```
sudo nix-env --list-generations --profile /nix/var/nix/profiles/system
```

List packages in current system generation:

```
nix-store --query --requisites /nix/var/nix/profiles/system
```

List packages in older generation:

```
nix-store --query --requisites /nix/var/nix/profiles/system-147-link
```

Garbage collection. If you need to cleanup old kernels run `nix-collect-garbage` and
then do a `rebuild switch` to update grub:

```
sudo nix-collect-garbage --delete-older-than 3d
sudo nixos-rebuild switch
```


Delete specific generations:
```
sudo nix-env --profile /nix/var/nix/profiles/system --delete-generations 225 224 223 222
```


## Overrides

# Patch a build:

```
self: super:
{
  gnupg = super.gnupg.overrideAttrs
    ({ patches ? [], ... }: {
      patches =  patches ++ [ ./0001-Revert-scd-Add-workaround-for-ECC-attribute-on-Yubik.patch ];
    });
}

#
  nixpkgs.overlays = [
    (import ./overlays/gnupg.nix)

```

# Override a version

Pin an old version:

```
@@ -1,4 +1,16 @@
 { config, pkgs, ... }:
+
+let
+  # 2022/08/19: pin gnupg to 2.3.6. 2.3.7 broke yubikey compat for some reason
+  pinGPG = pkgs.fetchFromGitHub {
+    owner  = "NixOS";
+    repo   = "nixpkgs";
+    rev    = "9cfb24af2d4f5778284ec441f9d6cf4c2b174f3d";
+    # Hash obtained using `nix-prefetch-url --unpack --unpack https://github.com/nixos/nixpkgs/archive/<rev>.tar.gz`
+    sha256 = "1k83d6hzaad8144jqyd57wfs8ia638i33ahilicbkc0wyg2gh5kb";
+  };
+  pinGPGPkgs = import pinGPG {};
+in
 {
@@ -69,7 +81,7 @@
  environment.systemPackages = with pkgs; [
-    gnupg
+    pinGPGPkgs.gnupg
```

Override fetch* version:

```

```

# Build a package

To build a local package with dependencies, where the local package is systray-inbox.nix:

```
$ nix-build -K -E 'with import <nixpkgs> {}; callPackage ./systray-inbox.nix {}'
```


# Language

Its common to see things like:
```
{ config, pkgs, ...}:
...
```

That is a function definition that takes an attribute set that at least includes attributes named `config` and `pkgs`.

Like wise:

```
self: super:
{...}
```

This is a curried function that takes `self` and then `super`.
