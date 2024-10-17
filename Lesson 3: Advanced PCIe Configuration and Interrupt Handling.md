# Lesson: Understanding PCIe Configuration and Interrupt Handling on Xilinx Artix-7 Platforms

## **Overview**

This lesson delves into the intricacies of PCI Express (PCIe) configuration, specifically focusing on determining maximal payload sizes and handling interrupts within FPGA-based systems using Xilinx Artix-7. Additionally, it explores the PCIe data link layer mechanisms, including Transaction Layer Packets (TLPs), Data Link Layer Packets (DLLPs), and flow control. By the end of this lesson, you will have a comprehensive understanding of configuring PCIe devices, defining interrupts in FPGA designs, and optimizing PCIe performance on Artix-7 platforms.

---
    
## **Objectives**

By the end of this lesson, you will be able to:

1. **Determine Maximal Payload Sizes** for PCIe devices using both automated tools and manual methods.
2. **Define and Handle Interrupts** in FPGA-based PCIe designs for Xilinx Artix-7 platforms.
3. **Understand PCIe Data Link Layer Mechanisms**, including TLPs, DLLPs, and flow control.
4. **Optimize PCIe Performance** by analyzing payload sizes and their impact on data transmission.

---
    
## **1. Introduction to PCI Express (PCIe)**

PCI Express (PCIe) is a high-speed serial computer expansion bus standard designed to replace older bus standards like PCI and PCI-X. It is widely used for connecting peripheral devices to the motherboard, including graphics cards, SSDs, and network cards. Understanding PCIe configuration and interrupt handling is crucial for optimizing system performance and ensuring reliable communication between devices.

On FPGA platforms like Xilinx Artix-7, implementing PCIe interfaces allows for the creation of custom peripherals, enabling applications in hardware testing, system debugging, data acquisition, and more.

---
    
## **2. Determining Maximal Payload Size**

### **2.1. Automated Approach with `lspci`**

The `lspci` utility in Linux provides detailed information about all PCI devices connected to the system. It can automatically determine the maximal payload size a PCIe device can handle without manual intervention.

**Example Command:**

```bash
lspci -vv
```

This command displays verbose information about all PCI devices, including their capabilities and configuration registers.

### **2.2. Manual Approach: Parsing Configuration Space**

Understanding how to manually determine the maximal payload size can demystify underlying processes and provide deeper insights into PCIe configurations.

#### **2.2.1. PCIe Configuration Space**

Each PCIe device has a 256-byte configuration space containing various registers that define its capabilities and settings. Key registers related to payload size include:

- **Device Capabilities Register (DevCap):** Located at offset `0x04` in the PCI Express Capability structure. Bits `2-0` indicate the `max_payload_size_capable`.
  
- **Device Control Register (DevCtrl):** Located at offset `0x08` in the PCI Express Capability structure. Bits `7-5` indicate the `max_payload_size_in_effect`.

#### **2.2.2. Calculating Maximal Payload Size**

The maximal payload size is determined using the following formula:

```c
max_payload_size_capable = 1 << ((DevCapReg & 0x07) + 7); // In bytes
max_payload_size_in_effect = 1 << (((DevCtrlReg >> 5) & 0x07) + 7); // In bytes
```

- **`DevCapReg`**: Value from the Device Capabilities Register.
- **`DevCtrlReg`**: Value from the Device Control Register.

#### **2.2.3. Example: Manual Parsing with `lspci -xxx`**

Consider the following `lspci -xxx` output for a specific device:

```
01:00.0 Class ff00: Xilinx Corporation Artix-7 PCIe Core
00: ee 10 34 12 07 04 10 00 00 00 00 ff 01 00 00 00
...
40: 01 48 03 70 08 00 00 00 05 58 81 00 0c 30 e0 fe
...
58: 00 00 00 00 00 00 00 00
60: 10 28 00 00 11 f4 03 00
...
```

**Steps:**

1. **Identify the PCI Express Capability Structure:**
   - Locate the capability with **Cap ID `0x10`**.
   - Follow the linked list pointers to find the structure at offset `0x58`.

2. **Extract Registers:**
   - **Device Capabilities Register (`0x5C`):** `0x00288fc2`
     - Bits `2-0`: `2` → `1 << (2 + 7) = 512 bytes`
   - **Device Control Register (`0x60`):** `0x00002810`
     - Bits `7-5`: `0` → `1 << (0 + 7) = 128 bytes`

