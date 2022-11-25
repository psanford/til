# NixOS cheatsheet

List system generations:

```
sudo nix-env --list-generations --profile /nix/var/nix/profiles/system
```

List packages in current system generation:

```
nix-store --query --references /nix/var/nix/profiles/system
```

List packages in older generation:

```
nix-store --query --references /nix/var/nix/profiles/system-147-link
```

Garbage collection. If you need to cleanup old kernels run `nix-collect-garbage` and
then do a `rebuild switch` to update grub:

```
sudo nix-collect-garbage --delete-older-than 3d
sudo nixos-rebuild switch
```
