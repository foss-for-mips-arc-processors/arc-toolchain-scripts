# ARC GNU Toolchains

## Prerequisites

On Ubuntu:

```sh
sudo apt-get install -y --no-install-recommends \
    autoconf automake autotools-dev curl \
    libmpc-dev libmpfr-dev libgmp-dev libexpat1-dev \
    gawk build-essential libncurses-dev bison flex texinfo \
    gperf libtool patchutils bc zlib1g-dev meson ninja-build python3
```

Regenerate `configure` after editing `configure.ac`:

```sh
autoconf
```

## Configure examples

```sh
# ARC64 little-endian Picolibc (default --with-os=baremetal --with-libc=picolibc)
# Tools: arc64-gf-elf-*, alias arc64-elf-*
./configure --target=arc64 --prefix=$PWD/install

# Classic ARC big-endian Newlib
./configure --target=arc --with-endian=big --with-libc=newlib \
  --enable-multilib --with-multilib-list=release --prefix=$PWD/install

# ARC-V Newlib (riscv64-gf-elf-* plus riscv64-unknown-elf-* aliases)
./configure --target=riscv64 --with-arch=rv64gc --with-abi=lp64d \
  --with-libc=newlib --prefix=$PWD/install

# --target is only the triplet CPU; --with-arch is not required to match XLEN
./configure --target=riscv32 --with-arch=rv64gc --prefix=$PWD/install

# ARC-V Picolibc (explicit)
./configure --target=riscv32 --with-arch=rv32imac --with-abi=ilp32 \
  --with-libc=picolibc --prefix=$PWD/install
```

## Makefile targets

| Target    | Description                                                         |
|-----------|---------------------------------------------------------------------|
| `all`     | Build a full toolchain with TCF scripts, aliases and nano libraries |
| `openocd` | Build and install OpenOCD                                           |
| `qemu`    | Build and install QEMU                                              |

## Options

### Required

| Option      | Values                                        |
|-------------|-----------------------------------------------|
| `--target=` | `arc`, `arc32`, `arc64`, `riscv32`, `riscv64` |

## Shared

| Option                           | Default                                   | Notes                                              |
|----------------------------------|-------------------------------------------|----------------------------------------------------|
| `--prefix=`                      | `/usr/local`                              | Install root                                       |
| `--with-vendor=`                 | `gf`                                      | GNU triplet vendor field                           |
| `--with-os=`                     | `baremetal`                               | `baremetal` or `linux` (linux not implemented yet) |
| `--with-libc=`                   | `picolibc` if baremetal, `glibc` if linux | `picolibc` / `newlib` (baremetal), `glibc` (linux) |
| `--enable-multilib`              | disabled                                  |                                                    |
| `--enable-gdb` / `--disable-gdb` | enabled                                   |                                                    |
| `--with-cmodel=`                 | RISC-V: `medlow`; ARC: none               |                                                    |
| `--with-*-src=`                  | `src/<component>`                         | Per-component source path                          |

### ARC Classic

| Option                  | Default  | Notes                                                     |
|-------------------------|----------|-----------------------------------------------------------|
| `--with-endian=`        | `little` | `big` rewrites the CPU to `arceb` / `arc32eb` / `arc64eb` |
| `--with-cpu=`           |          | e.g. `archs`, `hs6x`                                      |
| `--with-fpu=`           |          | `fpus`, `fpud`                                            |
| `--with-multilib-list=` |          | `reduced` or `release`; `--target=arc` only               |

### ARC-V

| Option                       | Default             | Notes                          |
|------------------------------|---------------------|--------------------------------|
| `--with-arch=`               | `rv32gc` / `rv64gc` | Not checked against `--target` |
| `--with-abi=`                | derived from arch   |                                |
| `--with-tune=`               |                     |                                |
| `--with-isa-spec=`           | `20191213`          |                                |
| `--with-multilib-generator=` |                     | Implies `--enable-multilib`    |

RISC-V is little-endian only (`--with-endian=big` is an error). After install,
`*-unknown-elf-*` symlinks are created unless `--with-vendor=unknown`.