3. **Interpretation:**
   - **Max Payload Size Capable:** `512 bytes`
   - **Max Payload Size in Effect:** `128 bytes`

### **2.3. Impact on Performance**

A smaller maximal payload size (e.g., `128 bytes`) can lead to increased overhead, as more TLPs (Transaction Layer Packets) are required to transmit the same amount of data compared to larger payload sizes (e.g., `512 bytes`). This overhead affects the effective bandwidth:

- **128-byte TLPs:**
  - Overhead: ~12%
  - Effective Bandwidth: ~219 MB/s

- **512-byte TLPs:**
  - Overhead: ~3.4%
  - Effective Bandwidth: ~241 MB/s

Understanding and optimizing payload sizes can significantly enhance system performance, especially in high-throughput applications.

---
    
## **3. Interrupt Definitions in FPGA-Based PCIe Designs**

### **3.1. Overview of Interrupts in PCIe**

Interrupts are essential for asynchronous event handling in PCIe devices. Properly defining and handling interrupts ensures that the FPGA-based PCIe device can notify the host system of events, such as data availability or error conditions.

### **3.2. Defining Interrupts in FPGA Designs**

Interrupts in FPGA-based PCIe designs are typically managed through Message Signaled Interrupts (MSI) or MSI-X, which allow devices to generate interrupts by writing to a specific memory address.

#### **3.2.1. Example: MSI Interrupt Definition**

In the FPGA firmware, defining MSI involves configuring the MSI capability structure and ensuring that the host system can map and handle these interrupts.

**Verilog Example:**

```verilog
// MSI Capability Structure
module msi_cap (
    input wire clk,
    input wire reset,
    // Other signals
    output reg [31:0] msi_address,
    output reg [31:0] msi_data,
    output reg msi_enable
);

// Configuration
always @(posedge clk or posedge reset) begin
    if (reset) begin
        msi_address <= 32'h00000000;
        msi_data <= 32'h00000000;
        msi_enable <= 1'b0;
    end else begin
        // Set MSI Address and Data based on host configuration
        msi_address <= 32'hFEE00000; // Example address
        msi_data <= 32'h00000001;    // Example data
        msi_enable <= 1'b1;          // Enable MSI
    end
end

endmodule
```

**Explanation:**

- **`msi_address`:** The address where the MSI will write to notify the host.
- **`msi_data`:** The data payload sent with the MSI.
- **`msi_enable`:** Enables or disables MSI generation.

### **3.3. Handling Interrupts in the Host System**

On the host side (e.g., Linux), interrupt handling involves mapping the MSI addresses and ensuring that the kernel can process the interrupts generated by the FPGA.

**Example Steps on Linux:**

1. **Device Tree Configuration:**

   Define the MSI interrupt properties in the device tree to inform the kernel about the interrupt capabilities.

   ```dts
   pcie_fpga@0000:01:00.0 {
       compatible = "xilinx,artix7-pcie-core";
       reg = <0x0000 0x0001 0x0000 0x0000>;
       interrupts = <1 45 4>;
       interrupt-parent = <&msi_controller>;
       msix-num = <16>;
       // Other properties
   };
   ```

   **Explanation:**
   
   - **`interrupts = <1 45 4>;`**
     - **`1`**: Indicates MSI interrupt.
     - **`45`**: Interrupt number.
     - **`4`**: Level-sensitive, active high.

2. **Kernel Module Driver:**

   In the FPGA's kernel driver, register the interrupt handler to respond to MSIs.

   ```c
   irq = irq_of_parse_and_map(op->dev.of_node, 0);
   rc = request_irq(irq, artix7_pcie_isr, 0, "artix7_pcie", op->dev);
   if (rc) {
       // Handle error
   }
   ```

   - **`irq_of_parse_and_map`:** Parses the `interrupts` property and maps it to a Linux IRQ number.
   - **`request_irq`:** Registers the interrupt handler (`artix7_pcie_isr`) for the specified IRQ.

---
    
## **4. PCIe Data Link Layer Mechanisms**

### **4.1. Transaction Layer Packets (TLPs)**

TLPs are the fundamental units of communication in PCIe, carrying both control and data information between devices.

**Components of a TLP:**

- **Header:** Contains information like type, length, and address.
  - **3 DWs** for 32-bit addressing.
  - **4 DWs** for 64-bit addressing.
  
- **Data Payload:** The actual data being transmitted.
  
- **Optional Digest:** Typically a 1-DW TLP digest (ECRC).

### **4.2. Data Link Layer Packets (DLLPs)**

