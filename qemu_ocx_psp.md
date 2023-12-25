# [Unicorn_OCX](https://github.com/snps-virtualprototyping/ocx-qemu-arm)
## 1. Building hierarchy
```
ocx-qemu-arm
- ocx/src/ocx-runner.cpp
    int main(int argc, char** argv) {
        corelib cl(ocx_lib_path);
        ocx::memory mem(memsize, 0x1000);
        ocx::runenv env(mem);
        ocx::core* core = cl.create_core(env, ocx_variant);
        threads.emplace_back(run_core, core, quantum, 0);
        thread.join();
    }
    - ocx/src/{arch}core.cpp  
        core::core(env &env, const model* modl) {
            uc_open(m_model->name, this, &helper_config, &m_uc);
            uc_mem_map_io(m_uc, 0, ADDR_SIZE, &helper_transport, this);
            uc_mem_map_portio(m_uc, &helper_transport, this);
            uc_register_tlb_cluster(m_uc, this, &helper_tlb_cluster_flush,
                                      &helper_tlb_cluster_flush_page,
                                      &helper_tlb_cluster_flush_mmuidx,
                                      &helper_tlb_cluster_flush_page_mmuidx);
        }
        core::helper_transport(uc_engine* uc, void* opaque, uc_mmio_tx_t* tx) {
            cpu->m_env.transport(xt);
        }
    - unicorn/uc.c
        uc_open(const char* model, void *cfg_opaque, uc_get_config_t cfg_func,
               uc_engine **result) // SNPS changed
    - unicorn/qemu/target/riscv/unicorn.c
        void riscv64_uc_init(struct uc_struct *uc) {
            spike_v1_10_0_machine_init_register_types(uc);
        }
```
## 2. APIs for Unicorn
[unicorn design](https://www.unicorn-engine.org/BHUSA2015-unicorn.pdf)
```
Support various levels of instrumentation:
    - Single-step or o particular instruction (TCG level)
    - Intrumentation of memory accesses (TLB level)
    - Dynamically read and write register or memory during emulation.
    - Handle exception, interrupt, syscall (arch-level) through user provided callback.
```

# [OCX_PSP]

# Native Qemu interface
## 1. TCG enegine
## 2. MTTCG enhancement
## 3. VirtIO-MMIO
## 4. VirtGPU for GUI
## 5. interconnect mimic

# Explorations TODOs
- [ ] MTTCG
- [ ] TLB memory accesss
- [ ] MMIO-VirtIO