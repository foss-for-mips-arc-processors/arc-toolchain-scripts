## Process this file with autoconf. Other versions than 2.69 may not work.
AC_INIT([arc-toolchains],[1.0])

AC_PROG_CC
AC_PROG_FGREP
AC_PROG_GREP

AC_PATH_PROG([BASH], [bash],
	[AC_MSG_ERROR([bash not found])])

AC_PATH_PROGS_FEATURE_CHECK([GAWK], [gawk awk],
	[AS_IF([$ac_path_GAWK --version 2>&1 | $FGREP GNU >/dev/null],
		[ac_cv_path_GAWK=$ac_path_GAWK ac_path_GAWK_found=:])],
	[AC_MSG_ERROR([GNU Awk not found])])
AC_SUBST([GAWK], [$ac_cv_path_GAWK])

AC_PATH_PROGS_FEATURE_CHECK([GSED], [gsed sed],
	[AS_IF([$ac_path_GSED --version 2>&1 | $FGREP GNU >/dev/null],
		[ac_cv_path_GSED=$ac_path_GSED ac_path_GSED_found=:])],
	[AC_MSG_ERROR([GNU sed not found])])
AC_SUBST([GSED], [$ac_cv_path_GSED])

need_gcc_external_libraries="no"
AC_CHECK_LIB(gmp, __gmpz_init, ,
        [need_gcc_external_libraries="yes"])
AC_CHECK_LIB(mpfr, mpfr_init, ,
        [need_gcc_external_libraries="yes"])
AC_CHECK_LIB(mpc, mpc_init2, ,
        [need_gcc_external_libraries="yes"])
AS_IF([test x"$need_gcc_external_libraries" != xno],
      [AC_SUBST(NEED_GCC_EXTERNAL_LIBRARIES,true)],
      [AC_SUBST(NEED_GCC_EXTERNAL_LIBRARIES,false)])

AC_PATH_PROG([CURL], [curl], [no])
AC_PATH_PROG([WGET], [wget], [no])
AC_PATH_PROG([FTP], [ftp], [no])
AS_IF([test x"$CURL" != xno], [FETCHER="$CURL -L -o - --ftp-pasv --retry 10"],
	[test x"$WGET" != xno], [FETCHER="$WGET -O -"],
	[test x"$FTP" != xno], [FETCHER="$FTP -o - -p"],
	[FETCHER=""])
AC_SUBST(FETCHER)

#
# --target selects the architecture family. It is required at build time.
# Allowed values: arc, arc32, arc64, riscv32, riscv64.
# For RISC-V this is only the CPU field of the GNU triplet; it is not
# checked against --with-arch.
#
family=
AS_IF([test "x$target_alias" != x],
	[AS_CASE([$target_alias],
		[arc|arc32|arc64], [family=arc],
		[riscv32|riscv64], [family=riscv],
		[AC_MSG_ERROR([--target must be arc, arc32, arc64, riscv32, or riscv64 (got: $target_alias)])])])
AC_SUBST(family)
AC_SUBST(target_cpu, [$target_alias])

AC_ARG_WITH(os,
	[AS_HELP_STRING([--with-os=OS],
		[Target OS: baremetal or linux @<:@baremetal@:>@])],
	[],
	[with_os=baremetal])
AS_CASE([$with_os],
	[baremetal|linux], [],
	[AC_MSG_ERROR([--with-os must be baremetal or linux])])

AC_ARG_WITH(libc,
	[AS_HELP_STRING([--with-libc=LIBC],
		[C library: picolibc or newlib (baremetal), glibc (linux). Default picolibc if --with-os=baremetal, glibc if --with-os=linux])],
	[],
	[with_libc=])
AS_IF([test "x$with_libc" = x],
	[AS_IF([test "x$with_os" = xlinux],
		[with_libc=glibc],
		[with_libc=picolibc])])
AS_IF([test "x$with_os" = xbaremetal],
	[AS_CASE([$with_libc],
		[newlib|picolibc], [],
		[AC_MSG_ERROR([--with-os=baremetal requires --with-libc=picolibc or newlib])])],
	[AS_CASE([$with_libc],
		[glibc], [],
		[AC_MSG_ERROR([--with-os=linux requires --with-libc=glibc])])
	 AC_MSG_WARN([Linux toolchains (--with-os=linux) are not implemented yet])])
AC_SUBST(with_os)
AC_SUBST(libc, [$with_libc])
AC_SUBST(default_target, [$with_libc])

AC_ARG_WITH(vendor,
	[AS_HELP_STRING([--with-vendor=gf],
		[Vendor field of the GNU triplet @<:@gf@:>@])],
	[],
	[with_vendor=gf])