DLLPs are used by the Data Link Layer to manage reliable transmission over PCIe. They include:

- **Ack DLLP:** Acknowledges successful receipt of TLPs.
- **Nack DLLP:** Indicates a corrupted TLP, prompting retransmission.
- **Flow Control DLLPs:** Manage credits for data transmission.
- **Power Management DLLPs:** Handle power state transitions.

### **4.3. Flow Control Mechanism**

Flow Control ensures that a sender does not overwhelm a receiver by transmitting more data than it can handle.

**Key Concepts:**

- **Flow Control Units:** Correspond to 16 bytes (4 DWs) of traffic.
  
- **Credit Types:** Six distinct types based on the TLP category:
  1. Posted Requests TLPs (headers)
  2. Posted Requests TLPs (data)
  3. Non-Posted Requests TLPs (headers)
  4. Non-Posted Requests TLPs (data)
  5. Completion TLPs (headers)
  6. Completion TLPs (data)

- **Doorkeeper Analogy:** Manages the number of flow control units to prevent buffer overflows.

**Flow Control Operations:**

1. **Initial Exchange:** Both sides exchange their initial credit limits upon link establishment.
2. **Credit Updates:** As TLPs are transmitted and processed, credit limits are updated via UpdateFC DLLPs.
3. **Credit Overflow Handling:** Uses modulo arithmetic to handle counter overflows gracefully.

**Infinite Credit Option:**

Endpoints must advertise infinite credit for completion headers and data, ensuring they can always accept completion TLPs without relying on flow control.

### **4.4. Virtual Channels**

Virtual Channels (VCs) allow multiple independent streams of TLPs to coexist without interfering with each other.

**Key Points:**

- **Traffic Class (TC):** Identifies the virtual channel.
- **Default Usage:** Most systems use a single virtual channel (`TC0`), rendering VCs optional for standard applications.
- **Advanced Usage:** Useful in scenarios requiring segregated traffic streams to prevent blocking.

### **4.5. Packet Reordering**

PCIe allows certain degrees of TLP reordering to optimize transmission and prevent deadlocks.

**Reordering Rules:**

1. **Posted Writes and MSIs:** Always arrive in the order sent.
2. **Read Requests:** Never arrive before preceding write requests or MSIs.
3. **Write Requests:** May arrive before preceding read requests.
4. **Read Completions:** Ordered per request but can be reordered across different requests.

**Implications:**

- **Consistency:** Ensures memory operations maintain expected order for data integrity.
- **Deadlock Prevention:** Reordering rules are designed to prevent communication deadlocks in complex topologies.

### **4.6. Zero-Length Read Requests**

Zero-length read requests are used to ensure write operations have completed without retrieving any actual data.

**Behavior:**

- **Completion Requirement:** Must return a single DW of data, which is typically disregarded.
- **Usage Scenario:** Acts as a synchronization mechanism to confirm the completion of preceding write operations.

---
    
## **5. Practical Considerations**

### **5.1. FPGA Firmware Probing and Mapping**

When developing FPGA firmware for PCIe devices, it's essential to correctly probe and map hardware resources to ensure seamless communication with the host system.

**Key Steps:**

1. **Configure PCIe Parameters:**
   - Set device IDs, vendor IDs, subsystem IDs, and BARs to match the desired emulated device.
   
2. **Implement Interrupt Handling:**
   - Configure MSI/MSI-X capabilities in the FPGA firmware.
   - Ensure the host system can map and handle these interrupts.

3. **Map Memory Regions:**
   - Define BAR sizes and address spaces accurately to match the target device's configuration.

4. **Integrate with Host Drivers:**
   - Develop or modify host-side drivers to interact with the FPGA-based PCIe device effectively.

### **5.2. Accessing Hardware Registers Correctly**

Directly accessing hardware registers via pointers can lead to cache coherency issues, especially on platforms interfacing with FPGAs like Artix-7.

**Best Practices:**

- **Use IO Functions:** Utilize provided API functions or interfaces for register access to maintain cache coherency.
- **Avoid `volatile`:** The Linux kernel discourages the use of the `volatile` keyword for hardware registers. Instead, use appropriate memory barriers or synchronization mechanisms.

**Example in Kernel Module:**

```c
#include <linux/io.h>

u32 value;
value = ioread32(registers + OFFSET);
iowrite32(new_value, registers + OFFSET);
```

### **5.3. Interrupt Handler Registration**

Properly registering interrupt handlers ensures that your FPGA-based PCIe device responds to hardware events correctly.

