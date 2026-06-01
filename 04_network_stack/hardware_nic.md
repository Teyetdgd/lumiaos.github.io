# Network Interface Cards (NIC)

To connect the TCP/IP stack to the physical (or virtual) world, LumiaOS includes native drivers for popular network interface cards.

## Intel PRO/1000 (E1000)

Located in `src/net/nic_e1000.cpp`. The E1000 is the default and most robust network adapter supported by LumiaOS (commonly used in QEMU and VirtualBox).

### PCI Initialization
During the boot sequence, the PCI bus is scanned for the E1000 Vendor ID (`0x8086`) and Device IDs (`0x100E` / `0x10D3`).
- **MMIO Base:** The driver extracts the Base Address Register (BAR0) to find where the NIC's registers are mapped in physical memory.
- **Paging:** The kernel maps these physical addresses into virtual memory so the driver can safely interact with the hardware registers.

### Ring Buffers (RX / TX)
The E1000 driver uses DMA (Direct Memory Access) via Ring Buffers.
- **Receive (RX):** The driver allocates an array of `e1000_rx_desc` structures. When the NIC receives a packet from the network, it writes the data directly to the memory pointed to by the descriptor and fires an interrupt (IRQ).
- **Transmit (TX):** To send a packet, the TCP/IP stack fills a buffer, and the E1000 driver updates a `e1000_tx_desc` structure. It then advances the Tail pointer register (`TDT`), signaling the hardware to fetch the packet from RAM and transmit it over the wire.

## AMD PCnet-FAST III (RTL8139 / PCnet)

Located in `src/net/nic_pcnet.cpp`. This is a legacy driver kept for compatibility with older emulators. It operates similarly to the E1000 but relies more on Port I/O (`inb`/`outb`) rather than strict MMIO.

### The NIC Abstraction Layer
Both drivers implement a common interface defined in `src/net/nic.cpp`. This allows the upper IP/Ethernet layers to call a generic `nic_send_packet()` function, without needing to know whether the underlying hardware is an Intel E1000 or an AMD PCnet adapter.
