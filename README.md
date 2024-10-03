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