**Example:**

```c
irq = irq_of_parse_and_map(op->dev.of_node, 0);
rc = request_irq(irq, artix7_pcie_isr, 0, "artix7_pcie", op->dev);
if (rc) {
    dev_err(&op->dev, "Failed to request IRQ\n");
    return rc;
}
```

- **`irq_of_parse_and_map`:** Parses the `interrupts` property and maps it to a Linux IRQ number.
- **`request_irq`:** Registers the interrupt handler (`artix7_pcie_isr`) for the specified IRQ.

---
    
## **6. Summary and Best Practices**

- **Maximal Payload Size:**
  - Use `lspci -vv` for automated retrieval.
  - Manual parsing provides deeper insights and can be essential for custom configurations.
  - Larger payload sizes reduce overhead and improve bandwidth efficiency.

- **Interrupt Definitions:**
  - Clearly define interrupts in FPGA designs with correct flags, numbers, and types.
  - Understand the distinction between MSI and MSI-X interrupts and their respective configurations.
  - Ensure the interrupt type aligns with hardware capabilities and kernel expectations.

- **PCIe Data Link Layer:**
  - TLPs and DLLPs manage data transmission and flow control.
  - Flow control mechanisms prevent buffer overflows and ensure reliable communication.
  - Virtual Channels and packet reordering enhance transmission efficiency and prevent deadlocks.

- **FPGA Firmware Development:**
  - Always use IO functions for register access to maintain cache coherency.
  - Properly probe and map hardware resources to avoid conflicts.
  - Register interrupt handlers accurately to respond to hardware events effectively.

---
    
## **7. Exercises and Questions**

### **Exercise 1: Calculating Maximal Payload Size**

Given a Device Capabilities Register value of `0x00000005` and a Device Control Register value of `0x00000060`, calculate:

1. **Max Payload Size Capable**
2. **Max Payload Size in Effect**

*Solution:*

1. **Max Payload Size Capable:**
   - `(DevCapReg & 0x07) + 7 = (5) + 7 = 12`
   - `1 << 12 = 4096 bytes`

2. **Max Payload Size in Effect:**
   - `((DevCtrlReg >> 5) & 0x07) + 7 = ((0x60 >> 5) & 0x07) + 7 = (3 & 0x07) + 7 = 10`
   - `1 << 10 = 1024 bytes`

### **Question 1: Interrupt Type Flags**

Explain the significance of each value in the `interrupts` property `<0x0 0x32 0x0>` for a device in an FPGA-based PCIe design.

*Answer:*

- **`0x0`:** Indicates the interrupt is an MSI (Message Signaled Interrupt).
- **`0x32`:** The interrupt number, used by the host to identify the interrupt source.
- **`0x0`:** Specifies the interrupt type as default (leave as set by the FPGA firmware).

### **Question 2: Flow Control Units**

Why are flow control units rounded up to the nearest integer when calculating data consumption in TLPs?

*Answer:*

Flow control units correspond to fixed sizes (16 bytes) and ensure that buffer space is allocated efficiently without mixing data from different TLPs. Rounding up ensures that any partial data payload still consumes a full flow control unit, maintaining alignment and preventing buffer overflow.

### **Exercise 2: FPGA Firmware Interrupt Mapping**

Given the following interrupt definition in FPGA firmware:

```verilog
interrupts = <1 45 4>;
```

Determine:

1. **Is the interrupt an MSI or MSI-X?**
2. **What is the interrupt number?**
3. **What is the interrupt type?**

*Solution:*

1. **`1`:** Indicates it's an MSI (Message Signaled Interrupt).
2. **`45`:** Interrupt number as specified.
3. **`4`:** Level-sensitive, active high.

---
    
## **8. Further Reading and Resources**

