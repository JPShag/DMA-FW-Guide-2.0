# **Custom Firmware Development Guide for Full Device Emulation**

---

## **Preface**

Welcome to the comprehensive guide on developing custom firmware for full device emulation using FPGA-based DMA hardware. This guide is designed to cater to both beginners and advanced developers. It is divided into two parts:

- **Part 1: The Basics** – Covers foundational knowledge and step-by-step instructions for initial firmware development.
- **Part 2: Advanced Tutorials** – Delves into complex concepts, optimizations, and advanced techniques for enhancing your firmware.

Whether you're a firmware developer, hardware engineer, security researcher, or FPGA enthusiast, this guide will equip you with the knowledge and tools necessary to create accurate and efficient emulation firmware.

---

## **Contact Information**

For assistance, inquiries, or collaboration opportunities, please join our community on Discord:

### **Discord** - **VCPU** ([Join Now](https://discord.gg/dS2gDUDQmV))

---

# **Table of Contents**

## **Part 1: The Basics**

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

## **Part 2: Advanced Tutorials**

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
17. [Incorporating Echnod's FPGA DMA Firmware](#17-incorporating-echnods-fpga-dma-firmware)
    - [17.1 Overview of Echnod's Contributions](#171-overview-of-echnods-contributions)
    - [17.2 Adapting Echnod's Techniques](#172-adapting-echnods-techniques)
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

# **Part 1: The Basics**

---

## **1. Introduction**

### **1.1 Purpose of the Guide**

The primary goal of this guide is to provide a step-by-step approach to developing custom firmware for FPGA-based DMA hardware, enabling accurate emulation of PCIe devices. This facilitates applications in hardware testing, security research, system debugging, and more.

### **1.2 Target Audience**

- **Beginners**: Individuals new to firmware development and FPGA programming.
- **Firmware Developers**: Engineers creating custom firmware for device emulation or testing.
- **Hardware Engineers**: Professionals working on hardware device development and testing.
- **Security Researchers**: Individuals interested in hardware-based security analysis.

---

## **2. Key Definitions**

Understanding the terminology is crucial for effective firmware development.

- **DMA (Direct Memory Access)**: Allows hardware devices to access system memory directly, bypassing the CPU.
- **FPGA (Field Programmable Gate Array)**: A reconfigurable integrated circuit that can be programmed to perform specific hardware functions.
- **PCIe (Peripheral Component Interconnect Express)**: A high-speed serial computer expansion bus standard for connecting hardware devices.
- **TLP (Transaction Layer Packet)**: The fundamental unit of communication in PCIe, encapsulating control and data information.
- **BAR (Base Address Register)**: Registers in PCIe devices that map device memory into system memory space.
- **Device Serial Number (DSN)**: A unique identifier for a hardware device.

---

## **3. Device Compatibility**

### **3.1 Supported FPGA-Based Hardware**

The following FPGA devices are compatible with this guide:

- **Squirrel (35T)**
- **EnigmaX1 (75T)**
- **ZDMA (100T)**
- **Kintex-7**

These devices are capable of performing DMA operations and can be programmed using Xilinx Vivado.

### **3.2 PCIe Hardware Considerations**

- **Disable IOMMU/VT-d**: To allow unrestricted DMA access.
- **Disable Kernel DMA Protection**: In BIOS/UEFI settings.
- **Ensure PCIe Slot Compatibility**: Use appropriate PCIe slots (e.g., x1, x4, x16).

### **3.3 System Requirements**

- **Host System**:
  - Multi-core CPU (Intel i5/i7 or equivalent)
  - Minimum 16GB RAM
  - SSD with at least 100GB free space
  - Windows 10/11 (64-bit) or a compatible Linux distribution

- **Peripheral Devices**:
  - **JTAG Programmer**: For flashing firmware onto the FPGA.
  - **Available PCIe Slot**: Compatible with the FPGA device.

---

## **4. Project Hierarchy Overview**

Understanding the project structure helps in navigating and modifying the firmware effectively.

### **4.1 Directory Structure**

The project is organized as follows:

```
📦 pcileech-wifi-main
 ┣ 📂 ip
 ┃ ┣ 📂 100t
 ┃ ┃ ┣ 📜 bram_bar_zero4k.xci
 ┃ ┃ ┣ 📜 bram_pcie_cfgspace.xci
 ┃ ┃ ┣ 📜 clk_wiz_0.xci
 ┃ ┃ ┣ 📜 ...
 ┃ ┃ ┗ 📜 pcileech_cfgspace_writemask.coe
 ┣ 📂 pcie_7x
 ┃ ┣ 📂 zdma
 ┃ ┃ ┣ 📜 pcie_7x_0.v
 ┃ ┃ ┣ 📜 ...
 ┃ ┃ ┗ 📜 pcie_7x_0_rxeq_scan.v
 ┣ 📂 src
 ┃ ┣ 📜 pcileech_pcie_a7.sv
 ┃ ┣ 📜 pcileech_pcie_cfg_a7.sv
 ┃ ┣ 📜 ...
 ┃ ┗ 📜 pcileech_squirrel_top.sv
 ┣ 📜 build.md
 ┣ 📜 generate-squirrel.bat
 ┣ 📜 vivado_build.tcl
 ┗ 📜 README.md
```

### **4.2 Key Files and Their Roles**

- **`ip` Directory**: Contains IP cores used in the project.
- **`pcie_7x` Directory**: Houses PCIe-specific modules.
- **`src` Directory**: Contains the main source files, including:

  - **`pcileech_pcie_a7.sv`**: PCIe interface logic.
  - **`pcileech_pcie_cfg_a7.sv`**: Configuration space implementation.
  - **`pcileech_squirrel_top.sv`**: Top-level module for the Squirrel FPGA.

- **Build Scripts**:

  - **`generate-squirrel.bat`**: Batch script to generate the Vivado project for Squirrel.
  - **`vivado_build.tcl`**: Tcl script for building the project in Vivado.

---

## **5. Requirements**

### **5.1 Hardware**

- **Donor PCIe Device**: The device you wish to emulate.
- **DMA FPGA Card**: One of the supported FPGA devices.
- **JTAG Programmer**: For flashing the firmware.

### **5.2 Software**

- **Xilinx Vivado**: For FPGA development.
- **Visual Studio Code**: For editing source code.
- **PCILeech-FPGA Repository**: Base code for firmware development.
- **PCIe Device Scanning Tools**: 

  - **Arbor**: Commercial tool with a trial version.
  - **Alternatives**: `lspci` (Linux), **PCI-Z** (Windows).

### **5.3 Environment Setup**

1. **Install Xilinx Vivado**:

   - Download from [Xilinx Vivado Download Page](https://www.xilinx.com/support/download.html).
   - Follow installation instructions.

2. **Install Visual Studio Code**:

   - Download from [Visual Studio Code](https://code.visualstudio.com/).
   - Install necessary extensions for Verilog/SystemVerilog support.

3. **Clone the PCILeech-FPGA Repository**:

   ```bash
   git clone https://github.com/ufrisk/pcileech-fpga.git
   cd pcileech-fpga
   ```

4. **Set Up a Clean Development Environment**:

   - Use a dedicated machine or virtual environment.
   - Ensure no conflicts with other FPGA projects.

---

## **6. Gathering Donor Device Information**

Accurate emulation requires detailed information from the donor device.

### **6.1 Using PCIe Device Scanning Tools**

#### **Option 1: Using Arbor**

1. **Install Arbor**:

   - Download from [Arbor Download Page](https://www.mindshare.com/software/Arbor).
   - Install and launch the application.

2. **Scan for PCIe Devices**:

   - Navigate to the **Local System** tab.
   - Click **Scan/Rescan**.

3. **Select Donor Device**:

   - Locate your device in the list.
   - View detailed configuration.

#### **Option 2: Using `lspci` (Linux)**

1. **Open Terminal**.

2. **List PCI Devices**:

   ```bash
   lspci -vvv
   ```

3. **Identify Donor Device**:

   - Find the device by name or class.

4. **Extract Information**:

   - Note down **Device ID**, **Vendor ID**, **Subsystem ID**, etc.

#### **Option 3: Using PCI-Z (Windows)**

1. **Download PCI-Z** from [PCI-Z Download Page](https://www.pci-z.com/).

2. **Run PCI-Z**.

3. **Identify Donor Device**:

   - Locate the device in the list.

4. **Extract Information**:

   - Record necessary attributes.

### **6.2 Extracting and Recording Device Attributes**

Ensure you collect the following:

- **Device ID** (e.g., `0x1234`)
- **Vendor ID** (e.g., `0xABCD`)
- **Subsystem ID** (e.g., `0x5678`)
- **Revision ID**
- **BARs**:

  - Number of BARs.
  - Type (Memory or I/O).
  - Size.

- **Capabilities**:

  - MSI/MSI-X support.
  - Power Management.
  - PCIe link speed and width.

- **Device Serial Number (DSN)** (if available).

---

## **7. Initial Firmware Customization**

### **7.1 Modifying Configuration Space**

1. **Open `pcileech_pcie_cfg_a7.sv`** in Visual Studio Code.

2. **Modify Device and Vendor IDs**:

   ```verilog
   cfg_deviceid <= 16'h1234; // Replace with donor Device ID
   cfg_vendorid <= 16'hABCD; // Replace with donor Vendor ID
   ```

3. **Modify Subsystem IDs**:

   ```verilog
   cfg_subsysid <= 16'h5678; // Replace with donor Subsystem ID
   cfg_subsysvid <= 16'hABCD; // Typically same as Vendor ID
   ```

4. **Adjust Class Codes and Revision IDs**:

   ```verilog
   cfg_rev_id <= 8'h01;      // Replace with donor Revision ID
   cfg_class_code <= 24'h020000; // Example for Network Controller
   ```

5. **Configure BARs**:

   - Set the number, type, and size of BARs.

   ```verilog
   cfg_bar_0 <= 32'hFFFFFFF0; // 32-bit Memory BAR
   // Set other BARs as needed
   ```

### **7.2 Inserting the Device Serial Number (DSN)**

1. **Locate DSN Configuration**:

   ```verilog
   cfg_dsn <= 64'h01000000684CE000; // Replace with donor DSN
   ```

2. **If DSN is Unavailable**:

   - You may generate a unique DSN or use zeros.

   ```verilog
   cfg_dsn <= 64'h0000000000000000;
   ```

---

## **8. Vivado Project Setup and Customization**

### **8.1 Generating Vivado Project Files**

1. **Open Vivado**.

2. **Launch Tcl Console**.

3. **Navigate to Project Directory**:

   ```tcl
   cd C:/path/to/pcileech-fpga/PCIeSquirrel
   ```

4. **Generate Project**:

   ```tcl
   source vivado_generate_project_squirrel.tcl -notrace
   ```

### **8.2 Modifying IP Blocks**

1. **Open the PCIe IP Core Configuration**:

   - In Vivado, go to **Sources** > **pcileech_squirrel_top**.
   - Locate **`pcie_7x_0`**.
   - Right-click and select **Customize IP**.

2. **Modify Device IDs**:

   - Enter the **Device ID** and **Vendor ID** from your donor device.

3. **Configure BARs**:

   - Match the BAR configuration to the donor device.

4. **Set Class Codes and Revision IDs**:

   - Ensure these match the donor device.

5. **Lock the IP Core**:

   - Prevents Vivado from overwriting changes during synthesis.

   ```tcl
   set_property is_managed false [get_files pcie_7x_0.xci]
   ```

---

## **9. Building, Flashing, and Testing**

### **9.1 Synthesis and Implementation**

1. **Run Synthesis**:

   - In Vivado, click **Run Synthesis**.
   - Resolve any errors.

2. **Run Implementation**:

   - Click **Run Implementation**.
   - Ensure timing constraints are met.

3. **Generate Bitstream**:

   - Click **Generate Bitstream**.
   - Wait for completion.

### **9.2 Flashing the Bitstream**

1. **Connect FPGA via JTAG**.

2. **Open Hardware Manager**:

   - In Vivado, click **Open Hardware Manager**.

3. **Program Device**:

   - Right-click on the FPGA device.
   - Select **Program Device**.
   - Choose the generated bitstream file.
   - Click **Program**.

### **9.3 Testing and Validation**

1. **Verify Device Detection**:

   - Use **Device Manager** (Windows) or `lspci` (Linux) to check if the device is recognized as the donor device.

2. **Functional Testing**:

   - Install drivers if necessary.
   - Test basic functionality (e.g., network connectivity for a network card).

3. **Performance Testing**:

   - Use benchmarking tools to assess performance.

---

## **10. Troubleshooting**

### **10.1 Device Detection Issues**

- **Check Physical Connections**.
- **Verify Configuration Settings**.
- **Ensure BIOS/UEFI Settings Allow DMA Operations**.

### **10.2 Memory Mapping and BAR Configuration Errors**

- **Confirm BAR Sizes and Addresses**.
- **Ensure Proper Memory Alignment**.
- **Check for Overlapping Memory Regions**.

### **10.3 DMA Performance and TLP Errors**

- **Optimize FIFO Sizes**.
- **Adjust TLP Payload Sizes**.
- **Review PCIe Link Settings**.

---

# **Part 2: Advanced Tutorials**

---

## **11. Advanced Firmware Customization**

### **11.1 Configuring PCIe Parameters for Emulation**

1. **Match PCIe Link Speed and Width**:

   - In `pcileech_pcie_cfg_a7.sv`, set `pcie_link_speed` and `pcie_link_width`.

   ```verilog
   pcie_link_speed <= 4'b0010;   // Gen2 (5 GT/s)
   pcie_link_width <= 8'b00000100; // x4 lanes
   ```

2. **Set Capability Pointers**:

   - Adjust `capability_pointer` to match the donor device.

   ```verilog
   capability_pointer <= 8'h40;
   ```

3. **Modify Extended Capabilities**:

   - Implement capabilities like AER, LTR, etc.

### **11.2 Adjusting BARs and Memory Mapping**

1. **Set BAR Sizes**:

   ```verilog
   bar0_size <= 32'h00004000; // 16KB for BAR0
   ```

2. **Define BAR Address Spaces**:

   - Ensure addresses do not overlap.

3. **Handle Multiple BARs**:

   - Configure additional BARs as needed.

### **11.3 Emulating Device Power Management and Interrupts**

1. **Power Management (PM) Configuration**:

   - Implement PM capabilities.

   ```verilog
   PM_CAP_VERSION <= 4'b0011;
   PM_CAP_D1SUPPORT <= 1'b1;
   ```

2. **MSI/MSI-X Configuration**:

   - Enable and configure MSI/MSI-X interrupts.

   ```verilog
   MSI_CAP_64_BIT_ADDR_CAPABLE <= 1'b1;
   cfg_interrupt <= 1'b1;
   ```

3. **Implement Interrupt Handling Logic**:

   - Ensure correct routing of interrupt signals.

---

## **12. Emulating Device-Specific Capabilities**

### **12.1 Implementing Advanced PCIe Capabilities**

1. **Advanced Error Reporting (AER)**:

   - Enable and configure AER in the firmware.

   ```verilog
   AER_CAP_VERSION <= 4'b0001;
   ```

2. **Latency Tolerance Reporting (LTR)**:

   - Implement LTR if supported by the donor device.

   ```verilog
   LTR_CAP_SUPPORTED <= 1'b1;
   ```

### **12.2 Emulating Vendor-Specific Features**

1. **Identify Vendor-Specific Features**:

   - Use PCIe traffic analyzers to capture unique behaviors.

2. **Implement Vendor-Specific Logic**:

   - Add custom registers and handling in the firmware.

3. **Testing**:

   - Use vendor-specific drivers or applications to validate functionality.

---

## **13. Transaction Layer Packet (TLP) Emulation**

### **13.1 Understanding and Capturing TLPs**

1. **Use PCIe Traffic Analyzers**:

   - Tools like Teledyne LeCroy’s Telescan PE.

2. **Analyze TLP Structures**:

   - Understand headers, payloads, and control fields.

### **13.2 Crafting Custom TLPs for Specific Operations**

1. **Memory Read TLP**:

   ```verilog
   tlp_header <= {fmt, type, tc, td, ep, attr, length};
   tlp_address <= target_address;
   ```

2. **Memory Write TLP**:

   ```verilog
   tlp_data <= data_payload;
   ```

3. **Configuration Access TLP**:

   - For accessing configuration space registers.

---

## **14. Advanced Debugging Techniques**

### **14.1 Using Vivado's Integrated Logic Analyzer**

1. **Insert ILA Cores**:

   - In Vivado, add ILA cores to monitor internal signals.

2. **Set Trigger Conditions**:

   - Define events that capture data.

3. **Analyze Captured Data**:

   - Use the waveform viewer to inspect signal behavior.

### **14.2 PCIe Traffic Analysis Tools**

1. **Teledyne LeCroy Telescan PE**:

   - Capture and analyze PCIe traffic.

2. **Total Phase Beagle**:

   - Real-time PCIe protocol analysis.

3. **Wireshark with PCIe Extensions**:

   - Monitor PCIe packets on supported systems.

---

## **15. Emulation Accuracy and Optimizations**

### **15.1 Techniques for Accurate Timing Emulation**

1. **Clock Synchronization**:

   - Align FPGA clocks with PCIe reference clocks.

2. **Response Timing Control**:

   - Implement delays to match donor device response times.

### **15.2 Dynamic Response to System Calls**

1. **State Machines**:

   - Manage device states based on system interactions.

2. **Adaptive Logic**:

   - Adjust behavior dynamically during runtime.

---

## **16. Best Practices for Firmware Development**

### **16.1 Continuous Testing and Documentation**

1. **Regular Testing**:

   - After each change, test the firmware.

2. **Detailed Documentation**:

   - Maintain records of modifications and configurations.

### **16.2 Managing Firmware Versioning**

1. **Use Version Control Systems**:

   - Git or other systems to track changes.

2. **Branching Strategies**:

   - Use branches for new features or experiments.

### **16.3 Security Considerations**

1. **Prevent Unauthorized Access**:

   - Implement access controls in the firmware.

2. **Code Audits**:

   - Regularly review code for vulnerabilities.

---

## **17. Incorporating Echnod's FPGA DMA Firmware**

### **17.1 Overview of Echnod's Contributions**

- **Optimized TLP Handling**
- **Advanced DMA Techniques**
- **Enhanced Error Handling**
- **Multimedia Stream Emulation**

### **17.2 Adapting Echnod's Techniques**

1. **Implement Efficient TLP Parsing**:

   - Use pipelined architectures.

2. **Zero-Copy DMA**:

   - Reduce latency and CPU overhead.

3. **Robust Error Detection**:

   - Implement comprehensive error handling.

### **17.3 Enhancements for Wi-Fi and Multimedia Emulation**

1. **Wi-Fi Emulation**:

   - Simulate wireless protocols and signal processing.

2. **Multimedia Stream Emulation**:

   - Handle high-bandwidth data streams with synchronization.

---

## **18. Implementing Additional Features**

### **18.1 Power Management Emulation**

1. **Advanced Power States**:

   - Implement ASPM and LPM.

2. **Dynamic Power Adjustment**:

   - Emulate power consumption based on activity.

### **18.2 PCIe Gen3 and Beyond**

1. **Higher Bandwidth Support**:

   - Enable PCIe Gen3 or higher in the IP core.

2. **Enhanced Equalization**:

   - Implement link training procedures.

### **18.3 Virtual Functions and SR-IOV**

1. **Enable SR-IOV Capabilities**:

   - Modify configuration space for SR-IOV.

2. **Implement Virtual Function Logic**:

   - Handle multiple virtual devices.

---

## **19. Appendices**

### **19.1 Appendix A: Shadow Configuration Space**

- **Utilize Shadow Configuration Space**:

  - Allows customization beyond the constraints of the IP core.

- **Implementation Steps**:

  - Convert donor configuration to `.coe` file.
  - Modify `pcileech_cfgspace.coe`.

### **19.2 Appendix B: Writemask Implementation**

- **Enable Write Capabilities**:

  - Implement writemask to allow writes to configuration registers.

- **Implementation Steps**:

  - Modify `pcileech_fifo.sv`.
  - Generate writemask file using provided scripts.

---

## **20. Conclusion**

By following this guide, you have learned how to develop custom firmware for full device emulation using FPGA-based DMA hardware. You are now equipped with the foundational knowledge and advanced techniques necessary to create sophisticated emulation solutions.

---

## **21. Additional Resources**

- **PCILeech-FPGA Repository**: [GitHub](https://github.com/ufrisk/pcileech-fpga)
- **Xilinx Vivado Documentation**: [Xilinx Documentation](https://www.xilinx.com/support/documentation.html)
- **PCI-SIG Specifications**: [PCI-SIG](https://pcisig.com/specifications)
- **Echnod's FPGA DMA Firmware**: *[Provide link if available]*

---

## **22. Support and Community**

Join our vibrant community on Discord to connect with fellow developers, seek assistance, and collaborate on enhancing custom firmware projects:

[![Discord Banner](https://discord.gg/dS2gDUDQmV)](https://discord.gg/dS2gDUDQmV)

---

# **Final Thoughts**

This guide has been divided into two comprehensive parts to cater to both beginners and advanced developers. By starting with the basics and progressively moving to advanced topics, you can build a strong foundation and then delve into complex aspects of firmware development.

Remember, firmware development is an iterative process. Regular testing, continuous learning, and community engagement are key to success.

---

# **Next Steps**

- **Practice**: Apply the concepts learned by working on a real FPGA device.
- **Experiment**: Try implementing advanced features from Part 2.
- **Contribute**: Share your experiences and improvements with the community.

---

# **Additional Enhancements**

To further improve your understanding:

- **Explore Case Studies**: Review real-world applications of FPGA-based device emulation.
- **Attend Workshops/Webinars**: Participate in events related to FPGA development.
- **Read Technical Papers**: Stay updated with the latest research and advancements.

By continuously enhancing your skills and knowledge, you'll be well-equipped to tackle complex firmware development challenges.

---

# **Acknowledgments**

We extend our gratitude to all contributors and community members who have shared their knowledge and experience, making this guide possible.

---

# **Happy Coding!**

We hope this guide serves as a valuable resource in your firmware development journey. Should you have any questions or need further assistance, don't hesitate to reach out through our community channels.

---

*This guide is provided for educational purposes. Ensure compliance with all relevant laws and regulations when applying the techniques described.*

---

# **End of Guide**

Thank you for your commitment to learning and excellence in firmware development.
