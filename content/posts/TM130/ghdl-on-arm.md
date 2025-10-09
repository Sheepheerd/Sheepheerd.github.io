---
title: "Building GHDL on Arm"
date: "2025-09-10"
tags: ["m1", "asahi", "ghdl", "nix"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Arm is capable (most of the time)"
disableHLJS: true
disableShare: true
hideSummary: false
searchHidden: false
ShowReadingTime: false
ShowBreadCrumbs: false
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: false
UseHugoToc: false
---

# The Problem

[GHDL](https://github.com/ghdl) is a popular VHDL simulator. Currently, I am taking an FPGA course and I need to use either 100GB [Vivado](https://www.xilinx.com/support/download.html) or tiny GHDL. I use an M1 MacBook Air as my daily driver. It is not x86_64 . . . . it is ARM. It's also running NixOS instead of the more common Fedora Remix. Because of this you might assume I can use the ghdl that is packaged in the Nix repos. Wrong! Arm is not currently supported :)

# Solution
Of course I can go to the github repo, clone it, and build it myself. But that is not the Nix way. Imperative bad. So I've have crafted a Flake that will solve this issue. The problem with the current build of ghdl on the Nix repos is that it is using gnat13. We can solve this issue by making a derivation that will build with an updated version. Problem Solved

```nix
{
  description = "Flake to build GHDL from upstream source with gnat-bootstrap14";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    ghdl-src = {
      url = "github:ghdl/ghdl";
      flake = false;
    };
  };

  outputs =
    {
      self,
      nixpkgs,
      ghdl-src,
    }:
    let
      systems = [
        "aarch64-linux"
      ];
      forAllSystems = f: nixpkgs.lib.genAttrs systems (system: f (import nixpkgs { inherit system; }));
    in
    {
      packages = forAllSystems (pkgs: {
        default = pkgs.stdenv.mkDerivation {
          pname = "ghdl";
          version = "git";

          src = ghdl-src;

          nativeBuildInputs = with pkgs; [
            git
            cmake
            gcc
            llvm
            wget
            gnumake
            automake
            zlib
            gnat-bootstrap14
            python3
          ];

          configurePhase = ''
            export GHDL_BACKEND=llvm
            export CC=${pkgs.gcc}/bin/gcc
            export CXX=${pkgs.gcc}/bin/g++

            ./configure --prefix=$out --with-llvm-config --enable-libghdl

          '';

          buildPhase = ''
            make
          '';

          installPhase = ''
            make install
          '';
        };
      });
    };
}
```

This is my current flake. If you would like to add just the package to your own flake, see below

# Example for User flake
```nix

{
  description = "Flake to build GHDL from upstream source with gnat-bootstrap14";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    ghdl = {
      url = "github:Sheepheerd/nix-ghdl";
    };

  };

  outputs =
    {
      self,
      nixpkgs,
      ghdl,
    }:
    let
      systems = [
        "aarch64-linux"
        "aarch64-darwin"
      ]; # adjust if needed
      forAllSystems = f: nixpkgs.lib.genAttrs systems (system: f (import nixpkgs { inherit system; }));
    in
    {
      devShells = forAllSystems (pkgs: {
        default = pkgs.mkShell {
          buildInputs = with pkgs; [
            ghdl.packages.${pkgs.system}.default
            pkgs.gtkwave
          ];
        };
      });
    };
}
```
