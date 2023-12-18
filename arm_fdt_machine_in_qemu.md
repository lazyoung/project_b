# Code walk
## QOM object
- qemu-master/include/hw/remote-port-device.h
```
typedef struct RemotePortDeviceClass {
    /*< private >*/
    InterfaceClass parent_class;

    /*< public >*/
    /**
     * ops - operations to perform when remote port packets are recieved for
     * this device. Function N will be called for a remote port packet with
     * cmd == N in the header.
     *
     * @obj - Remote port device to recieve packet
     * @pkt - remote port packets
     */

    void (*ops[RP_CMD_max+1])(RemotePortDevice *obj, struct rp_pkt *pkt);

} RemotePortDeviceClass;
```
## Device interface
- qemu-master/include/hw/remote-port.h
```
typedef struct RemotePortRespSlot
struct RemotePort {
    DeviceState parent;
    QemuThread thread;
    Chardev *chrdev;
    CharBackend chr;
    struct rp_peer_state peer;

    union {
       int pipes[2];
       struct {
           int read;
           int write;
       } pipe;
    } event;

    struct {
        ptimer_state *ptimer;
        ptimer_state *ptimer_resp;
        bool resp_timer_enabled;
        bool need_sync;
        struct rp_pkt rsp;
        uint64_t quantum;
    } sync;

    struct {
        /* This array must be sized minimum 2 and always a power of 2.  */
        RemotePortDynPkt pkt[RX_QUEUE_SIZE];
        bool inuse[RX_QUEUE_SIZE];
        QemuSemaphore sem;
        unsigned int wpos;
        unsigned int rpos;
    } rx_queue;

    /*
     * rsp holds responses for the remote side.
     * Used by the slave.
     */
    RemotePortDynPkt rsp;

    /*
     * rspqueue holds received responses from the remote side.
     * Only one for the moment but it might grow.
     * Used by the master.
     */
    RemotePortDynPkt rspqueue;

    struct {
        RemotePortRespSlot rsp_queue[RP_MAX_OUTSTANDING_TRANSACTIONS];
    } dev_state[REMOTE_PORT_MAX_DEVS];

    RemotePortDevice *devs[REMOTE_PORT_MAX_DEVS];
}
```
## machine init
- qemu-master/hw/arm/arm_generic_fdt.c
```
#define ZYNQ7000_MACHINE_NAME "arm-generic-fdt-7series"
DEFINE_MACHINE(ZYNQ7000_MACHINE_NAME, arm_generic_fdt_7000_machine_init)

static void arm_generic_fdt_7000_machine_init(MachineClass *mc)
    - static void arm_generic_fdt_7000_init(MachineState *machine)
        - 

```
- qemu-master/hw/arm/xlnx-zynqmp.c
```
static void xlnx_zynqmp_class_init(ObjectClass *oc, void *data)
```
- dtsi/zynq-zc702.dts
    - dtsi/zynq-pl-remoteport.dtsi
```
		m_axi_gp0: rp_m_axi_gp0@40000000 {
			compatible = "remote-port-memory-master";
			remote-ports = < &cosim_rp_0 7 >;
			reg = < 0x40000000 0x40000000 >;
		};
```