- **PCI Express Base Specification:** Comprehensive guide to PCIe standards.
- **Xilinx Artix-7 FPGA Documentation:** Official documentation for Artix-7 FPGA configurations and capabilities.
- **Linux Device Drivers Documentation:** Official documentation for writing and understanding PCIe drivers in Linux.
- **Xilinx Vivado Documentation:** Detailed guides and tutorials for using Vivado with Artix-7 FPGAs.
- **PCILeech-FPGA Repository:** [https://github.com/ufrisk/pcileech-fpga](https://github.com/ufrisk/pcileech-fpga)
- **Wireshark PCIe Extensions:** [https://www.wireshark.org/docs/](https://www.wireshark.org/docs/)
- **Teledyne LeCroy Telescan PE Documentation:** [Teledyne LeCroy Telescan PE](https://www.teledynelecroy.com/protocolanalyzer/pci-express/telescan-pe-software/resources/analysis-software)
- **Arbor Software User Guide:** [Arbor User Guide](https://www.mindshare.com/software/Arbor)
- **Field Programmable Gate Array (FPGA) Basics:** [FPGA Basics](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2020_2/ug901-vivado-tutorial.pdf)
- **PCI-SIG Specifications:** [PCI-SIG](https://pcisig.com/specifications)

---
    
**End of Lesson**

---

## **Feedback and Donation Request**

I aimed to provide this lesson in a clear and structured format to enhance understanding and facilitate easier absorption of complex PCIe concepts on Xilinx Artix-7 platforms. Your feedback is invaluable in improving the quality and effectiveness of this content. If you find this lesson helpful and would like to support further development, please consider making a donation.

Thank you for your support!

---
    
## **Support My Work**

If you found this guide helpful and want to fuel more projects, consider contributing to help keep everything going. Every donation helps me continue to create, share, and support the community, and it’s greatly appreciated.

### **Crypto Donations (LTC)**  
- MPMyQD5zgy2b2CpDn1C1KZ31KmHpT7AwRi

I don’t make much from selling firmware—maybe 5 sales adding up to around $300 total—but I know many people using this guide will go on to make far more. If this guide helped you on your path to success, consider giving back so I can keep these resources available for everyone.

### **Special Bonus**  
If you donate, reach out to me on Discord (VCPU) and let me know. I’d love to thank you personally and offer something in return. I’ll even do a random prize spin—whether it's free firmware, private source codes, or some insights on bypassing ACS, there’ll be something special for you.

Your support means the world to me, and together we can keep building and sharing for the future. Thank you!

---
    
Don't have the time or energy to develop firmware yourself? I offer custom firmware solutions starting at $60. Resellers are also welcome!

Reach out anytime for support or further discussion on this guide or related topics. Whether you’re a developer needing in-depth help or a researcher diving into FPGA emulation, I’m here to ensure your path to success is smooth and informed. Let's build something remarkable together.

If you are unsure if you completed a step properly or want me to review your implementation, I will do so, but you must have the section for review marked with `//VCPU-REVIEW//` and explain your problems so my time is not wasted.

I am sure quite a few of you will exceed my capabilities. If you find something new or just develop some impressive firmware, I would love to see it in action. Moreover, I also have some very powerful bytes and bits to share, but if I shared here Riot PD would put me in a squad car.

Share knowledge and spread loving kindness. God bless.

---

## **Contact Information**

If you need assistance, have inquiries, or are looking to collaborate, feel free to reach out. I’m available to provide guidance, troubleshoot complex problems, or discuss ideas in detail.

### **Discord** - **VCPU** | [**Server**](https://discord.gg/dS2gDUDQmV)

---

## **Best Practices for Firmware Development**

Adhering to best practices ensures the development process is efficient, maintainable, and secure.

### **15.1. Continuous Testing and Documentation**

- **Test Frequently**
  - **Steps**:
    1. Conduct regular tests after each modification to ensure the firmware behaves as expected.
    2. Use automated scripts or test benches to validate firmware functionality continuously.

- **Document Changes**
  - **Steps**:
    1. Maintain detailed documentation for each change made to the firmware.
    2. Include explanations for why changes were made and their impact on the overall design.

### **15.2. Managing Firmware Versioning**

- **Use Version Control**
  - **Steps**:
    1. Implement a version control system (e.g., **Git**) to manage different iterations of the firmware.
    2. Commit changes regularly with descriptive messages to track the evolution of the project.

- **Branching Strategy**
  - **Steps**:
    1. Use branches to manage feature development, bug fixes, and experimental changes.
    2. Merge stable branches into the main branch only after thorough testing.

### **15.3. Security Considerations**

- **Prevent Unintended Access**
  - **Steps**:
    1. Ensure that the firmware does not expose system memory or hardware to unauthorized access.
    2. Implement access controls and validation checks within the firmware.

- **Protect Firmware Integrity**
  - **Steps**:
    1. Avoid introducing vulnerabilities or backdoors during firmware development.
    2. Conduct regular security reviews and code audits to maintain firmware integrity.

- **Handle Sensitive Data Securely**
  - **Steps**:
    1. If the firmware interacts with sensitive data, implement encryption and secure data handling practices.
    2. Ensure that sensitive information is not exposed through firmware interfaces or logs.
