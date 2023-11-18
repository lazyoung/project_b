#build qemu
cd qemu/build; ../configure --target-list=aarch64-softmmu --enable-fdt --disable-kvm --disable-xen

#toolchain
https://releases.linaro.org/components/toolchain/binaries/latest-7/aarch64-linux-gnu/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar.xz
https://releases.linaro.org/components/toolchain/binaries/latest-7/aarch64-linux-gnu/sysroot-glibc-linaro-2.25-2019.12-aarch64-linux-gnu.tar.xz

#build zynq_demo
vi systemctlm-cosim-demo/.config.mk
```
HAVE_VERILOG=n
HAVE_VERILOG_VERILATOR=n
HAVE_VERILOG_VCS=n
```
make zynq_demo

#buildroot
git clone https://github.com/buildroot/buildroot.git
cd buildroot
git checkout 36edacce9c2c3b90f9bb11
# Pull the .dtsi files for the Zynq-7000
wget https://raw.githubusercontent.com/Xilinx/qemu-devicetrees/master/zynq-pl-remoteport.dtsi
wget https://raw.githubusercontent.com/Xilinx/linux-xlnx/master/arch/arm/boot/dts/zynq-7000.dtsi
# kernel 5.4.75 zynq-zc702.dts
wget https://raw.githubusercontent.com/torvalds/linux/6e97ed6efa701db070da0054b055c085895aba86/arch/arm/boot/dts/zynq-zc702.dts
# Include zynq-pl-remoteport.dtsi in zynq-zc702.dts
echo "#include \"zynq-pl-remoteport.dtsi\"" >> zynq-zc702.dts
# Fetch xilinx_zynq_defconfi
wget https://raw.githubusercontent.com/Xilinx/linux-xlnx/3fbb3a6c57a43342a7daec3bbae2f595c50bc969/arch/arm/configs/xilinx_zynq_defconfig
# Setup the Buildroot compilation configeration
vi .config
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
vi output/build/host-fakeroot-1.25.3/libfakeroot.c
```
#define _STAT_VER 0
```
vi output/build/host-m4-1.4.18/lib/c-stack.c
```
define SIGSTKSZ 16384
```
make clean & make olddefconfig
./utils/brmake

# terminal for systemc
./systemctlm-cosim-demo/zynq_demo unix:machine_path/qemu-rport-_cosim@0 1000000

# terminal for qemu
./qemu/build/qemu-system-aarch64 -M arm-generic-fdt-7series -m 1G -kernel buildroot/output/images/uImage -dtb buildroot/output/images/zynq-zc702.dtb --initrd buildroot/output/images/rootfs.cpio.gz -serial /dev/null -serial mon:stdio -display none -net nic -net user -machine-path machine_path/ -icount 0,sleep=off -rtc clock=vm -sync-quantum 1000000 -monitor telnet:127.0.0.1:3333,server,nowait
