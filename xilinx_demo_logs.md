# build qemu
```
cd qemu/build;
../configure --target-list=aarch64-softmmu --enable-fdt --disable-kvm --disable-xen
```

# toolchain
```
https://releases.linaro.org/components/toolchain/binaries/latest-7/aarch64-linux-gnu/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar.xz
https://releases.linaro.org/components/toolchain/binaries/latest-7/aarch64-linux-gnu/sysroot-glibc-linaro-2.25-2019.12-aarch64-linux-gnu.tar.xz
```

# build zynq_demo
- vi systemctlm-cosim-demo/.config.mk
    ```
    HAVE_VERILOG=n
    HAVE_VERILOG_VERILATOR=n
    HAVE_VERILOG_VCS=n
    ```
- make zynq_demo

# buildroot building
```
git clone https://github.com/buildroot/buildroot.git
cd buildroot
git checkout 36edacce9c2c3b90f9bb11
```
- Pull the .dtsi files for the Zynq-7000
    ```
    wget https://raw.githubusercontent.com/Xilinx/qemu-devicetrees/master/zynq-pl-remoteport.dtsi
    wget https://raw.githubusercontent.com/Xilinx/linux-xlnx/master/arch/arm/boot/dts/zynq-7000.dtsi
    ```
- Get zynq-zc702.dts for kernel 5.4.75
    ```
    wget https://raw.githubusercontent.com/torvalds/linux/6e97ed6efa701db070da0054b055c085895aba86/arch/arm/boot/dts/zynq-zc702.dts
    ```
- Include zynq-pl-remoteport.dtsi in zynq-zc702.dts
    ```
    echo "#include \"zynq-pl-remoteport.dtsi\"" >> zynq-zc702.dts
    ```
- Fetch xilinx_zynq_defconfig
    ```
    wget https://raw.githubusercontent.com/Xilinx/linux-xlnx/3fbb3a6c57a43342a7daec3bbae2f595c50bc969/arch/arm/configs/xilinx_zynq_defconfig
    ```
- Setup the Buildroot compilation configeration
    - vi .config
    ```
    BR2_arm=y
    BR2_cortex_a9=y
    BR2_ARM_ENABLE_NEON=y
    BR2_ARM_ENABLE_VFP=y
    BR2_TOOLCHAIN_EXTERNAL=y
    BR2_TOOLCHAIN_EXTERNAL_BOOTLIN=y
    BR2_TOOLCHAIN_EXTERNAL_BOOTLIN_ARMV7_EABIHF_GLIBC_STABLE=y
    BR2_TARGET_GENERIC_GETTY_PORT="ttyPS0"
    BR2_LINUX_KERNEL=y
    BR2_LINUX_KERNEL_CUSTOM_VERSION=y
    BR2_LINUX_KERNEL_CUSTOM_VERSION_VALUE="5.4.75"
    BR2_LINUX_KERNEL_USE_CUSTOM_CONFIG=y
    BR2_LINUX_KERNEL_CUSTOM_CONFIG_FILE="xilinx_zynq_defconfig"
    BR2_LINUX_KERNEL_UIMAGE=y
    BR2_LINUX_KERNEL_UIMAGE_LOADADDR="0x8000"
    BR2_LINUX_KERNEL_DTS_SUPPORT=y
    BR2_LINUX_KERNEL_CUSTOM_DTS_PATH="zynq-7000.dtsi  zynq-zc702.dts zynq-pl-remoteport.dtsi"
    BR2_TARGET_ROOTFS_CPIO=y
    BR2_TARGET_ROOTFS_CPIO_GZIP=y
    BR2_TARGET_ROOTFS_EXT2=y
    #BR2_TARGET_ROOTFS_TAR is not set
    ```
# fix error in br.log
-  vi output/build/host-fakeroot-1.25.3/libfakeroot.c
    ```
    # define _STAT_VER 0
    ```
- vi output/build/host-m4-1.4.18/lib/c-stack.c
    ```
    define SIGSTKSZ 16384
    ```
- make clean & make olddefconfig
- ./utils/brmake
# Run co-simulation
- terminal for systemc
    ```
    ./systemctlm-cosim-demo/zynq_demo unix:machine_path/qemu-rport-_cosim@0 1000000
    ```
- terminal for qemu
    ```
    ./qemu/build/qemu-system-aarch64 -M arm-generic-fdt-7series -m 1G -kernel buildroot/output/images/uImage -dtb buildroot/output/images/zynq-zc702.dtb --initrd buildroot/output/images/rootfs.cpio.gz -serial /dev/null -serial mon:stdio -display none -net nic -net user -machine-path machine_path/ -icount 0,sleep=off -rtc clock=vm -sync-quantum 1000000 -monitor telnet:127.0.0.1:3333,server,nowait
    ```

