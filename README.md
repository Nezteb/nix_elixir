# Going Full Functional: Using Nix to build Erlang and Elixir Projects

Eventually this will turn into a blog post/guide/tutorial of intermediate "difficulty".

### Goals

1. Create most basic Elixir/Phoenix/Liveview app
2. Set up Nix boilerplate on macOS (arm64)
3. Use Nix (devenv/devbox?) to build container
4. Change Erlang/Elixir versions based on current project
5. Editor and language server setup
6. Set up GitHub CI
7. Deploy Nix to Fly
8. Document troubleshooting, uninstalling, reinstalling

#### Questions / TODO

- Get Nix working on my machine...
  - https://github.com/NixOS/nix/issues/1402
  - https://github.com/NixOS/nix/issues/4716
  - https://github.com/NixOS/nix/issues/6078
  - https://github.com/NixOS/nix/issues/6675
  - https://github.com/NixOS/nix/issues/6540 
- For goal #3, which install "method" should we use?
  -  Vanilla?
    - `nix-env` (installs permanently)
      - People seem to recommend against this and to use `home-manager`.
    - `nix-shell` (only installs in new shell)
      - `nix shell` (flakes-only variant?)
        - https://nixos.wiki/wiki/Nix_command
        - `nix.settings.experimental-features = [ "nix-command" "flakes" ];`
    - `nix-command` (experimental?)
  -  Flake profiles?
    - `nix profile install` 
    - https://nixos.wiki/wiki/Flakes#Install_packages_with_.60nix_profile.60 
  -  Declarative with flakes?
    - `nix flake init` 
    - https://devenv.sh/guides/using-with-flakes/ 
  -  Declarative without flakes?
    - `configuration.nix` with `environment.systemPackages` set
- For goal #3, should I use the NixOS container image instead of Debian?
- For goal #6, is it worth considering using Hydra/NixOps?
  - https://nixos.wiki/wiki/Hydra
  - https://github.com/NixOS/nixops?
- For goal #7, check out https://github.com/fly-apps/nix-base

## Outline/Snippets

