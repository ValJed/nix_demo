---
title: "NixOS Demo"
author: ValJed
---

What is Nix?
===

Initially created to write packages.

There is a conceptual difference between:
- the Nix language itself (pretty minimal)
- the Nixos options (contains a lot of things to build systems)

---

Characteristics
===
1. Lazily evaluated 
2. Dynamically typed
3. Functional
4. Reproducible


<!-- end_slide -->

Language
===

Everything is an expresion.
Anything written in a *nix* file can be evaluated.

You can test to evaluate pieces of code using **nix repl** command.

```bash +exec
nix repl --expr '{a = map (x: x * 2) [1 2 3]}'
```

Data Types
===

```nix
# Variable and Set
let set = {
  string = "hello";
  integer = 1; 
  float = 3.141;
  bool = true;
  null = null;
  list = [ 1 "two" false ];
  path = ./path; # Native support for paths
  attribute-set = {
    a = "hello";
    b = 2;
    c = 2.718;
    d = false;
  }; # comments are supported
}
```

<!-- end_slide -->

Advanced
===

I won't dive too much here, more concepts to now:
# Recursive sets

Accessible set values inside the set.

```nix
rec {
	a = 2;
	b = a + 3
}
```
---
# with keyword

Brings values of a set to the scope of any expression.

```nix
with { a = "Hello"; b = "World"; };
"${a} ${b}"

# Will be evaluated:
# "Hello World"
```

<!-- end_slide -->

Let and In expressions
===

Allow to evaluate  an expression based on some defined variables.
Can be assigned to a variable.

```nix
let
 x = 1;
 y = 2;
in x + y
```

<!-- end_slide -->

Functions
===

```nix
myFunc = a: a + 1
```

# Mixed with let and in syntax:

```nix
let 
	myFunc = a: a + 1;
in
	myFunc 4

# Evaluated: 5
```

# Example, in the context of NixOs:

```nix
{config, pkgs, ...}: # Function arguments
# Function returned value
{
	programs.hyprland.enable = true;
	programs.zsh.enable = true;
	# ...
}
```

# Function Attributes:

Allow to get individual arguments, but also to get all of them inside args (to pass them to other modules for example).
```nix
# Capture arguments
myFunc = { a, b, ...}@args: a + b;
```

# Default values:

```nix
myFUnc = { a ? 100, b, ... }
```

<!-- end_slide -->

If Else
===

```nix
if args.a then args.a else;
```

<!-- end_slide -->

Setup system
===

When installed, Nixos contains two files in */etc/nixos*:
- configurations.nix
- hardware-configuration.nix

*hardware-configuration.nix* contains an auto generated config for the computer hardware, should stay untouched.

*configurations.nix* is the entry point of your config.

<!-- end_slide -->

Nixos Store
===

All nix packages are installed in the same location: */nix/store*
Each package contain a hash to be uniquely indentified.
This way your system can have multiple versions of the same package installed.
And different softwares using a different verion of the same package.


```bash +exec
ls /nix/store | grep presenterm
```

<!-- end_slide -->

Nix for dev
===

In dev projects, you can add a **flake.nix** file at the root folder. 

It will allow developers that use Nix to run the project without having to manually install dependencies.
The Lodash one looks like this:

```nix
# Lodash flake
{
  inputs = {
    utils.url = "github:numtide/flake-utils";
  };

  outputs = {
    self,
    nixpkgs,
    utils,
  }:
    utils.lib.eachDefaultSystem (system: let
      pkgs = nixpkgs.legacyPackages."${system}";
    in rec {
      devShell = pkgs.mkShell {
        nativeBuildInputs = with pkgs; [
          yarn
          nodejs-14_x
          nodePackages.typescript-language-server
          nodePackages.eslint
        ];
      };
    });
}
```

<!-- end_slide -->

Nix for servers
===

Nix is cool for personal computers when you want to have an up to date but also safe work environment.
But for servers, it looks even more powerful.

You can use a single Nix configuration for multiple servers.

Can simplify configuration for server stuff like **Nginx**.

Example:
```nix
services.nginx.enable = true;
services.nginx.virtualHosts."myhost.org" = {
    addSSL = true;
    enableACME = true;
    root = "/var/www/myhost.org";
};
security.acme = {
  acceptTerms = true;
  defaults.email = "foo@bar.com";
};
```

https://nixos.wiki/wiki/Nginx