AC_SUBST(vendor, [$with_vendor])

AC_ARG_WITH(endian,
	[AS_HELP_STRING([--with-endian=little|big],
		[ARC endianness (RISC-V is little-endian only) @<:@little@:>@])],
	[],
	[with_endian=little])
AS_CASE([$with_endian],
	[little|big], [],
	[AC_MSG_ERROR([--with-endian must be little or big])])

AC_ARG_WITH(cpu,
	[AS_HELP_STRING([--with-cpu=CPU],
		[ARC default CPU (e.g. archs, hs6x); ARC only])],
	[],
	[with_cpu=])

AC_ARG_WITH(fpu,
	[AS_HELP_STRING([--with-fpu=FPU],
		[ARC default FPU (fpus, fpud); ARC only])],
	[],
	[with_fpu=])

AC_ARG_WITH(arch,
	[AS_HELP_STRING([--with-arch=rv64gc],
		[RISC-V ISA string; RISC-V only])],
	[],
	[with_arch=])

AC_ARG_WITH(abi,
	[AS_HELP_STRING([--with-abi=lp64d],
		[RISC-V ABI; RISC-V only, default derived from --with-arch])],
	[],
	[with_abi=])

AC_ARG_WITH(tune,
	[AS_HELP_STRING([--with-tune=CPU],
		[RISC-V -mtune CPU; RISC-V only])],
	[],
	[with_tune=])

AC_ARG_WITH(isa-spec,
	[AS_HELP_STRING([--with-isa-spec=20191213],
		[RISC-V ISA spec (2.2, 20190608, 20191213); RISC-V only @<:@20191213@:>@])],
	[],
	[with_isa_spec=])

AC_ARG_ENABLE(multilib,
	[AS_HELP_STRING([--enable-multilib],
		[Build all configured runtime libraries @<:@--disable-multilib@:>@])],
	[],
	[enable_multilib=no])

AC_ARG_WITH(multilib-list,
	[AS_HELP_STRING([--with-multilib-list=LIST],
		[Classic ARC GCC multilib list (reduced, release); --target=arc only])],
	[],
	[with_multilib_list=])

AC_ARG_WITH(multilib-generator,
	[AS_HELP_STRING([--with-multilib-generator=GEN],
		[RISC-V bare-metal multilib generator string; implies --enable-multilib])],
	[],
	[with_multilib_generator=])

AC_ARG_ENABLE(gdb,
	[AS_HELP_STRING([--disable-gdb],
		[Do not build GDB @<:@--enable-gdb@:>@])],
	[],
	[enable_gdb=yes])
AS_IF([test "x$enable_gdb" != xno],
	[AC_SUBST(enable_gdb, --enable-gdb)],
	[AC_SUBST(enable_gdb, --disable-gdb)])

AC_ARG_ENABLE(werror,
	[AS_HELP_STRING([--enable-werror],
		[Build with -Werror @<:@--disable-werror@:>@])],
	[],
	[enable_werror=no])
AS_IF([test "x$enable_werror" != xno],
	[AC_SUBST(werror_flag, --enable-werror)],
	[AC_SUBST(werror_flag, --disable-werror)])

AC_ARG_ENABLE(debug_info,
	[AS_HELP_STRING([--enable-debug-info],
		[Build target libraries with debug information @<:@--disable-debug-info@:>@])],
	[],
	[enable_debug_info=no])
AS_IF([test "x$enable_debug_info" != xno],
	[AC_SUBST(debug_info, [-g])],
	[AC_SUBST(debug_info, [])])

AC_ARG_ENABLE(gcc-checking,
	[AS_HELP_STRING([--enable-gcc-checking],
		[Enable GCC internal checking (slow) @<:@--disable-gcc-checking@:>@])],
	[],
	[enable_gcc_checking=no])
AS_IF([test "x$enable_gcc_checking" != xno],
	[AC_SUBST(gcc_checking, --enable-checking=yes)],
	[AC_SUBST(gcc_checking, --enable-checking=release)])

AC_ARG_WITH(cmodel,
	[AS_HELP_STRING([--with-cmodel],
		[Code model for libc and libgcc; RISC-V defaults to medlow])],
	[],
	[with_cmodel=])