The term "Nix" can mean few things depending on the context:
- The purely functional programming language, Nix
  - ["You can kind of think of it as JSON with types; I’m stealing this from a friend of mine. And it’s so simple that it doesn’t even really have named top-level variables. You can’t even give names to things. It’s just a data structure, and then you can write some transformations instead of the data structure." - Vincent Ambo](https://changelog.com/shipit/37#transcript-25)
- The package manager, Nix
  - ["The package manager implements a concept called derivations. A derivation is essentially a data structure that says “We have a transformation that we want to apply to some data.” Usually, this transformation is something like running a compiler or running some other tool that does file transformations. And these derivations specify all of the inputs that they have, fully pinned, which means that we have full SHA hashes of everything that gets into the derivation, and then this information together can be used to create a hash." - Vincent Ambo](https://changelog.com/shipit/37#transcript-25)
  - Each package is essentially a Merkel tree / DAG of input hashes.
- The operating system, NixOS.
  - https://nixos.org/manual/nixos/stable/#preface

Commands: `nix shell` “installs” requested packages into an ephemeral shell environment. `nix develop` is sort of like `nix build --interactive` or `nix build --debug`; it creates a build environment for a particular Nix package and instead of automatically running build phases, it drops you into a shell in the build envionrment so that you can debug the build itself. `nix develop` also allows you to set up other aspects of the build environment, such as variables. [(source: Nix forum)](https://discourse.nixos.org/t/difference-between-nix-shell-nix-shell-nix-develop/32469)

## Link Dump

### Docs

- Nix
  - https://github.com/NixOS/nix
  - https://zero-to-nix.com/start
  - https://zero-to-nix.com/concepts
  - https://nixos.org/manual/nix/stable/installation/installing-binary#macos-installation
    - Note: Do not use this. Use:
      - https://determinate.systems/posts/determinate-nix-installer
      - https://determinate.systems/posts/graphical-nix-installer
  - https://nixos.org/guides/how-nix-works
  - https://nixos.org/guides/nix-pills/pr01
  - https://github.com/nixos/nix.dev
  - https://nix.dev/guides/best-practices
  - https://github.com/nix-community/nix-direnv
- Home Manager
  - https://github.com/nix-community/home-manager
  - https://nix-community.github.io/home-manager/index.html#id-1.2
- Devenv
  - https://github.com/cachix/devenv
  - https://devenv.sh/guides/using-with-flakes/
- Devbox
  - https://github.com/jetpack-io/devbox
  - https://www.jetpack.io/devbox/docs/quickstart/
- Nixery
  - https://github.com/tazjin/nixery
  - https://nixery.dev/
- Hydra
  - https://github.com/NixOS/hydra
 
### Experimental Things

- Nix-Darwin
  - https://github.com/LnL7/nix-darwin
- Nuenv
  - https://github.com/DeterminateSystems/nuenv
  - https://determinate.systems/posts/nuenv
- Fleek
  - https://github.com/ublue-os/fleek
- Flox
  - https://github.com/flox/flox
  - https://flox.dev/blog/homebrew

### Example Configs

- https://github.com/Misterio77/nix-starter-configs
- https://github.com/mitchellh/nixos-config
- https://github.com/hauleth/nix-elixir
- https://github.com/the-nix-way/dev-templates/blob/main/elixir/flake.nix
- https://github.com/search?q=language%3Anix+Elixir+stars%3A%3E1&type=repositories&s=updated&o=desc

### Text Resources

- https://github.com/nix-community/awesome-nix
- https://nixos-and-flakes.thiscute.world/preface
- https://ianthehenry.com/posts/how-to-learn-nix/
- https://paperless.blog/nix-shell-template
- https://jvns.ca/categories/nix/
- https://animeshz.github.io/site/notes/20-29--DevEnvironment/21--Linux/21.02-Nix.html
- https://christine.website/talks/nixos-pain-2021-11-10/
- https://shopify.engineering/what-is-nix
- https://serokell.io/blog/nix
- https://ersei.net/en/blog/tag:nixos
- https://earthly.dev/blog/what-is-nix/
- https://ejpcmac.net/blog/tags/nix/
- https://checkoway.net/musings/nix/
- https://lucperkins.dev/blog/home-manager/
- https://dee.underscore.world/blog/home-manager-flakes/
- https://juliu.is/tidying-your-home-with-nix/
- https://wickedchicken.github.io/post/macos-nix-setup/
- https://www.channable.com/tech/nix-is-the-ultimate-devops-toolkit
- https://grahamc.com/blog/nix-and-layered-docker-images/
- https://thewagner.net/blog/2021/02/25/building-container-images-with-nix/
- https://tonyfinn.com/blog/nix-from-first-principles-flake-edition/
- https://www.slice.zone/blog/nix-in-practice
- https://xeiaso.net/blog/nix-flakes-1-2022-02-21/
- https://fasterthanli.me/series/building-a-rust-service-with-nix
- https://blog.wesleyac.com/posts/the-curse-of-nixos

### Video Resources

- Nixology
  - https://www.youtube.com/playlist?list=PLRGI9KQ3_HP_OFRG6R-p4iFgMSK1t5BHs

### Audio Resources

- https://changelog.com/topic/nix
- https://pod.link/1602572955/episode/32b5c52e4bf53272fa0c5d27b1a30042
  - "Vim and Nix with Jasper Woudenberg", Software Unscripted

### Services Using Nix

- Replit (similar to GitHub Codespaces, but built on Nix instead of devcontainer)
  - https://replit.com/languages/elixir
  - https://docs.replit.com/programming-ide/nix-on-replit
  - https://docs.replit.com/tutorials/replit/nix-packaging
