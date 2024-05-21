# Set up a basic poetry2nix environment

A minimalist example using
[poetry2nix](https://github.com/nix-community/poetry2nix) without any
package development. These instructions should generate a new Python
environment using Nix. It does not require a Python development
package to use this since `package-mode = false`. It also uses a Flake
that doesn't need to be edited. All that needs editing is the
pyproject.toml file. This should be edited using the poetry tool. IMO,
this is the simplest way to use Python maintained as as Nix
environment. Note that here we're using Micromamba to install and use
poetry, but we could use any Python environment manager to do this or
even Nix probably. However, I tend to use Micromamba for this
boostrapping issue. These instructions do not require this respository to be cloned. However, it does use the `flake.nix` file in the repository, which should be downloaded.

## 1. Nix

Install Nix using the instructions on [nix.dev]. Make sure Flakes are
working correctly, [see below](#flakes)

## 2. Setup Poetry and the "basic" environment (if necessary)

Firstly install [Poetry](https://python-poetry.org) in way that works
with your current environment management system. I installed
[Micromamba using Nix][micromamba-nix] and home-manager as I already
use Nix. Micromamba is used outside of the Nix environment as a
boostrapping mechanism to deal with Poetry. You could use Conda or Pip
to install Poetry, it doesn't matter.

	$ mkdir basic
	$ cd basic
    $ eval "$(micromamba shell hook -s bash)"
    $ micromamba create -n basic python poetry
    $ micromamba activate basic

## 3. Create the pyproject.toml

The first step is to create a new Poetry package.

    $ poetry init

This drops you into a question and answer session. Use the default
answers, but say "no" to defining the dependencies interactively.

In addition add `package-mode = false` to the `[tool.poetry]` section
of the pyproject.toml. It should look something like this.

~~~
[tool.poetry]
name = "basic"
version = "0.1.0"
description = ""
authors = ["Daniel Wheeler <daniel.wheeler2@gmail.com>"]
readme = "README.md"
package-mode = false


[tool.poetry.dependencies]
python = "^3.12"


[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
~~~

## 4. Add packages

You can add new packages using

    $ poetry add numpy
	
for example and then run

    $ poetry lock
	
to update the lock file.

## 5. Setup the flake

Add the flake.nix from this repository to the directory with the new
environment.

    $ wget https://raw.githubusercontent.com/wd15/basic-poetry2nix/main/flake.nix
   
Set up the `basic` directory to be a Git repository.

    $ git init
    $ git add flake.nix pyproject.toml poetry.lock
    $ git ci -m "initial commit"
   
## 6. Run the environment

Now the development environment should now work

    $ nix develop
    $ python -c "import numpy; print(numpy.__version__)"
    $ exit

This should have generated `flake.lock` file. Add this to the
repository.

    $ git add flake.lock
    $ git ci -m "adding flake.lock"
   
## 7. Additional odds and ends

### `ModuleNotFoundError`

If you get the `ModuleNotFoundError: No module named 'PACKAGENAME'`
then [read
this](https://github.com/nix-community/poetry2nix/blob/master/docs/edgecases.md#cases). Basically,
the following section of the flake.nix needs to be edited.

```
      pypkgs-build-requirements = {
        # attrs = [ "hatchling" ];
        # urllib3 = [ "hatchling" ];
        # hbreader = [ "setuptools" ];
        # pytrie = [ "setuptools" ];
        # url-normalize = [ "poetry-core" ];
        # cfgraph = [ "setuptools" ];
        # pytest-logging = [ "setuptools" ];
        # mkdocs-windmill = [ "setuptools" ];
        # paginate = [ "setuptools" ];
      };
```

The package that is failing needs to correctly identify its build
mechanism. For example, in the above list, `pytest-logging` requires
`setuptools`.

### Nix Shell Prompt

**NOTE**: the `nix develop` command fails to change the shell prompt
to indicate that you are now in a Nix environment. To remedy this add
the following to your `~/.bashrc`.

``` bash
show_nix_env() {
  if [[ -n "$IN_NIX_SHELL" ]]; then
    echo "(nix)"
  fi
}
export -f show_nix_env
export PS1="\[\e[1;34m\]\$(show_nix_env)\[\e[m\]"$PS1
```

### Flakes

The PFHub Nix expressions use an experimental feature known as flakes,
see the [official flake documentation][flakes].

To enable Nix flakes, add a one liner to either
`~/.config/nix/nix.conf` or `/etc/nix/nix.conf`. The one liner is

``` text
experimental-features = nix-command flakes
```

If you need more help consult [this
tutorial](https://www.tweag.io/blog/2020-05-25-flakes/).

To test that flakes are working try

    $ nix flake metadata github:wd15/basic-poetry2nix
    Resolved URL:  github:wd15/basic-poetry2nix
    ⁞
    └───systems: github:nix-systems/default/da67096a3b9bf56a91d16901293e51ba5b49a27e

### Update Flakes

    $ nix flake update
    $ nix develop

When the flake is updated, a new `flake.lock` file is generated which
must be committed to the repository alongside the `flake.nix`. Note
that the flake files depend on the `nixos-23.11` branch which is only
updated with backports for bug fixes. To update to a newer version of
Nixpkgs then the `flake.nix` file requires editing to point at a later
Nixpkgs version.


[nix.dev]: https://nix.dev
[micromamba-nix]: https://nixos.wiki/wiki/Python#micromamba
[flakes]: https://nixos.wiki/wiki/Flakes
