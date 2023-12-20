# project_b
qemu based systemc co-simulation

## virtualization exploration
  - IA-ISS: Instruction accurated, QEMU-TCG, dynamic binary translate engine, single thread, multi-arch
      - Multi-thread enhancement: MTTCG ?
  - Co-simulation interface:
      - PetaLinux by Xilinx: Remote-Port, tlm_generic_payload with b_transport enhancement, need to be initiated by DTS under Linux, not avaliable for bare-metal
      - OCX ?
      - FIFO based: Qemu FIFO/Chardev backend + tlm_fifo utilization
      - PCIe based?
      - VirtIO-MMIO based?
      - AMBA based?
  - IO virtualization: VirtIO ?
  - Accelerate virtualization
    - Nvdia virtualGPU through mode ?
    - generic NPU ?
  - Memory virtualization:
    - DRAMSim ?
  - Interconnect virtualizaton ?
    - MMIO to mimic AMBA ?
