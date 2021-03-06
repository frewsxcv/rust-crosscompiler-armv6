# docker-rust

Public trusted images available on:

* [Docker Hub](https://registry.hub.docker.com/u/dlecan/rust-crosscompiler-armv6/)

This repository is used to build a Docker image based on Debian for the Rust programming language for armv6 (eg. : all models of Raspberry Pi) architectures and a few supporting tools. The image includes :
- `rustc`
- `rustdoc`
- `cargo`
- `git`
- SSL certificates
- build essentials
- Linaro cross compiler
- many libraries compiled for armv6

So it should be able to run `cargo build` on most projects out of the box. The path `/source` is a volume where you can mount a Cargo project from the host machine.

## Why ?

To setup a full cross-compiling environment is complicated. You need:
- a build toolchain for armhf architecture : not available on Debian, but easy to install on Ubuntu
- a Rust compiler for armhf architecture : no binaries are available, so you need to compile it from sources
- to tell Cargo how to cross compile
- many development libraries (headers + lib) such as openssl, compiled for armv6 architecture
- ...

After that, if you succeed, your workstation will be a real mess.

So I created this Docker image dedicated to Rust cross-compilation on armv6.

## Usage

The following command will run `cargo build --release` with the current directory on the host shared, with Rust 'stable' channel:

```bash
docker run -it --rm \
  -v $(pwd):/source \
  dlecan/rust-crosscompiler-armv6:stable
```

**Rust 'target' architecture is forced to `arm-unknown-linux-gnueabihf` within the container** (parameter `target=arm-unknown-linux-gnueabihf` automatically added).

If you want to run another command from within the container, add it at the end of the previous command line.
For example, to display Cargo version:

```bash
docker run -it --rm \
  -v $(pwd):/source \
  dlecan/rust-crosscompiler-armv6:stable \
  cargo --version
```

## Optimized usage

With the previous configuration, Cargo dependencies will be downloaded and compiled again and again each time you launch the Docker container.
If you want to avoid this behavior, you must persist Cargo cache folders within the container on the host. The best is to share them with the Cargo cache folders of the host.

So you can use the following command line:

```bash
docker run -it --rm \
  -v $(pwd):/source \
  -v ~/.cargo/git:/root/.cargo/git \
  -v ~/.cargo/registry:/root/.cargo/registry \
  dlecan/rust-crosscompiler-armv6:stable
```

This will cache and share sources of Cargo dependencies and Rust built librairies. It assumes that your Cargo configuration is located in `~/.cargo`.

Caveat: unfortunately, with configuration, `~.cargo` files will be owned by `root` user, so you will have to `chown` them to be able to use Cargo in the host.

To sum up, here is the script I use, located in `/usr/local/sbin/rxc` (rust cross compiler):

```bash
#!/bin/bash

docker run -it --rm \
  -v $(pwd):/source \
  -v ~/.cargo/git:/root/.cargo/git \
  -v ~/.cargo/registry:/root/.cargo/registry \
  dlecan/rust-crosscompiler-armv6:stable "$@"

sudo chown -R `stat -c %u:%g $HOME` $(pwd) ~/.cargo
```

You can then `rxc cargo build`.

## Usage with script

## Inspiration

- https://github.com/japaric/ruststrap/blob/master/1-how-to-cross-compile.md
- https://github.com/djmaze/armhf-ubuntu-docker
- https://github.com/sdt/docker-raspberry-pi-cross-compiler
- http://www.openframeworks.cc/setup/raspberrypi/Raspberry-Pi-Cross-compiling-guide.html
- http://exploreembedded.tumblr.com/post/115333857238/guide-to-cross-compile-qt-54-for-the-raspberry-pi
- https://github.com/twobitcircus/rpi-build-and-boot
- https://github.com/shahriman/cross-compile-tools


## License

[MIT](http://opensource.org/licenses/MIT)