# check status
- zynq_demo backtrace
```
#0  __futex_abstimed_wait_common64 (private=0, cancel=true, abstime=0x0, op=393, expected=0, futex_word=0x7f4113fc8f34 <sc_core::main_cor+116>) at ./nptl/futex-internal.c:57
#1  __futex_abstimed_wait_common (cancel=true, private=0, abstime=0x0, clockid=0, expected=0, futex_word=0x7f4113fc8f34 <sc_core::main_cor+116>) at ./nptl/futex-internal.c:87
#2  __GI___futex_abstimed_wait_cancelable64 (futex_word=futex_word@entry=0x7f4113fc8f34 <sc_core::main_cor+116>, expected=expected@entry=0, clockid=clockid@entry=0, abstime=abstime@entry=0x0, private=private@entry=0)
    at ./nptl/futex-internal.c:139
#3  0x00007f41139f6ac1 in __pthread_cond_wait_common (abstime=0x0, clockid=0, mutex=0x7f4113fc8ed8 <sc_core::main_cor+24>, cond=0x7f4113fc8f08 <sc_core::main_cor+72>) at ./nptl/pthread_cond_wait.c:503
#4  ___pthread_cond_wait (cond=0x7f4113fc8f08 <sc_core::main_cor+72>, mutex=0x7f4113fc8ed8 <sc_core::main_cor+24>) at ./nptl/pthread_cond_wait.c:627
#5  0x00007f4113e8afb2 in sc_core::sc_cor_pkg_pthread::yield (this=<optimized out>, next_cor_p=0x563b970d10c0) at ../../../src/sysc/kernel/sc_cor_pthread.cpp:254
#6  0x00007f4113ea2e0c in sc_core::sc_simcontext::crunch (this=this@entry=0x563b97012eb0, once=once@entry=false) at ../../../src/sysc/kernel/sc_simcontext.cpp:501
#7  0x00007f4113ea215a in sc_core::sc_simcontext::simulate (this=this@entry=0x563b97012eb0, duration=...) at ../../../src/sysc/kernel/sc_simcontext.cpp:889
#8  0x00007f4113ea249a in sc_core::sc_start (duration=..., p=sc_core::SC_EXIT_ON_STARVATION) at ../../../src/sysc/kernel/sc_simcontext.cpp:1710
#9  0x00007f4113ea270a in sc_core::sc_start () at ../../../src/sysc/kernel/sc_simcontext.cpp:1744
#10 0x0000563b9637c3e6 in sc_main (argc=<optimized out>, argv=0x563b9703fb00) at zynq_demo.cc:125
#11 0x00007f4113e8dec5 in sc_core::sc_elab_and_sim (argc=3, argv=<optimized out>) at ../../../src/sysc/kernel/sc_main_main.cpp:87
#12 0x00007f411398cd90 in __libc_start_call_main (main=main@entry=0x7f4113e85520 <main(int, char**)>, argc=argc@entry=3, argv=argv@entry=0x7ffe97503a28) at ../sysdeps/nptl/libc_start_call_main.h:58
#13 0x00007f411398ce40 in __libc_start_main_impl (main=0x7f4113e85520 <main(int, char**)>, argc=3, argv=0x7ffe97503a28, init=<optimized out>, fini=<optimized out>, rtld_fini=<optimized out>, stack_end=0x7ffe97503a18)
    at ../csu/libc-start.c:392
#14 0x0000563b9637bef5 in _start ()
```
- qemu_host backtrace
```
#0  futex_wait (private=0, expected=2, futex_word=0x56004ad73320 <qemu_global_mutex>) at ../sysdeps/nptl/futex-internal.h:146
#1  __GI___lll_lock_wait (futex=futex@entry=0x56004ad73320 <qemu_global_mutex>, private=0) at ./nptl/lowlevellock.c:49
#2  0x00007f57ef7cf082 in lll_mutex_lock_optimized (mutex=0x56004ad73320 <qemu_global_mutex>) at ./nptl/pthread_mutex_lock.c:48
#3  ___pthread_mutex_lock (mutex=mutex@entry=0x56004ad73320 <qemu_global_mutex>) at ./nptl/pthread_mutex_lock.c:93
#4  0x000056004a0b6358 in qemu_mutex_lock_impl (mutex=0x56004ad73320 <qemu_global_mutex>, file=0x56004a3b0287 "../util/main-loop.c", line=318) at ../util/qemu-thread-posix.c:88
#5  0x0000560049ea6c0e in qemu_mutex_lock_iothread_impl (file=file@entry=0x56004a3b0287 "../util/main-loop.c", line=line@entry=318) at ../softmmu/cpus.c:502
#6  0x000056004a0d382e in os_host_main_loop_wait (timeout=1000000000) at ../util/main-loop.c:318
#7  main_loop_wait (nonblocking=nonblocking@entry=0) at ../util/main-loop.c:596
#8  0x0000560049b52963 in qemu_main_loop () at ../softmmu/runstate.c:734
#9  0x0000560049880160 in qemu_main (argc=<optimized out>, argv=<optimized out>, envp=<optimized out>) at ../softmmu/main.c:38
#10 0x00007f57ef760d90 in __libc_start_call_main (main=main@entry=0x560049878070 <main>, argc=argc@entry=31, argv=argv@entry=0x7ffca93f9138) at ../sysdeps/nptl/libc_start_call_main.h:58
#11 0x00007f57ef760e40 in __libc_start_main_impl (main=0x560049878070 <main>, argc=31, argv=0x7ffca93f9138, init=<optimized out>, fini=<optimized out>, rtld_fini=<optimized out>, stack_end=0x7ffca93f9128)
    at ../csu/libc-start.c:392
#12 0x0000560049880085 in _start ()
```
- netstat -a
```
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 localhost:32857         0.0.0.0:*               LISTEN
tcp        0      0 localhost:3333          0.0.0.0:*               LISTEN
```