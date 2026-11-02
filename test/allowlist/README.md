# Testsuite allowlists

`make report-filtered` drops known DejaGnu results listed here. ARC Classic and ARC-V use different layouts and scripts.

Log files use the same line format in both trees:

- Lines starting with `#` are comments.
- Empty lines are ignored.
- A result line is `STATUS: test.name ...`.
- Status values are `FAIL`, `XPASS`, `ERROR`, and `UNRESOLVED`.

```
#
# Known failures
#
FAIL: gcc.dg/pr1.c (test for excess errors)
XPASS: gcc.dg/pr2.c scan-assembler pattern
```

## ARC Classic

`scripts/testsuite-filter-arc` reads one CPU directory. `make report-filtered` picks it from `--target` (`archs` for `arc`, `hs5x` for `arc32`, `hs6x` for `arc64`) or from `--with-cpu` when that is set.

```
arc/<tool>/<cpu>/common.log
arc/<tool>/<cpu>/<libc>.log
```

`<tool>` is `gcc`, `binutils`, or `qemu`. `<libc>` is `newlib` or `glibc`. `common.log` applies to every C library. Both files are loaded when they exist.

```
test/allowlist/arc/
├── gcc/
│   ├── archs/
│   │   ├── common.log
│   │   ├── newlib.log
│   │   └── glibc.log
│   └── hs6x/
├── binutils/
└── qemu/
```

```bash
scripts/testsuite-filter-arc gcc newlib test/allowlist/arc/gcc/archs/ gcc.sum,g++.sum
```

## ARC-V

`scripts/testsuite-filter-riscv` selects files from the architecture, ABI, and C library. A non-multilib board name has no `-march`, so `make report-filtered` passes `--with-arch` and `--with-abi`. Missing files are skipped.

```
riscv/<tool>/common.log
riscv/<tool>/[<libc>.][rv32|rv64|rv128.][<extension>.][<abi>.][<tune>.]log
```

`<tool>` is `gcc` or `binutils`. `common.log` applies to every configuration. `<libc>` includes `newlib`, `newlib-nano`, `picolibc`, and `glibc`. Tune names such as `arc-v-rmx-100-series` are matched the same way as an extension.

For `rv32im` / `ilp32` / `newlib`, the filter looks for `common.log`, `newlib.log`, `rv32.log`, `ilp32.log`, and the same names combined with each extension (`i`, `m`) and with the C library. The same pattern is used for `rv64`.

```bash
REPORT_ARCH=rv64gc REPORT_ABI=lp64d \
  scripts/testsuite-filter-riscv gcc picolibc test/allowlist/riscv gcc.sum,g++.sum
```
