# call stack
## hierarchy
```
- Top >
    - iconnet/bus >
        - tlm_utils::simple_target_socket_tagged
        - tlm_utils::simple_initiator_socket_tagged
        - memmap_entry >
    - xilinx_zynq
        - tlm_utils::simple_initiator_socket
        - tlm_utils::simple_target_socket
        - remoteport_tlm_wires
        - remoteport_tlm_memory_slave
        - remoteport_tlm_memory_master
            - remoteport_tlm >
                - remoteport_tlm_dev >
                    - remoteport_packet >
                        - rp_pkt >
                - rp_peer_state >
                - Iremoteport_tlm_sync
```
- systemctlm-cosim-demo/zynq_demo.cc, SC_MODULE(Top)
```
	iconnect<NR_MASTERS, NR_DEVICES> bus;
	xilinx_zynq zynq;
```
- systemctlm-cosim-demo/libsystemctlm-soc/soc/xilinx/zynq/xilinx-zynq.h, SC_MODULE(Top)
```
class xilinx_zynq: public remoteport_tlm
```
- systemctlm-cosim-demo/libsystemctlm-soc/libremote-port/remote-port-tlm.h
```
class remoteport_packet
class remoteport_tlm_dev
class Iremoteport_tlm_sync
class remoteport_tlm
```
- systemctlm-cosim-demo/libsystemctlm-soc/libremote-port/remote-port-proto.h
```
enum rp_cmd
struct rp_cfg_state
struct rp_pkt_hdr
struct rp_pkt_cfg
struct rp_capabilities
struct rp_pkt_hello
struct rp_pkt_busaccess
struct rp_pkt_busaccess_ext_base
struct rp_pkt_interrupt
struct rp_pkt_sync
struct rp_pkt_ats
struct rp_pkt
struct rp_peer_state
```

## summary
- b_transport based programming style
- transaction extended from tlm_generic_payload
```
systemctlm-cosim-demo/libsystemctlm-soc/libremote-port/remote-port-proto.c
enum rp_cmd {
    RP_CMD_nop         = 0,
    RP_CMD_hello       = 1,
    RP_CMD_cfg         = 2,
    RP_CMD_read        = 3,
    RP_CMD_write       = 4,
    RP_CMD_interrupt   = 5,
    RP_CMD_sync        = 6,
    RP_CMD_ats_req     = 7,
    RP_CMD_ats_inv     = 8,
    RP_CMD_max         = 8
};

struct rp_pkt {
    union {
        struct rp_pkt_hdr hdr;
        struct rp_pkt_hello hello;
        struct rp_pkt_busaccess busaccess;
        struct rp_pkt_busaccess_ext_base busaccess_ext_base;
        struct rp_pkt_interrupt interrupt;
        struct rp_pkt_sync sync;
        struct rp_pkt_ats ats;
    };
};

struct rp_pkt_cfg {
    struct rp_pkt_hdr hdr;
    uint32_t opt;
    uint8_t set;
} PACKED;

struct rp_pkt_hello {
    struct rp_pkt_hdr hdr;
    struct rp_version version;
    struct rp_capabilities caps;
} PACKED;

struct rp_pkt_interrupt {
    struct rp_pkt_hdr hdr;
    uint64_t timestamp;
    uint64_t vector;
    uint32_t line;
    uint8_t val;
} PACKED;

struct rp_pkt_sync {
    struct rp_pkt_hdr hdr;
    uint64_t timestamp;
} PACKED;

struct rp_pkt_busaccess {
    struct rp_pkt_hdr hdr;
    uint64_t timestamp;
    uint64_t attributes;
    uint64_t addr;

    /* Length in bytes.  */
    uint32_t len;

    /* Width of each beat in bytes. Set to zero for unknown (let the remote
       side choose).  */
    uint32_t width;

    /* Width of streaming, must be a multiple of width.
       addr should repeat itself around this width. Set to same as len
       for incremental (normal) accesses.  In bytes.  */
    uint32_t stream_width;

    /* Implementation specific source or master-id.  */
    uint16_t master_id;
} PACKED;

enum {
    /* Remote port responses. */
    RP_RESP_OK                  =  0x0,
    RP_RESP_BUS_GENERIC_ERROR   =  0x1,
    RP_RESP_ADDR_ERROR          =  0x2,
    RP_RESP_MAX                 =  0xF,
};

struct rp_peer_state {
    void *opaque;

    struct rp_pkt pkt;
    bool hdr_used;

    struct rp_version version;

    struct {
        bool busaccess_ext_base;
        bool busaccess_ext_byte_en;
        bool wire_posted_updates;
        bool ats;
    } caps;

    /* Used to normalize our clk.  */
    int64_t clk_base;

    struct rp_cfg_state local_cfg;
    struct rp_cfg_state peer_cfg;
};
```
- process
```
    1. get current handle
    2. sync reset
    3. wait negedge event
    4. socket open
    5. create pthread
    6. while (1) loop-1
        6.1 alloc receive buffer
        6.2 while (1) loop-2
            6.2.1 wait packet event
            6.2.2 pthread_mutex_lock
            6.2.3 read packet header
            6.2.4 decode packet header
            6.2.5 read packet payload
            6.2.6 pthread_mutex_unlock
            6.2.7 decode packet payload
            6.2.8 check response flag
                6.2.8.1 if posted, drop
                6.2.8.2 if unhandled, assert
                6.2.8.3 post_any_cmd
                6.2.8.3 if not response, continue
            6.2.9 command switch-case process
                6.2.9.1 receive only: hello, read, ats_inv
                6.2.9.2 receive then response: write,interrupt,sync
            6.2.10 post_any_cmd
    7. sync
```