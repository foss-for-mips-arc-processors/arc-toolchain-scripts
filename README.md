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
# Tools: arceb-gf-elf-*, alias arceb-elf32-*
./configure --target=arceb --with-libc=newlib --enable-multilib --prefix=$PWD/install

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

| Target                     | Description                                                              |
|----------------------------|--------------------------------------------------------------------------|
| `all`                      | Build a full toolchain with TCF scripts and aliases                      |
| `check-gcc-baremetal`      | Run the GCC DejaGnu testsuite                                            |
| `check-binutils-baremetal` | Run the binutils, gas and ld testsuites                                  |
| `check-gdb-baremetal`      | Run the GDB testsuite                                                    |
| `check-newlib-baremetal`   | Run the Newlib testsuite                                                 |
| `check-picolibc-baremetal` | Run the Picolibc tests                                                   |
| `check-libc-baremetal`     | Run the check for the configured `--with-libc=` (`newlib` or `picolibc`) |
| `report`                   | Summarize DejaGnu and Picolibc test counts                               |
| `report-filtered`          | Same results with the family allowlist applied                           |
| `openocd`                  | Build and install OpenOCD                                                |
| `qemu`                     | Build and install QEMU                                                   |

### Testing

The `check-*` targets build the toolchain pieces they need, then run the
testsuite on the simulator chosen with `--with-sim=nsim|qemu` (default `nsim`).
Selecting `qemu` also builds QEMU. DejaGnu is built automatically.

- A failing testsuite does not make the target fail. The target is recorded as
  done in `stamps/`, so a repeat run does nothing until you remove the stamp
  (for example `rm stamps/check-gcc-baremetal`) or run `make clean`.
- Results are written to the `.sum` and `.log` files in the matching
  `build-*-baremetal` directory (Picolibc: `build-picolibc-baremetal-tests/test.log`).
- `report` and `report-filtered` run the matching `check-*` targets first and
  then print the counts. `report` covers gcc, binutils, gdb and the configured libc.
  `report-filtered` covers gcc, binutils and, for `newlib`, newlib.
- Pass extra DejaGnu arguments with `make check-gcc-baremetal RUNTESTFLAGS="dg.exp=<test>.c"`.
  The board is `DEJAGNU_BOARD_NAME` (default `general-sim`). The optional
  `DEJAGNU_BOARD_MULTILIB` is appended as `/<value>` to form the
  `--target_board=` value, for example
  `make check-gcc-baremetal DEJAGNU_BOARD_MULTILIB=-march=rv32imac/-mabi=ilp32`
  runs on `general-sim/-march=rv32imac/-mabi=ilp32`. Flags are separated by `/`.

For example, this is how I am going to test HS5x from a multilib ARCv3 toolchain:

```bash
./configure --target=arc64 --enable-multilib --disable-gdb --disable-nano --with-languages=c --with-sim=nsim --prefix=$PWD/install
make -j 4
make check-gcc-baremetal RUNTESTFLAGS=arc64.exp DEJAGNU_BOARD_MULTILIB=-mcpu=hs5x
```

## Options

### Required

| Option      | Values                                                 |
|-------------|--------------------------------------------------------|
| `--target=` | `arc`, `arceb`, `arc32`, `arc64`, `riscv32`, `riscv64` |

## Shared

| Option                                   | Default           | Notes                                         |
|------------------------------------------|-------------------|-----------------------------------------------|
| `--prefix=`                              | `/usr/local`      | Install root                                  |
| `--with-vendor=`                         | `gf`              | GNU triplet vendor field                      |
| `--with-os={baremetal,linux}`            | `baremetal`       |                                               |
| `--with-libc={picolibc,newlib}`          | `picolibc`        |                                               |
| `--enable-multilib`                      | disabled          |                                               |
| `--with-multilib-list={reduced,release}` | disabled          |                                               |
| `--enable-gdb`                           | enabled           | Build GDB                                     |
| `--enable-nano`                          | disabled          | Build nano libc                               |
| `--enable-openocd`                       | disabled          | Build OpenOCD                                 |
| `--enable-qemu`                          | disabled          | Build QEMU                                    |
| `--with-languages=`                      | `c,c++`           | GCC stage 2 `--enable-languages`              |
| `--with-*-src=`                          | `src/<component>` | Per-component source path                     |
| `--with-sim={nsim,qemu}`                 | `nsim`            | Use a chosen simulator for testing by default |

### ARC Classic

| Option        | Default | Notes                |
|---------------|---------|----------------------|
| `--with-cpu=` |         | e.g. `archs`, `hs6x` |
| `--with-fpu=` |         | `fpus`, `fpud`       |

### ARC-V

| Option                       | Default             | Notes                       |
|------------------------------|---------------------|-----------------------------|
| `--with-arch=`               | `rv32gc` / `rv64gc` |                             |
| `--with-abi=`                | derived from arch   |                             |
| `--with-tune=`               |                     |                             |
| `--with-isa-spec=`           | `20191213`          |                             |
| `--with-cmodel=`             | `medlow`            |                             |
| `--with-multilib-generator=` |                     | Implies `--enable-multilib` |

After install, `*-unknown-elf-*` symlinks are created unless `--with-vendor=unknown`.