AC_ARG_WITH(system-zlib,
	[AS_HELP_STRING([--without-system-zlib],
		[Use GCC's bundled zlib])],
	[],
	[with_system_zlib=yes])
AS_IF([test "x$with_system_zlib" != xno],
	[AC_SUBST(with_system_zlib, --with-system-zlib)],
	[AC_SUBST(with_system_zlib, --without-system-zlib)])

AC_ARG_WITH(host,
	[AS_HELP_STRING([--with-host=TRIPLE],
		[Host triple for the toolchain (e.g. mingw cross)])],
	[],
	[with_host=])
AS_IF([test "x$with_host" != x],
	[AC_SUBST(configure_host, [--host=$with_host])],
	[AC_SUBST(configure_host, [])])

AC_ARG_WITH(target_cflags,
	[AS_HELP_STRING([--with-target-cflags],
		[Extra C flags for target libraries])],
	[],
	[])
AC_SUBST(target_cflags, [$with_target_cflags])
AC_ARG_WITH(target_cxxflags,
	[AS_HELP_STRING([--with-target-cxxflags],
		[Extra C++ flags for target libraries])],
	[],
	[])
AC_SUBST(target_cxxflags, [$with_target_cxxflags])

AC_ARG_WITH(host_cflags,
	[AS_HELP_STRING([--with-host-cflags],
		[Extra C flags for host Binutils, GDB and GCC])],
	[],
	[])
AC_SUBST(host_cflags, [$with_host_cflags])
AC_ARG_WITH(host_cxxflags,
	[AS_HELP_STRING([--with-host-cxxflags],
		[Extra C++ flags for host Binutils, GDB and GCC])],
	[],
	[])
AC_SUBST(host_cxxflags, [$with_host_cxxflags])

AC_DEFUN([AX_ARG_WITH_SRC],
	[{m4_pushdef([opt_name], with_$1_src)
	  AC_ARG_WITH($1-src,
		[AS_HELP_STRING([--with-$1-src],[Set $1 source path, default $(srcdir)/src/$2])],
		[],
		[opt_name=default]
		)
	  AS_IF([test "x$opt_name" != xdefault],
		[AC_SUBST(opt_name,$opt_name)],
		[AC_SUBST(opt_name,"\$(srcdir)/src/$2")])
	  m4_popdef([opt_name])
	}])

AX_ARG_WITH_SRC(binutils, binutils)
AX_ARG_WITH_SRC(gdb, gdb)
AX_ARG_WITH_SRC(gcc, gcc)
AX_ARG_WITH_SRC(newlib, newlib)
AX_ARG_WITH_SRC(picolibc, picolibc)
AX_ARG_WITH_SRC(dejagnu, dejagnu)
AX_ARG_WITH_SRC(qemu, qemu)
AX_ARG_WITH_SRC(openocd, openocd)

AC_ARG_WITH(sim,
	[AS_HELP_STRING([--with-sim=nsim],
		[Select a simulator (nsim or qemu)])],
	[],
	[with_sim=nsim]
	)

#
# Family-specific validation and derived values.
#
with_cpu_flag=
with_fpu_flag=
triple_cpu=
target_alias_name=
WITH_ARCH=
WITH_ABI=
WITH_TUNE=
WITH_ISA_SPEC=
multilib_list_flags=
multilib_gen=

AS_IF([test "x$family" = xarc],
	[AS_IF([test "x$with_arch" != x],
		[AC_MSG_ERROR([--with-arch is only valid with --target=riscv32|riscv64])])
	 AS_IF([test "x$with_abi" != x],
		[AC_MSG_ERROR([--with-abi is only valid with --target=riscv32|riscv64])])
	 AS_IF([test "x$with_tune" != x],
		[AC_MSG_ERROR([--with-tune is only valid with --target=riscv32|riscv64])])
	 AS_IF([test "x$with_isa_spec" != x],
		[AC_MSG_ERROR([--with-isa-spec is only valid with --target=riscv32|riscv64])])
	 AS_IF([test "x$with_multilib_generator" != x],
		[AC_MSG_ERROR([--with-multilib-generator is only valid with --target=riscv32|riscv64])])
	 AS_IF([test "x$with_cpu" != x],
		[with_cpu_flag="--with-cpu=$with_cpu"],
		[with_cpu_flag=""])
	 AS_IF([test "x$with_fpu" != x],
		[with_fpu_flag="--with-fpu=$with_fpu"],
		[with_fpu_flag=""])
	 AS_IF([test "x$with_endian" = xbig],
		[AS_CASE([$target_alias],
			[arc], [triple_cpu=arceb],
			[arc32], [triple_cpu=arc32eb],
			[arc64], [triple_cpu=arc64eb],
			[triple_cpu=$target_alias])],
		[triple_cpu=$target_alias])
	 AS_CASE([$target_alias],
		[arc], [target_alias_name=${triple_cpu}-elf32],
		[target_alias_name=${triple_cpu}-elf])
	 AS_IF([test "x$with_multilib_list" != x],
		[AS_IF([test "x$target_alias" = xarc],
			[multilib_list_flags="--with-multilib-list=$with_multilib_list"
			 enable_multilib=yes],
			[AC_MSG_WARN([--with-multilib-list is only supported for --target=arc, ignoring])
			 multilib_list_flags=""])],
		[multilib_list_flags=""])
	 WITH_ARCH=
	 WITH_ABI=
	 WITH_TUNE=
	 WITH_ISA_SPEC=
	 multilib_gen=
	],
      [test "x$family" = xriscv],
	[AS_IF([test "x$with_endian" = xbig],
		[AC_MSG_ERROR([ARC-V/RISC-V toolchains are little-endian only])])
	 AS_IF([test "x$with_cpu" != x],
		[AC_MSG_ERROR([--with-cpu is only valid with --target=arc|arc32|arc64])])
	 AS_IF([test "x$with_fpu" != x],
		[AC_MSG_ERROR([--with-fpu is only valid with --target=arc|arc32|arc64])])
	 AS_IF([test "x$with_multilib_list" != x],
		[AC_MSG_ERROR([--with-multilib-list is only valid with --target=arc])])
	 with_cpu_flag=
	 with_fpu_flag=
	 triple_cpu=$target_alias
	 target_alias_name=${target_alias}-unknown-elf
	 AS_IF([test "x$with_arch" = x],
		[AS_IF([test "x$target_alias" = xriscv32],
			[with_arch=rv32gc],
			[with_arch=rv64gc])])
	 AS_IF([test "x$with_abi" = x],
		[AS_CASE([$with_arch],
			[*rv64g* | *rv64*d*], [with_abi=lp64d],
			[*rv64*f*], [with_abi=lp64f],
			[*rv64*], [with_abi=lp64],
			[*rv32e*], [with_abi=ilp32e],
			[*rv32g* | *rv32*d*], [with_abi=ilp32d],
			[*rv32*f*], [with_abi=ilp32f],
			[*rv32*], [with_abi=ilp32],
			[AC_MSG_ERROR([Cannot derive --with-abi from --with-arch=$with_arch; pass --with-abi])])])
	 AS_IF([test "x$with_isa_spec" = x], [with_isa_spec=20191213])
	 WITH_ARCH="--with-arch=$with_arch"
	 WITH_ABI="--with-abi=$with_abi"
	 AS_IF([test "x$with_tune" != x],
		[WITH_TUNE="--with-tune=$with_tune"],
		[WITH_TUNE=])
	 WITH_ISA_SPEC="--with-isa-spec=$with_isa_spec"
	 AS_IF([test "x$with_multilib_generator" != x],
		[enable_multilib=yes
		 multilib_gen=$with_multilib_generator],
		[multilib_gen=])
	 multilib_list_flags=
	])

AC_SUBST(with_cpu, [$with_cpu_flag])
AC_SUBST(with_fpu, [$with_fpu_flag])
AC_SUBST(WITH_ARCH)
AC_SUBST(WITH_ABI)
AC_SUBST(WITH_TUNE)
AC_SUBST(WITH_ISA_SPEC)
AC_SUBST(multilib_list_flags)
AC_SUBST(multilib_gen)
AC_SUBST(endian, [$with_endian])
AC_SUBST(triple_cpu)
AC_SUBST(target_alias_name)
AC_SUBST(WITH_SIM, $with_sim)

AS_IF([test "x$triple_cpu" != x],
	[target_triplet="${triple_cpu}-${with_vendor}-elf"],
	[target_triplet=])
AC_SUBST(target_triplet)

AS_IF([test "x$enable_multilib" != xno || test "x$with_multilib_generator" != x],
	[AC_SUBST(multilib_flags, --enable-multilib)],
	[AC_SUBST(multilib_flags, --disable-multilib)])

AS_IF([test "x$family" = xriscv],
	[AS_IF([test "x$with_cmodel" = x], [with_cmodel=medlow])
	 AS_IF([test "x$with_cmodel" = xmedlow],
		[cmodel_flag=],
		[cmodel_flag="-mcmodel=$with_cmodel"])],
	[AS_IF([test "x$with_cmodel" != x],
		[cmodel_flag="-mcmodel=$with_cmodel"],
		[cmodel_flag=])])
AC_SUBST(cmodel, [$cmodel_flag])

AC_CONFIG_FILES([Makefile])
AC_CONFIG_FILES([scripts/wrapper/awk/awk], [chmod +x scripts/wrapper/awk/awk])
AC_CONFIG_FILES([scripts/wrapper/sed/sed], [chmod +x scripts/wrapper/sed/sed])
AC_OUTPUT
