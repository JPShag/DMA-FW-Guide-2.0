# **Custom Firmware Development Guide for Full Device Emulation**

---

## **Preface**

Welcome to **Part 1: The Basics** of this comprehensive guide on developing custom firmware for full device emulation using FPGA-based DMA hardware. This section is designed to walk you through foundational concepts, preparing you to create, customize, and test your first custom firmware. After completing this part, you'll have a solid understanding of how to gather device information, configure your FPGA, and emulate a PCIe device.

---

## **Contact Information**

For further assistance, feedback, or collaboration, feel free to connect with the community:

### **Discord** - **VCPU** ([Join Now](https://discord.gg/dS2gDUDQmV))

---

# **Table of Contents**

1. [Introduction](#1-introduction)
   - [1.1 Purpose of the Guide](#11-purpose-of-the-guide)
   - [1.2 Target Audience](#12-target-audience)
2. [Key Definitions](#2-key-definitions)
3. [Device Compatibility](#3-device-compatibility)
   - [3.1 Supported FPGA-Based Hardware](#31-supported-fpga-based-hardware)
   - [3.2 PCIe Hardware Considerations](#32-pcie-hardware-considerations)
   - [3.3 System Requirements](#33-system-requirements)
4. [Project Hierarchy Overview](#4-project-hierarchy-overview)
   - [4.1 Directory Structure](#41-directory-structure)
   - [4.2 Key Files and Their Roles](#42-key-files-and-their-roles)
5. [Requirements](#5-requirements)
   - [5.1 Hardware](#51-hardware)
   - [5.2 Software](#52-software)
   - [5.3 Environment Setup](#53-environment-setup)
6. [Gathering Donor Device Information](#6-gathering-donor-device-information)
   - [6.1 Using PCIe Device Scanning Tools](#61-using-pcie-device-scanning-tools)
   - [6.2 Extracting and Recording Device Attributes](#62-extracting-and-recording-device-attributes)
7. [Initial Firmware Customization](#7-initial-firmware-customization)
   - [7.1 Modifying Configuration Space](#71-modifying-configuration-space)
   - [7.2 Inserting the Device Serial Number (DSN)](#72-inserting-the-device-serial-number-dsn)
8. [Vivado Project Setup and Customization](#8-vivado-project-setup-and-customization)
   - [8.1 Generating Vivado Project Files](#81-generating-vivado-project-files)
   - [8.2 Modifying IP Blocks](#82-modifying-ip-blocks)
9. [Building, Flashing, and Testing](#9-building-flashing-and-testing)
   - [9.1 Synthesis and Implementation](#91-synthesis-and-implementation)
   - [9.2 Flashing the Bitstream](#92-flashing-the-bitstream)
   - [9.3 Testing and Validation](#93-testing-and-validation)
10. [Troubleshooting](#10-troubleshooting)
    - [10.1 Device Detection Issues](#101-device-detection-issues)
    - [10.2 Memory Mapping and BAR Configuration Errors](#102-memory-mapping-and-bar-configuration-errors)
    - [10.3 DMA Performance and TLP Errors](#103-dma-performance-and-tlp-errors)

---

# **1. Introduction**

### **1.1 Purpose of the Guide**

The primary goal of this guide is to help developers create custom firmware to emulate PCIe devices using FPGA-based DMA hardware. The first part of this guide focuses on the basics, walking through the essential steps for firmware development, device information gathering, environment setup, and initial firmware customization. This will lay the groundwork for more advanced custom firmware development covered in **Part 2**.

### **1.2 Target Audience**

This guide is intended for:

- **Beginners**: People new to firmware development, FPGA programming, or PCIe device emulation.
- **Firmware Developers**: Engineers creating custom firmware for testing, emulation, or specific hardware functionality.
- **Hardware Engineers**: Professionals working on hardware design and testing using FPGAs.
- **Security Researchers**: Individuals leveraging hardware-based security tools and custom emulated devices.

---

# **2. Key Definitions**

Understanding the following key concepts is essential for effective firmware development:

- **DMA (Direct Memory Access)**: A feature that allows hardware devices to access the system’s memory independently of the CPU, often used for fast data transfers.
- **FPGA (Field Programmable Gate Array)**: A programmable integrated circuit that allows developers to create custom hardware designs.
- **PCIe (Peripheral Component Interconnect Express)**: A fast data transfer interface used for connecting hardware devices to a computer's motherboard.
- **TLP (Transaction Layer Packet)**: The data packet format used in PCIe communication, containing control and data information.
- **BAR (Base Address Register)**: Registers in PCIe devices that specify the memory regions mapped to system memory for device communication.
- **Device Serial Number (DSN)**: A unique identifier used by some devices for enhanced identification and emulation accuracy.

---

# **3. Device Compatibility**

### **3.1 Supported FPGA-Based Hardware**

This guide focuses on FPGA-based devices that are compatible with **PCILeech-FPGA** and capable of emulating PCIe devices through DMA operations. Supported hardware includes:

- **Squirrel (35T)**
- **EnigmaX1 (75T)**
- **ZDMA (100T)**
- **Kintex-7**

Each of these devices can be programmed using Xilinx Vivado and can perform full PCIe device emulation tasks.

### **3.2 PCIe Hardware Considerations**

Several important PCIe-related configurations must be addressed for successful emulation:

- **IOMMU/VT-d**: Disable Intel’s Virtualization Technology for Directed I/O to allow unrestricted DMA access.
- **Kernel DMA Protection**: Disable Kernel DMA protection features to ensure unrestricted memory access.
- **PCIe Slot Requirements**: Ensure that the FPGA card is installed in a compatible PCIe slot (e.g., x1, x4, x16) that matches the card’s capabilities.

### **3.3 System Requirements**

To successfully develop, test, and run custom firmware, the following system setup is recommended:

- **Host System**:
  - **Processor**: Multi-core CPU (Intel i5/i7 or equivalent)
  - **Memory**: Minimum 16GB of RAM
  - **Storage**: SSD with at least 100GB of free space
  - **Operating System**: Windows 10/11 (64-bit) or a compatible Linux distribution such as Ubuntu or Debian
- **Peripheral Devices**:
  - **JTAG Programmer**: Required for flashing firmware onto the FPGA
  - **Available PCIe Slot**: Ensure your system has an available PCIe slot for the FPGA card

---

# **4. Project Hierarchy Overview**

To make effective changes and implement custom firmware, it’s crucial to understand the project structure.

### **4.1 Directory Structure**

Here is an example of the structure you’ll encounter when working on the **PCILeech-FPGA** project:

```
📦 pcileech-wifi-main
 ┣ 📂 ip
 ┃ ┣ 📂 100t
 ┃ ┃ ┣ 📜 bram_bar_zero4k.xci
 ┃ ┃ ┣ 📜 bram_pcie_cfgspace.xci
 ┃ ┃ ┣ 📜 clk_wiz_0.xci
 ┃ ┃ ┗ 📜 pcileech_cfgspace_writemask.coe
 ┣ 📂 pcie_7x
 ┃ ┣ 📂 zdma
 ┃ ┃ ┣ 📜 pcie_7x_0.v
 ┃ ┃ ┣ 📜 pcie_7x_0_axi_basic_rx.v
 ┃ ┃ ┗ 📜 pcie_7x_0_pcie_top.v
 ┣ 📂 src
 ┃ ┣ 📜 pcileech_pcie_a7.sv
 ┃ ┣ 📜 pcileech_pcie_cfg_a7.sv
 ┃ ┣ 📜 pcileech_squirrel_top.sv
 ┣ 📜 build.md
 ┣ 📜 vivado_build.tcl
 ┣ 📜 generate-squirrel.bat
 ┗ 📜 README.md
```

### **4.2 Key Files and Their Roles**

- **`ip` Directory**: Contains the FPGA Intellectual Property (IP) cores for various components like the PCIe configuration space and DMA buffers.
- **`pcie_7x` Directory**: Houses PCIe-specific components including PCIe top modules and DMA logic for FPGA communication with PCIe devices.
- **`src` Directory**: Contains Verilog files that define the FPGA’s behavior. Key files include:
  - **`pcileech_pcie_a7.sv`**: Implements the PCIe interface logic.
  - **`pcileech_pcie_cfg_a7.sv`**: Contains the PCIe configuration space logic.
  - **`pcileech_squirrel_top.sv`**: Top-level module for the Squirrel FPGA.
- **Build Scripts**:
  - **`generate-squirrel.bat`**: Automates the generation of Vivado projects for the Squirrel FPGA.
  - **`vivado_build.tcl`**: Tcl script used to build the project in Vivado.

---

# **5. Requirements**

### **5.1 Hardware**

To get started with firmware development

, you’ll need the following hardware:

- **Donor PCIe Device**: This is the device you plan to emulate. Typical examples include network adapters or storage controllers.
- **FPGA Card**: A DMA-capable FPGA device such as the **Squirrel (35T)** or **EnigmaX1 (75T)**.
- **JTAG Programmer**: Necessary for flashing the firmware onto the FPGA card.

### **5.2 Software**

- **Xilinx Vivado**: Required for FPGA development. This is used to synthesize and implement the custom firmware.
- **Visual Studio Code**: A code editor for writing and modifying the Verilog/SystemVerilog source code.
- **PCILeech-FPGA Repository**: The base code repository where you’ll find the necessary scripts and source files.
- **PCIe Device Scanning Tools**: Tools for extracting configuration data from PCIe devices.
  - **Arbor**: A commercial PCIe device scanning tool (14-day trial available).
  - **Alternatives**: Linux’s `lspci` command or **PCI-Z** for Windows.

### **5.3 Environment Setup**

1. **Install Xilinx Vivado**
   - Visit the [Xilinx Vivado Download Page](https://www.xilinx.com/support/download.html) and download the appropriate version for your FPGA device.
   - Follow the installation instructions to install Vivado on your system.

2. **Install Visual Studio Code**
   - Download Visual Studio Code from [here](https://code.visualstudio.com/).
   - Install any necessary extensions for Verilog/SystemVerilog support.

3. **Clone the PCILeech-FPGA Repository**
   - Open a terminal or command prompt, then run the following commands:
     ```bash
     git clone https://github.com/ufrisk/pcileech-fpga.git
     cd pcileech-fpga
     ```

4. **Set Up a Clean Development Environment**
   - It is recommended to use a dedicated machine or virtual environment for development, especially if you plan to work with sensitive data. Ensure no other FPGA projects or tools interfere with your setup.

---

# **6. Gathering Donor Device Information**

Accurate emulation depends on gathering the correct data from the donor device, including configuration details like Device ID, Vendor ID, and Base Address Registers (BARs).

### **6.1 Using PCIe Device Scanning Tools**

#### **Option 1: Using Arbor**

1. **Install Arbor**
   - Download from [Arbor’s Website](https://www.mindshare.com/software/Arbor) and create an account to access the trial version.
   - Install the software on your host system.

2. **Scan PCIe Devices**
   - Launch Arbor and navigate to the **Local System** tab.
   - Click **Scan/Rescan** to detect all connected PCIe devices.

3. **Identify the Donor Device**
   - Locate the donor device in the scan results and select it to view its detailed configuration.

#### **Option 2: Using `lspci` (Linux)**

1. **Open a Terminal**

2. **List PCI Devices**
   - Run the following command:
     ```bash
     lspci -vvv
     ```

3. **Identify Donor Device**
   - Locate the donor PCIe device by name or class (e.g., a network adapter).

4. **Extract Information**
   - Take note of important information such as Device ID, Vendor ID, Subsystem ID, BAR sizes, and capabilities.

#### **Option 3: Using PCI-Z (Windows)**

1. **Download PCI-Z**
   - Download from the [PCI-Z Website](https://www.pci-z.com/).

2. **Run PCI-Z**
   - Launch the software and view the list of connected PCI devices.

3. **Identify Donor Device**
   - Locate the device from the list and extract the necessary information.

### **6.2 Extracting and Recording Device Attributes**

For accurate emulation, record the following details from the donor device:

- **Device ID**: The unique identifier for the hardware device.
- **Vendor ID**: The identifier for the manufacturer.
- **Subsystem ID**: The identifier for any subsystems associated with the device.
- **Revision ID**: The version number of the hardware.
- **BARs (Base Address Registers)**: The size, type (memory or I/O), and layout of the device’s memory regions.
- **Capabilities**: Features like MSI/MSI-X, power management, and PCIe link speed.
- **Device Serial Number (DSN)**: If available, the unique serial number of the device.

---

# **7. Initial Firmware Customization**

### **7.1 Modifying Configuration Space**

The PCIe configuration space defines how the FPGA is identified by the host system. To spoof the donor device, you’ll need to customize this space with the correct values.

1. **Open `pcileech_pcie_cfg_a7.sv`** in Visual Studio Code.

2. **Modify Device and Vendor IDs**
   - Replace the default Device ID and Vendor ID with the values from the donor device:
     ```verilog
     cfg_deviceid <= 16'h1234; // Replace with donor Device ID
     cfg_vendorid <= 16'hABCD; // Replace with donor Vendor ID
     ```

3. **Modify Subsystem IDs**
   - Update the Subsystem IDs to match the donor device:
     ```verilog
     cfg_subsysid <= 16'h5678;  // Replace with donor Subsystem ID
     cfg_subsysvid <= 16'hABCD; // Typically same as Vendor ID
     ```

4. **Set Revision IDs and Class Codes**
   - Set the device class and revision:
     ```verilog
     cfg_rev_id <= 8'h01;  // Replace with donor Revision ID
     cfg_class_code <= 24'h020000; // Example for Network Controller
     ```

5. **Configure BARs**
   - Define the number and size of the Base Address Registers (BARs):
     ```verilog
     cfg_bar_0 <= 32'hFFFFFFF0; // 32-bit Memory BAR
     ```

### **7.2 Inserting the Device Serial Number (DSN)**

1. **Locate DSN Configuration**
   - Insert the donor device’s serial number:
     ```verilog
     cfg_dsn <= 64'h01000000684CE000; // Replace with donor DSN
     ```

2. **Handle Missing DSN**
   - If the donor device does not have a DSN, use zeros or generate a unique DSN:
     ```verilog
     cfg_dsn <= 64'h0000000000000000;
     ```

---

# **8. Vivado Project Setup and Customization**

### **8.1 Generating Vivado Project Files**

1. **Open Vivado**

2. **Launch the Tcl Console**
   - In the Vivado interface, open the Tcl Console from the toolbar or navigation menu.

3. **Navigate to the Project Directory**
   - Use the `cd` command to navigate to the PCILeech project:
     ```tcl
     cd C:/path/to/pcileech-fpga/PCIeSquirrel
     ```

4. **Generate the Vivado Project**
   - Run the following command to generate the project:
     ```tcl
     source vivado_generate_project_squirrel.tcl -notrace
     ```

### **8.2 Modifying IP Blocks**

1. **Open the PCIe IP Core Configuration**
   - In Vivado, navigate to the **Sources** pane and find `pcileech_squirrel_top`.
   - Locate the **`pcie_7x_0`** module, right-click it, and choose **Customize IP**.

2. **Modify Device and Vendor IDs**
   - In the customization window, enter the correct Device ID and Vendor ID from the donor device.

3. **Configure BARs**
   - Set the correct number, type, and size of BARs based on the donor device’s configuration.

4. **Set Revision and Class Codes**
   - Ensure that the revision and class codes match the donor device.

5. **Lock the IP Core**
   - Lock the IP core to prevent Vivado from overwriting these changes during synthesis:
     ```tcl
     set_property is_managed false [get_files pcie_7x_0.xci]
     ```

---

# **9. Building, Flashing, and Testing**

### **9.1 Synthesis and Implementation**

1. **Run Synthesis**
   - In Vivado, click **Run Synthesis** to synthesize the design.
   - Check for errors and warnings, resolving them as needed.

2. **Run Implementation**
   - Once synthesis is complete, run **Implementation** to generate the necessary files for your FPGA.

3. **Generate Bitstream**
   - Click **Generate Bitstream** and wait for the process to complete.

### **9.2 Flashing the Bitstream**

1. **Connect FPGA via JTAG**
   - Connect your JTAG programmer to the FPGA and ensure it is powered on.

2. **Open Vivado Hardware Manager**
   - In Vivado, go to **Open Hardware Manager** and select **Open Target** to detect the connected FPGA device.

3. **Program the FPGA**
   - Right-click on the FPGA device and select **Program Device**. Choose the generated bitstream file and click **Program**.

### **9.3 Testing and Validation**

1. **Verify Device Detection**
   - Use **Device Manager** (Windows) or `lspci` (Linux) to check if the FPGA is detected as the donor device.

2. **Test Device Functionality

**
   - Depending on the device, perform basic tests such as checking network connectivity (for network cards) or accessing storage (for storage devices).

3. **Test Performance**
   - Use benchmarking tools to verify that the device performs as expected, comparing results with the original hardware.

---

# **10. Troubleshooting**

### **10.1 Device Detection Issues**

If the host system does not detect the FPGA, try the following:

- **Check Physical Connections**: Ensure the FPGA card is properly seated in the PCIe slot.
- **Review Configuration Settings**: Verify the correctness of the Device ID, Vendor ID, and BAR configurations.
- **BIOS/UEFI Settings**: Ensure that DMA protection features are disabled and that the system supports external DMA devices.

### **10.2 Memory Mapping and BAR Configuration Errors**

If you experience issues with memory mapping or accessing the device:

- **Verify BAR Sizes and Addresses**: Double-check that the BAR sizes and address ranges match those of the donor device.
- **Memory Alignment**: Ensure that BARs are properly aligned to the system memory.
- **Resolve Overlapping BARs**: Avoid assigning overlapping memory regions to different BARs.

### **10.3 DMA Performance and TLP Errors**

For performance or data transfer issues:

- **Optimize FIFO Sizes**: Adjust FIFO buffers for better data handling.
- **TLP Payload Sizes**: Set the maximum TLP payload size to match the donor device’s specifications.
- **PCIe Link Settings**: Ensure the PCIe link speed and width match the host system's PCIe capabilities.

---

# **Part 2: Advanced Tutorials**

---

## **Introduction to Advanced Concepts**

In **Part 2**, we’ll dive into advanced techniques for customizing and optimizing your custom firmware for full device emulation. This section will cover more intricate topics such as modifying PCIe parameters for specific emulation tasks, advanced debugging, and techniques for improving emulation accuracy. You’ll also learn how to incorporate additional capabilities like power management, virtual functions, and advanced PCIe features.

By the end of Part 2, you’ll be well-equipped to tackle more sophisticated firmware development challenges and optimize your FPGA design for specific use cases.

---

## **Table of Contents**

11. [Advanced Firmware Customization](#11-advanced-firmware-customization)
    - [11.1 Configuring PCIe Parameters for Emulation](#111-configuring-pcie-parameters-for-emulation)
    - [11.2 Adjusting BARs and Memory Mapping](#112-adjusting-bars-and-memory-mapping)
    - [11.3 Emulating Device Power Management and Interrupts](#113-emulating-device-power-management-and-interrupts)
12. [Emulating Device-Specific Capabilities](#12-emulating-device-specific-capabilities)
    - [12.1 Implementing Advanced PCIe Capabilities](#121-implementing-advanced-pcie-capabilities)
    - [12.2 Emulating Vendor-Specific Features](#122-emulating-vendor-specific-features)
13. [Transaction Layer Packet (TLP) Emulation](#13-transaction-layer-packet-tlp-emulation)
    - [13.1 Understanding and Capturing TLPs](#131-understanding-and-capturing-tlps)
    - [13.2 Crafting Custom TLPs for Specific Operations](#132-crafting-custom-tlps-for-specific-operations)
14. [Advanced Debugging Techniques](#14-advanced-debugging-techniques)
    - [14.1 Using Vivado's Integrated Logic Analyzer](#141-using-vivados-integrated-logic-analyzer)
    - [14.2 PCIe Traffic Analysis Tools](#142-pcie-traffic-analysis-tools)
15. [Emulation Accuracy and Optimizations](#15-emulation-accuracy-and-optimizations)
    - [15.1 Techniques for Accurate Timing Emulation](#151-techniques-for-accurate-timing-emulation)
    - [15.2 Dynamic Response to System Calls](#152-dynamic-response-to-system-calls)
16. [Best Practices for Firmware Development](#16-best-practices-for-firmware-development)
    - [16.1 Continuous Testing and Documentation](#161-continuous-testing-and-documentation)
    - [16.2 Managing Firmware Versioning](#162-managing-firmware-versioning)
    - [16.3 Security Considerations](#163-security-considerations)
17. [Incorporating ekknod's FPGA DMA Firmware](#17-incorporating-ekknods-fpga-dma-firmware)
    - [17.1 Overview of ekknod's Contributions](#171-overview-of-ekknods-contributions)
    - [17.2 Adapting ekknod's Techniques](#172-adapting-ekknods-techniques)
    - [17.3 Enhancements for Wi-Fi and Multimedia Emulation](#173-enhancements-for-wi-fi-and-multimedia-emulation)
18. [Implementing Additional Features](#18-implementing-additional-features)
    - [18.1 Power Management Emulation](#181-power-management-emulation)
    - [18.2 PCIe Gen3 and Beyond](#182-pcie-gen3-and-beyond)
    - [18.3 Virtual Functions and SR-IOV](#183-virtual-functions-and-sr-iov)
19. [Appendices](#19-appendices)
    - [19.1 Appendix A: Shadow Configuration Space](#191-appendix-a-shadow-configuration-space)
    - [19.2 Appendix B: Writemask Implementation](#192-appendix-b-writemask-implementation)
20. [Conclusion](#20-conclusion)
21. [Additional Resources](#21-additional-resources)
22. [Support and Community](#22-support-and-community)

---

# **11. Advanced Firmware Customization**

## **11.1 Configuring PCIe Parameters for Emulation**

The accuracy of emulation greatly depends on how well the FPGA-based device replicates the donor device’s PCIe configuration, link speed, and width. Fine-tuning these settings will allow the host system to interact with the FPGA as if it were the original hardware.

### **Setting PCIe Link Speed and Width**

1. **Open `pcileech_pcie_cfg_a7.sv`** in your editor.

2. **Locate the PCIe Link Speed and Width Configurations**:
   - Ensure the values for link speed and width are consistent with the donor device.

    ```verilog
    pcie_link_speed <= 4'b0010;   // PCIe Gen2 (5 GT/s)
    pcie_link_width <= 8'b00000100; // x4 lanes
    ```

   **Example**: For a Gen2 x4 PCIe device, set the link speed to `4'b0010` and the width to `8'b00000100`.

### **Configuring Capability Pointers**

PCIe capability pointers are essential for ensuring the host system can access advanced PCIe features like MSI/MSI-X, power management, and error reporting.

1. **Set Capability Pointer**:
    - Adjust the capability pointer to match that of the donor device.

    ```verilog
    capability_pointer <= 8'h40;  // Starting location for PCIe capabilities
    ```

2. **Extended Capabilities**:
    - You can extend this section by adding or modifying support for features like Advanced Error Reporting (AER) or Latency Tolerance Reporting (LTR).

    ```verilog
    AER_CAP_VERSION <= 4'b0001;  // Enable AER support
    ```

---

## **11.2 Adjusting BARs and Memory Mapping**

Base Address Registers (BARs) allow PCIe devices to map their internal memory to system memory. Proper configuration of BARs is critical for the host system to communicate with the emulated device.

### **Setting BAR Sizes**

1. **Open `pcileech_pcie_cfg_a7.sv`**.

2. **Modify the BAR Size Values** to match the donor device’s BARs.

   **Example**: For a 16KB BAR0:
   ```verilog
   bar0_size <= 32'h00004000;  // 16KB BAR0
   ```

3. **Repeat for Additional BARs** as needed for BAR1, BAR2, etc.

### **Defining BAR Address Spaces**

1. Ensure BAR address spaces are **contiguous** and don’t overlap. Overlapping BARs can cause memory mapping errors in the host system.

   ```verilog
   bar0_addr <= 32'hF0000000;  // Example BAR0 address
   bar1_addr <= 32'hF0004000;  // Example BAR1 address
   ```

2. **Align Memory Regions**: Make sure BARs are aligned according to the donor device’s configuration.

---

## **11.3 Emulating Device Power Management and Interrupts**

Properly emulating power management and interrupts is crucial for seamless system integration. The host system expects certain behaviors from the device when it transitions between power states or handles interrupts.

### **Power Management (PM) Configuration**

1. **Set Power Management Capabilities**:
   - Adjust the power management settings to emulate the donor device's power features.

   ```verilog
   PM_CAP_VERSION <= 4'b0011;     // Power Management version 3.0
   PM_CAP_D1SUPPORT <= 1'b1;      // Support for D1 power state
   PM_CAP_D2SUPPORT <= 1'b0;      // No support for D2
   ```

2. **Power State Transitions**:
   - Implement logic for transitioning between power states (D0, D1, D3) as per the donor device’s capabilities.

### **MSI/MSI-X (Interrupts) Configuration**

1. **Enable MSI or MSI-X** in `pcileech_pcie_cfg_a7.sv`.

   ```verilog
   MSI_CAP_64_BIT_ADDR_CAPABLE <= 1'b1;  // Enable 64-bit address for MSI
   cfg_interrupt <= 1'b1;                // Enable MSI interrupts
   ```

2. **Route Interrupt Signals** correctly to ensure that the host system receives the expected interrupt behavior from the emulated device.

   ```verilog
   assign cfg_interrupt_di = cfg_int_di;
   assign cfg_interrupt_assert = cfg_int_assert;
   ```

---

# **12. Emulating Device-Specific Capabilities**

Many devices include advanced capabilities beyond standard PCIe behavior, such as custom power management or proprietary features. Emulating these device-specific capabilities enhances the fidelity of the emulation.

## **12.1 Implementing Advanced PCIe Capabilities**

### **Advanced Error Reporting (AER)**

1. **Enable AER** to match the donor device’s configuration:
   - Advanced Error Reporting (AER) allows the device to report and log PCIe errors.

    ```verilog
    AER_CAP_VERSION <= 4'b0001;  // AER capability version 1
    AER_CAP_NEXTPTR <= 8'h00;    // No next capability
    ```

### **Link Speed Negotiation**

Ensure that the emulated device supports dynamic link speed negotiation as seen in the donor device.

1. **Set the PCIe Link Speed**:
   - The FPGA should

 negotiate the link speed based on host system capabilities.

    ```verilog
    pcie_link_speed <= 4'b0010;  // Gen2 speed (5 GT/s)
    ```

---

## **12.2 Emulating Vendor-Specific Features**

Certain devices have proprietary features unique to the vendor. These must be carefully implemented to ensure that the FPGA behaves identically to the original device.

### **Capture Vendor-Specific TLPs**

Use PCIe traffic analysis tools (covered in [Section 14.2](#142-pcie-traffic-analysis-tools)) to monitor TLPs and identify vendor-specific behaviors.

### **Implement Vendor-Specific Registers**

Create custom logic to handle any vendor-specific registers or commands.

```verilog
reg [31:0] vendor_specific_reg;

always @(posedge clk) begin
    if (vendor_specific_write_enable) begin
        vendor_specific_reg <= vendor_specific_data_in;
    end
end
```

---

# **13. Transaction Layer Packet (TLP) Emulation**

TLPs are the building blocks of PCIe communication. Accurately crafting TLPs ensures smooth interactions between the host system and the emulated device.

## **13.1 Understanding and Capturing TLPs**

### **TLP Components**

A TLP consists of three key components:
- **Header**: Contains control information such as address, length, and type of transaction.
- **Data Payload**: The actual data being transferred.
- **Tail**: Includes additional control information and checksums.

### **Capturing TLPs**

1. **Use PCIe traffic analysis tools** (e.g., **Wireshark** with PCIe extensions or **Teledyne LeCroy Telescan PE**) to monitor TLP traffic from the donor device.

2. **Analyze TLP Patterns**:
   - Look for recurring TLP patterns, such as memory reads/writes or configuration accesses.

---

## **13.2 Crafting Custom TLPs for Specific Operations**

Once you understand the structure of TLPs, you can craft custom TLPs to match specific operations from the donor device.

### **Memory Write TLP Example**

```verilog
// TLP for Memory Write
tlp_header <= {2'b10, 5'b00000, 3'b000, 1'b0, 1'b0, 2'b00, 10'b0000000001};  // Fmt, Type, etc.
tlp_address <= 64'h0000000012345678;  // Target address
tlp_data <= 32'hDEADBEEF;             // Data payload
```

### **Configuration Access TLP Example**

```verilog
// TLP for Configuration Write
tlp_header <= {2'b10, 5'b00101, 3'b000, 1'b0, 1'b0, 2'b00, 10'b0000000001};  // Fmt, Type, etc.
tlp_address <= 32'h00000010;            // Configuration register address
tlp_data <= 32'h00000001;               // Data to write
```

---

# **14. Advanced Debugging Techniques**

## **14.1 Using Vivado's Integrated Logic Analyzer**

The **Integrated Logic Analyzer (ILA)** in Vivado allows you to capture and analyze real-time signals inside the FPGA. This tool is indispensable for debugging custom firmware.

### **Setting Up ILA Probes**

1. **Open Vivado** and navigate to **Tools > Insert Logic Analyzer**.

2. **Select Signals**: Choose the signals you want to monitor (e.g., TLP data or PCIe interrupts).

3. **Configure Triggers**: Set up triggers to capture specific events, such as TLP generation or MSI interrupts.

---

## **14.2 PCIe Traffic Analysis Tools**

In addition to Vivado’s ILA, external tools allow for comprehensive PCIe traffic analysis.

### **Wireshark with PCIe Extensions**

1. **Install Wireshark** with PCIe support.
   
2. **Capture Traffic**: Use Wireshark to capture PCIe traffic and filter for relevant TLPs.

### **Teledyne LeCroy Telescan PE**

1. **Install Telescan PE** for advanced PCIe protocol analysis.
   
2. **Monitor PCIe Traffic**: Capture and analyze traffic between the FPGA and host system.

---

# **15. Emulation Accuracy and Optimizations**

To ensure seamless integration and undetectable device emulation, follow these advanced optimization techniques.

## **15.1 Techniques for Accurate Timing Emulation**

### **Synchronize Clock Domains**

Ensure that all clocks within the FPGA are synchronized with the PCIe link clock to avoid timing issues during communication.

```verilog
clk_div <= clk / 2;  // Example of clock division for synchronization
```

---

## **15.2 Dynamic Response to System Calls**

1. **Implement State Machines**: Use state machines to manage different operational states of the emulated device.
   
2. **Handle System Requests Dynamically**: Monitor incoming TLPs and adjust the device’s response accordingly.

---

# **16. Best Practices for Firmware Development**

## **16.1 Continuous Testing and Documentation**

- **Test Frequently**: Regularly test the firmware after each modification.
- **Document Changes**: Maintain detailed documentation to track changes.

## **16.2 Managing Firmware Versioning**

- **Use Git for Version Control**: Ensure every significant change is tracked.
- **Branching Strategy**: Use feature branches to manage complex changes.

---

# **17. Incorporating ekknod's FPGA DMA Firmware**

## **17.1 Overview of ekknod's Contributions**

ekknod’s firmware advancements focus on enhancing FPGA-based DMA performance, particularly for Wi-Fi and multimedia applications.

---

# **18. Implementing Additional Features**

## **18.1 Power Management Emulation**

Ensure your firmware properly emulates transitions between power states to match modern energy-efficient devices.

---

# **19. Appendices**

### **19.1 Appendix A: Shadow Configuration Space**

Shadow configuration space allows you to dynamically modify PCIe configuration registers during runtime. Use this technique to emulate complex devices.

---

# **20. Conclusion**

By completing **Part 2: Advanced Tutorials**, you have gained a deeper understanding of advanced firmware customization techniques. You are now prepared to handle complex emulation tasks, optimize your design for specific hardware, and troubleshoot advanced issues.

---

# **21. Additional Resources**

- **PCILeech-FPGA Repository**: [GitHub](https://github.com/ufrisk/pcileech-fpga)
- **Xilinx Vivado Documentation**: [Xilinx](https://www.xilinx.com/support/documentation.html)

---

# **22. Support and Community**

If you need further assistance or want to collaborate, join our Discord community:

**[Discord - VCPU](https://discord.gg/dS2gDUDQmV)**

