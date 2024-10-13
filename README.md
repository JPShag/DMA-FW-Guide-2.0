# **Custom Firmware Development Guide for Full Device Emulation**


## **Table of Contents**

1. [Introduction](#1-introduction)
   - [1.1 Purpose of the Guide](#11-purpose-of-the-guide)
   - [1.2 Target Audience](#12-target-audience)
2. [Key Definitions](#2-key-definitions)
3. [Device Compatibility](#3-device-compatibility)
   - [3.1 Supported FPGA-Based Hardware](#31-supported-fpga-based-hardware)
   - [3.2 PCIe Hardware Considerations](#32-pcie-hardware-considerations)
   - [3.3 System Requirements](#33-system-requirements)
4. [Requirements](#4-requirements)
   - [4.1 Hardware](#41-hardware)
   - [4.2 Software](#42-software)
   - [4.3 Environment Setup](#43-environment-setup)
5. [Gathering Donor Device Information](#5-gathering-donor-device-information)
   - [5.1 Using Arbor for PCIe Device Scanning](#51-using-arbor-for-pcie-device-scanning)
   - [5.2 Extracting and Recording Device Attributes](#52-extracting-and-recording-device-attributes)
6. [Initial Firmware Customization](#6-initial-firmware-customization)
   - [6.1 Modifying Configuration Space](#61-modifying-configuration-space)
   - [6.2 Inserting the Device Serial Number (DSN)](#62-inserting-the-device-serial-number-dsn)
7. [Vivado Project Setup and Customization](#7-vivado-project-setup-and-customization)
   - [7.1 Generating Vivado Project Files](#71-generating-vivado-project-files)
   - [7.2 Modifying IP Blocks](#72-modifying-ip-blocks)
8. [Advanced Firmware Customization](#8-advanced-firmware-customization)
   - [8.1 Configuring PCIe Parameters for Emulation](#81-configuring-pcie-parameters-for-emulation)
   - [8.2 Adjusting BARs and Memory Mapping](#82-adjusting-bars-and-memory-mapping)
   - [8.3 Emulating Device Power Management and Interrupts](#83-emulating-device-power-management-and-interrupts)
9. [Emulating Device-Specific Capabilities](#9-emulating-device-specific-capabilities)
   - [9.1 Implementing Advanced PCIe Capabilities](#91-implementing-advanced-pcie-capabilities)
   - [9.2 Emulating Vendor-Specific Features](#92-emulating-vendor-specific-features)
10. [Transaction Layer Packet (TLP) Emulation](#10-transaction-layer-packet-tlp-emulation)
    - [10.1 Understanding and Capturing TLPs](#101-understanding-and-capturing-tlps)
    - [10.2 Crafting Custom TLPs for Specific Operations](#102-crafting-custom-tlps-for-specific-operations)
11. [Building, Flashing, and Testing](#11-building-flashing-and-testing)
    - [11.1 Synthesis and Implementation](#111-synthesis-and-implementation)
    - [11.2 Flashing the Bitstream](#112-flashing-the-bitstream)
    - [11.3 Testing and Validation](#113-testing-and-validation)
12. [Advanced Debugging Techniques](#12-advanced-debugging-techniques)
    - [12.1 Using Vivado's Integrated Logic Analyzer](#121-using-vivados-integrated-logic-analyzer)
    - [12.2 PCIe Traffic Analysis Tools](#122-pcie-traffic-analysis-tools)
13. [Troubleshooting](#13-troubleshooting)
    - [13.1 Device Detection Issues](#131-device-detection-issues)
    - [13.2 Memory Mapping and BAR Configuration Errors](#132-memory-mapping-and-bar-configuration-errors)
    - [13.3 DMA Performance and TLP Errors](#133-dma-performance-and-tlp-errors)
14. [Emulation Accuracy and Optimizations](#14-emulation-accuracy-and-optimizations)
    - [14.1 Techniques for Accurate Timing Emulation](#141-techniques-for-accurate-timing-emulation)
    - [14.2 Dynamic Response to System Calls](#142-dynamic-response-to-system-calls)
15. [Best Practices for Firmware Development](#15-best-practices-for-firmware-development)
    - [15.1 Continuous Testing and Documentation](#151-continuous-testing-and-documentation)
    - [15.2 Managing Firmware Versioning](#152-managing-firmware-versioning)
    - [15.3 Security Considerations](#153-security-considerations)
16. [Additional Resources](#16-additional-resources)

---

## **Preface**

**Attention, you thieving scum.** I’m not here to coddle or indulge your pitiful schemes. **FUCK AQUA**, better known as Aqua Teen Paster Force—a disgrace to anyone with a shred of integrity. **FUCK DIVINER**, too. And DUCK? Go degrade yourself elsewhere. **DMA KINGDOM**? You and your parasitic empire can rot in hell with the rest of these frauds. SHITLETTE and all of you money-grubbing thieves, prepare for a reckoning. The **Wrath of God**? You’re about to understand the meaning of true devastation. Either atone for your greed, or get swept away by the destruction you deserve. While we are on this topic, FUCK YOU TOO BILL GATES.

For those who’ve come to **learn**, you’ve chosen the path of **true enlightenment**. What you build with this guide will outshine anything these charlatans could dream of. You’re on the road to **excellence**, and together, we’ll torch the failures they’ve built their fortunes on. 

---

## **Contact Information**

If you need assistance, have inquiries, or are looking to collaborate, feel free to reach out. I’m available to provide guidance, troubleshoot complex problems, or discuss ideas in detail.

### **Discord** - **VCPU** | [**Server**](https://discord.gg/dS2gDUDQmV)

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

Don't have the time or energy to make firmware yourself? I offer firmware starting at $60. Resellers are also welcome!

Reach out anytime for support or further discussion on this guide or related topics. Whether you’re a developer needing in-depth help or a researcher diving into FPGA emulation, I’m here to ensure your path to success is smooth and informed. Let's build something remarkable together.

If you are unsure if you completed a step properly or want me to review your implementation I will do so but you must have the section for review marked with //VCPU-REVIEW// and explain your problems so my time is not wasted.

I am sure quite a few of you will exceed my capabilities, If you find something new or just make some sick firmware, I would love to see it in action. Moreover, I also have some very powerful bytes and bits to share, but if I shared here Riot PD would put me in a squad car.

Share knowledge and spread loving kindness. God bless.

---

## **1. Introduction**

### **1.1 Purpose of the Guide**

The primary objective of this guide is to equip developers, security researchers, and hardware engineers with the knowledge and practical steps necessary to develop custom DMA firmware for accurate 1:1 hardware device emulation using FPGA-based systems like **PCILeech-FPGA**. This enables applications in hardware testing, system debugging, malware analysis, and other scenarios requiring undetectable or legitimate-looking device emulation.

### **1.2 Target Audience**

- **Firmware Developers**: Engineers building custom firmware for hardware emulation, testing, or bypassing hardware restrictions.
- **Hardware Testers**: Professionals emulating faulty or outdated hardware devices to assess system resilience or compatibility.
- **Security Researchers**: Individuals utilizing custom firmware for vulnerability testing, malware analysis, or security assessments.
- **FPGA Enthusiasts**: Hobbyists exploring FPGA customization and low-level hardware emulation.

---

## **2. Key Definitions**

Understanding the terminology is crucial for effectively following this guide. Below are key definitions related to PCIe, DMA, and device emulation:

- **DMA (Direct Memory Access)**: A capability allowing hardware devices to directly read from or write to system memory without CPU intervention, facilitating rapid data transfers.
- **TLP (Transaction Layer Packet)**: The fundamental unit of communication in PCIe architecture, encapsulating control and data information.
- **BAR (Base Address Register)**: Registers in PCIe devices that map device memory into system memory space, defining memory and I/O address regions.
- **FPGA (Field Programmable Gate Array)**: A reconfigurable integrated circuit that can be programmed to perform specific hardware functions, enabling custom device emulation.
- **MSI/MSI-X (Message Signaled Interrupts)**: Mechanisms used by PCIe devices to send interrupts to the CPU, handling asynchronous events.
- **Device Serial Number (DSN)**: A unique identifier associated with a specific device, often used for advanced device identification and verification.
- **PCIe Configuration Space**: A memory area where PCIe devices provide information about themselves and configure operational parameters.
- **Donor Card**: A PCIe device used to extract configuration and identification details for the purpose of emulating its behavior on an FPGA.

---

## **3. Device Compatibility**

### **3.1 Supported FPGA-Based Hardware**

While this guide primarily focuses on the **Squirrel DMA (35T)** card, the methodologies outlined are adaptable to other FPGA-based DMA hardware. Below is a list of compatible devices:

- **Squirrel (35T)**
  - **Description**: Affordable and widely accessible FPGA-based DMA device.
  - **Use Case**: Suitable for standard memory acquisition and device emulation tasks.

- **EnigmaX1 (75T)**
  - **Description**: Mid-tier FPGA offering enhanced resources and performance.
  - **Use Case**: Ideal for more demanding memory operations requiring higher bandwidth.

- **ZDMA (100T)**
  - **Description**: High-performance FPGA, optimized for rapid memory interactions.
  - **Use Case**: Best suited for scenarios requiring fast and extensive memory reads/writes.

- **Kintex-7**
  - **Description**: Advanced FPGA with robust capabilities for complex projects.
  - **Use Case**: Suitable for large-scale or highly customized DMA solutions.

### **3.2 PCIe Hardware Considerations**

To ensure smooth emulation, several PCIe-specific features must be addressed:

- **IOMMU/VT-d Settings**
  - **Recommendation**: Disable IOMMU (Intel's VT-d) to allow unrestricted DMA access.
  - **Rationale**: IOMMU can restrict DMA operations, potentially interfering with memory acquisition and emulation.

- **Kernel DMA Protection**
  - **Recommendation**: Disable Kernel DMA Protection features found in modern systems.
  - **Steps**:
    - **Windows**: This may involve disabling Secure Boot or Virtualization-Based Security (VBS).
    - **BIOS/UEFI**: Access firmware settings to turn off related security features.
  - **Caution**: Disabling these features can expose the system to risks; ensure you're operating within a secure and isolated environment.

- **PCIe Slot Requirements**
  - **Recommendation**: Use a compatible PCIe slot that matches the FPGA device's requirements (e.g., x1, x4, x16).
  - **Rationale**: Ensures optimal performance and compatibility with the host system.

### **3.3 System Requirements**

- **Host System**
  - **Processor**: Multi-core CPU (Intel i5/i7 or equivalent)
  - **Memory**: Minimum 16GB RAM
  - **Storage**: SSD with at least 100GB free space
  - **Operating System**: Windows 10/11 (64-bit) or a compatible Linux distribution (e.g., Ubuntu, Debian) with necessary drivers

- **Peripheral Devices**
  - **JTAG Adapter**: For flashing firmware onto the FPGA
  - **PCIe Slot**: Ensure the host system has available PCIe slots compatible with the DMA card

---

## **4. Requirements**

### **4.1 Hardware**

- **Donor PCIe Device**
  - **Purpose**: Source of device IDs and configuration data for spoofing.
  - **Examples**: Network adapters, storage controllers, or any generic PCIe card not used on the main PC.

- **DMA FPGA Card**
  - **Description**: FPGA-based device capable of performing DMA operations.
  - **Examples**: Squirrel (35T), EnigmaX1 (75T), ZDMA (100T), Kintex-7

- **JTAG Programmer**
  - **Purpose**: For flashing firmware onto the FPGA.
  - **Examples**: Xilinx Platform Cable USB, Digilent JTAG USB Cable

### **4.2 Software**

- **Vivado**
  - **Description**: Xilinx's FPGA development software for synthesizing and building firmware projects.
  - **Download**: [Xilinx Vivado](https://www.xilinx.com/support/download.html)

- **Visual Studio**
  - **Description**: Integrated Development Environment (IDE) for editing Verilog or VHDL code.
  - **Download**: [Visual Studio Community](https://visualstudio.microsoft.com/vs/community/)

- **PCILeech-FPGA**
  - **Description**: The repository and base code for DMA firmware development.
  - **Repository**: [PCILeech-FPGA on GitHub](https://github.com/ufrisk/pcileech-fpga)

- **Arbor**
  - **Description**: PCIe device scanning tool for gathering device information.
  - **Download**: [Arbor by MindShare](https://www.mindshare.com/software/Arbor)
  - **Note**: Requires account creation; offers a 14-day trial.

- **Alternative Tools**
  - **Telescan PE**
    - **Description**: PCIe traffic analysis tool that can be used as an alternative to Arbor.
    - **Download**: [Teledyne LeCroy Telescan PE](https://www.teledynelecroy.com/protocolanalyzer/pci-express/telescan-pe-software/resources/analysis-software)
    - **Note**: Free but requires manual registration approval.

### **4.3 Environment Setup**

1. **Install Vivado**
   - **Steps**:
     1. Visit the [Xilinx Vivado Download Page](https://www.xilinx.com/support/download.html).
     2. Download the appropriate version compatible with your FPGA device.
     3. Follow the installation instructions provided by Xilinx.
     4. Launch Vivado and ensure it is properly configured.

2. **Install Visual Studio**
   - **Steps**:
     1. Visit the [Visual Studio Download Page](https://visualstudio.microsoft.com/vs/community/).
     2. Download and install the **Visual Studio Community Edition**.
     3. During installation, ensure you include workloads related to **Desktop development with C++** to support hardware description languages (HDLs) like Verilog or VHDL.

3. **Clone the PCILeech-FPGA Repository**
   - **Steps**:
     1. Open a terminal or command prompt.
     2. Clone the repository using Git:
        ```bash
        git clone https://github.com/ufrisk/pcileech-fpga.git
        ```
     3. Navigate to the cloned directory:
        ```bash
        cd pcileech-fpga
        ```

4. **Set Up a Clean Development Environment**
   - **Recommendation**: Work in an isolated environment to prevent unintended interactions, especially if using the firmware for sensitive tasks like malware analysis.
   - **Steps**:
     1. Use a dedicated development machine or a virtual environment.
     2. Ensure no other applications interfere with PCIe operations or FPGA programming.

---

## **5. Gathering Donor Device Information**

Accurate device emulation relies on extracting critical information from the donor device. This data allows your FPGA to mimic the target hardware in terms of PCIe configuration and behavior.

### **5.1 Using Arbor for PCIe Device Scanning**

**Arbor** is a powerful tool for scanning PCIe devices and extracting necessary information. Follow these steps to gather donor device details:

1. **Install Arbor**
   - **Steps**:
     1. Visit the [Arbor Download Page](https://www.mindshare.com/software/Arbor).
     2. Create an account if required.
     3. Download and install Arbor on your system.

2. **Scan PCIe Devices**
   - **Steps**:
     1. Launch Arbor.
     2. Navigate to the **Local System** tab.
     3. Under **Scan Options**, ensure default settings are appropriate.
     4. Click **Scan/Rescan** to detect all connected PCIe devices.

3. **Identify the Donor Device**
   - **Criteria**:
     - Should not be used on your main PC.
     - Examples: PCIe WiFi cards, storage controllers, or generic PCIe devices.
   - **Steps**:
     1. Locate your donor device in the list of scanned devices.
     2. Click on the device to view detailed configuration.

4. **Capture Device Data**
   - **Information to Extract**:
     - **Device ID**
     - **Vendor ID**
     - **Subsystem ID**
     - **Revision ID**
     - **Base Address Registers (BARs)**
     - **Capabilities** (e.g., MSI, power management, PCIe link width/speed)
     - **Device Serial Number (DSN)** (if available)

   - **Steps**:
     1. Navigate to the **PCI Config** tab within Arbor.
     2. Scroll through the **Decode** section to locate and record the above details.
     3. Take screenshots or notes of each value for reference during firmware customization.

   - **Example Extraction**:
     
     ![Device IDs](https://github.com/Silverr12/DMA-CFW-Guide/assets/89455475/8baec3fe-c4bd-478e-9f95-d262804d6f67)
     
     ![Vendor ID](https://github.com/Silverr12/DMA-CFW-Guide/assets/89455475/39c7de6d-d8db-4744-b0a0-ddeca0dfd7d7)
     
     ![Revision ID](https://github.com/Silverr12/DMA-CFW-Guide/assets/89455475/c2374ea7-ca9c-47b7-8a8d-4ceff5dffe3b)
     
     ![BAR Sizing](https://github.com/Silverr12/DMA-CFW-Guide/assets/89455475/19239179-057a-4ed5-a79f-45cf242787a5)
     
     ![Subsystem ID](https://github.com/Silverr12/DMA-CFW-Guide/assets/89455475/94522a95-70bd-4336-8e38-58c0839e38ad)
     
     ![DSN](https://github.com/Silverr12/DMA-CFW-Guide/assets/89455475/595ae3e2-4cd8-4b3d-bcfa-cf6a59f289d5)
     

   - **Note**: The DSN may not be present on all devices. If unavailable, proceed with zeros in the DSN field during customization.

### **5.2 Extracting and Recording Device Attributes**

After scanning, ensure you have accurately recorded the following attributes from the donor device:

1. **Device ID**: A unique identifier for the hardware device.
2. **Vendor ID**: The identifier of the device manufacturer.
3. **Subsystem ID**: Identifies the specific subsystem associated with the device.
4. **Revision ID**: The revision number of the hardware version.
5. **Base Address Registers (BARs)**: Defines memory and I/O address regions of the device.
6. **Capabilities**: Such as Power Management (PM), MSI/MSI-X, PCIe link speed, and width.
7. **Device Serial Number (DSN)**: If applicable, the unique serial number associated with the device.

**Important Considerations**:

- **BAR Sizes**: Ensure the memory-mapped I/O regions match the donor device’s configuration.
- **Capabilities**: Properly emulate all capabilities to ensure seamless integration with the host system.
- **DSN**: Enhances the fidelity of emulation; use if available.

---

## **6. Initial Firmware Customization**

With the necessary donor information in hand, proceed to customize the PCIe configuration space and memory mapping within the firmware to spoof the donor device.

### **6.1 Modifying Configuration Space**

1. **Navigate to the Configuration File**
   - **Path**: `/PCIeSquirrel/src/pcileech_pcie_cfg_a7.sv`
   - **Description**: This Verilog file contains the PCIe configuration logic for the device.

2. **Open the File in Visual Studio**
   - **Steps**:
     1. Launch **Visual Studio**.
     2. Open the `pcileech_pcie_cfg_a7.sv` file located in the `/PCIeSquirrel/src/` directory.

3. **Modify Device ID and Vendor ID**
   - **Steps**:
     1. Use **Ctrl + F** to search for `cfg_deviceid`.
     2. Update the Device ID with the donor's value:
        ```verilog
        cfg_deviceid <= 16'hXXXX;  // Replace XXXX with donor Device ID
        ```
     3. Similarly, search for `cfg_vendorid` and update:
        ```verilog
        cfg_vendorid <= 16'hYYYY;  // Replace YYYY with donor Vendor ID
        ```

4. **Modify Subsystem ID**
   - **Steps**:
     1. Search for `cfg_subsysid`.
     2. Update the Subsystem ID:
        ```verilog
        cfg_subsysid <= 16'hZZZZ;  // Replace ZZZZ with donor Subsystem ID
        ```

5. **Adjust BARs Based on Donor Device**
   - **Steps**:
     1. Locate the BAR size configurations.
     2. Set the BAR sizes to match those of the donor device:
        ```verilog
        bar0_size <= 32'hXXXX_YYYY;  // Replace with donor's BAR0 size
        ```
     3. Repeat for additional BARs (BAR1, BAR2, etc.) as necessary.

   - **Example**:
     ```verilog
     bar0_size <= 32'h00004000;  // 16KB for BAR0
     ```

### **6.2 Inserting the Device Serial Number (DSN)**

If your donor device has a **Device Serial Number (DSN)**, incorporating it into the firmware enhances the emulation's fidelity.

1. **Locate the DSN Field**
   - **Steps**:
     1. In `pcileech_pcie_cfg_a7.sv`, search for `rw[127:64]`.
     2. This field represents the `cfg_dsn` (Configuration Space Device Serial Number).

2. **Insert the DSN**
   - **Steps**:
     1. Replace the placeholder with your donor device's DSN:
        ```verilog
        rw[127:64] <= 64'hXXXXXXXX_YYYYYYYY;  // Replace Xs and Ys with donor DSN
        ```
     2. **Example**:
        - **Donor DSN**: Upper DW: `01 00 00 00`, Lower DW: `68 4C E0 00`
        - **Combined DSN**: `64'h01000000684CE000`
        ```verilog
        rw[127:64] <= 64'h01000000684CE000;  // Donor DSN
        ```

   - **No DSN Available**:
     ```verilog
     rw[127:64] <= 64'h0000000000000000;  // No DSN
     ```

3. **Save Changes**
   - **Steps**:
     1. After modifying the DSN, save the file to retain changes.

---

## **7. Vivado Project Setup and Customization**

After customizing the configuration space, integrate these changes into the Vivado project to prepare the firmware for synthesis and implementation.

### **7.1 Generating Vivado Project Files**

1. **Open Vivado**
   - **Steps**:
     1. Launch **Vivado** on your development machine.
     2. Ensure Vivado is properly installed and configured for your FPGA device.

2. **Access the Tcl Console**
   - **Steps**:
     1. In Vivado, locate the **Tcl Console** at the bottom of the application window.
     2. If not visible, navigate to **Window > Tcl Console** to display it.

3. **Navigate to the PCIeSquirrel Directory**
   - **Steps**:
     1. In the Tcl Console, determine your current directory:
        ```tcl
        pwd
        ```
     2. Change the directory to the `PCIeSquirrel` folder within the cloned `pcileech-fpga` repository:
        ```tcl
        cd C:/Users/YourUsername/Desktop/pcileech-fpga/PCIeSquirrel
        ```
        *Replace `YourUsername` and the path as per your setup.*

     - **Note**: If you encounter errors with backslashes (`\`), use forward slashes (`/`):
       ```tcl
       cd C:/Users/YourUsername/Desktop/pcileech-fpga/PCIeSquirrel
       ```

4. **Generate the Vivado Project**
   - **Steps**:
     1. In the Tcl Console, execute the project generation script:
        ```tcl
        source vivado_generate_project.tcl -notrace
        ```
     2. Wait for the script to complete. This process sets up the Vivado project with the necessary configurations.

5. **Open the Generated Project**
   - **Steps**:
     1. Upon successful generation, Vivado should automatically open the `.xpr` (Vivado Project) file.
     2. Keep the project open for further customization.

### **7.2 Modifying IP Blocks**

1. **Access the PCIe IP Core**
   - **Steps**:
     1. In the **Sources** pane, navigate to:
        ```
        pcileech_squirrel_top > i_pcileech_pcie_a7 : pcileech_pcie_a7
        ```
     2. Double-click on the PCIe IP core (`i_pcie_7x_0 : pcie_7x_0`) to open the **Re-customize IP** window.

2. **Customize Device IDs and BARs**
   - **Steps**:
     1. In the **Re-customize IP** dialog, navigate to the **IDs** tab.
     2. Enter the **Device ID**, **Vendor ID**, and **Subsystem ID** gathered from the donor device.
     3. Verify the **Class Code**:
        - Go back to Arbor or your scanning tool to determine the class code of your donor device.
        - In the **Re-customize IP** window, set the class code accordingly to match the donor device.
     4. **Example**:
        - **Device ID**: `0x1234`
        - **Vendor ID**: `0xABCD`
        - **Subsystem ID**: `0x5678`
        - **Class Code**: `0x030000` (e.g., Network Controller)

3. **Configure BAR Sizes**
   - **Steps**:
     1. Navigate to the **BARs** tab within the **Re-customize IP** dialog.
     2. Set the **BAR0 Size** to match the donor device's BAR0 size.
        - **Example**: If the donor's BAR0 is 16KB:
          ```tcl
          BAR0 Size: 16KB
          ```
     3. Repeat for additional BARs (BAR1, BAR2, etc.) if the donor device utilizes them.

4. **Finalize IP Customization**
   - **Steps**:
     1. After setting all necessary parameters, click **OK** to apply the changes.
     2. Vivado may prompt to regenerate the IP core; confirm and allow the process to complete.

5. **Lock the IP Core**
   - **Purpose**: Prevent Vivado from overwriting manual configurations during synthesis.
   - **Steps**:
     1. Open the **Tcl Console** within Vivado.
     2. Execute the following command to lock the IP core:
        ```tcl
        set_property is_managed false [get_files pcie_7x_0.xci]
        ```
     3. **To Unlock** (if needed in the future):
        ```tcl
        set_property is_managed true [get_files pcie_7x_0.xci]
        ```

---

## **8. Advanced Firmware Customization**

To achieve precise 1:1 emulation, further customize PCIe parameters, BARs, memory mapping, power management, and interrupt handling.

### **8.1 Configuring PCIe Parameters for Emulation**

1. **Match PCIe Link Speed and Width**
   - **Importance**: Ensures the emulated device communicates at the same speed and width as the donor device.
   - **Steps**:
     1. In `pcileech_pcie_cfg_a7.sv`, locate the PCIe link speed and width configurations.
     2. Update these parameters to match the donor device's specifications.
        ```verilog
        pcie_link_speed <= 4'bXXXX;  // Replace XXXX with donor's PCIe link speed
        pcie_link_width <= 8'b00000100;  // Replace with donor's PCIe link width (e.g., x1, x4, x8)
        ```
     - **Example**:
       - **Donor PCIe Link Speed**: Gen3 (8 GT/s)
       - **Donor PCIe Link Width**: x4
         ```verilog
         pcie_link_speed <= 4'b0011;  // Gen3
         pcie_link_width <= 8'b00000100;  // x4
         ```

2. **Set Capability Pointers**
   - **Purpose**: Ensure the PCIe capabilities are correctly linked and recognized by the host system.
   - **Steps**:
     1. Locate the capability pointer configurations in `pcileech_pcie_cfg_a7.sv`.
     2. Set the capability pointers to match the donor device's configuration.
        ```verilog
        capability_pointer <= 8'h40;  // Example value; replace with donor's capability pointer
        ```

### **8.2 Adjusting BARs and Memory Mapping**

Accurate memory mapping is critical for emulating hardware devices. Base Address Registers (BARs) define where the device's memory and registers appear in system memory space.

1. **Set BAR Sizes**
   - **Steps**:
     1. In `pcileech_pcie_cfg_a7.sv`, locate the BAR size assignments.
     2. Set the BAR sizes to match those of the donor device.
        ```verilog
        bar0_size <= 32'h00004000;  // 16KB for BAR0
        bar1_size <= 32'h00008000;  // 32KB for BAR1 (if applicable)
        ```

2. **Define BAR Address Spaces**
   - **Steps**:
     1. Ensure the BAR address spaces do not overlap and match the donor device's memory layout.
     2. Use the recorded BAR sizes to set the address ranges appropriately.
        ```verilog
        bar0_addr <= 32'hF0000000;  // Example address; replace with donor's BAR0 address
        bar1_addr <= 32'hF0004000;  // Example address; replace as needed
        ```

3. **Handle Multiple BARs**
   - **Steps**:
     1. If the donor device uses multiple BARs, repeat the configuration for each BAR.
     2. Ensure each BAR's size and address align with the donor device's specifications.

### **8.3 Emulating Device Power Management and Interrupts**

Properly emulating power management and interrupt handling ensures the host system interacts seamlessly with the emulated device.

1. **Power Management (PM) Configuration**
   - **Steps**:
     1. In `pcileech_pcie_cfg_a7.sv`, locate the Power Management capability settings.
     2. Set the PM capabilities to match the donor device.
        ```verilog
        PM_CAP_VERSION <= 4'b0011;  // Example version; replace with donor's PM version
        PM_CAP_D1SUPPORT <= 1'b1;   // Enable D1 support if the donor does
        PM_CAP_AUXCURRENT <= 4'b1000; // Example value; adjust as per donor
        PM_CSR_NOSOFTRST <= 1'b0;   // Example value; adjust as needed
        ```

2. **MSI/MSI-X (Interrupts) Configuration**
   - **Steps**:
     1. Locate MSI/MSI-X configuration in `pcileech_pcie_cfg_a7.sv`.
     2. Enable and configure MSI/MSI-X to handle interrupts correctly.
        ```verilog
        MSI_CAP_64_BIT_ADDR_CAPABLE <= 1'b1;  // Enable 64-bit MSI if supported
        cfg_interrupt <= 1'b1;               // Enable MSI interrupts
        ```

3. **Implementing Interrupt Handling Logic**
   - **Steps**:
     1. In `pcileech_pcie_cfg_a7.sv`, ensure the interrupt signals are correctly routed.
        ```verilog
        assign cfg_interrupt_di = cfg_int_di;
        assign cfg_interrupt_assert = cfg_int_assert;
        ```
     2. Test interrupt functionality to ensure the host system correctly receives and handles interrupts from the emulated device.

---

## **9. Emulating Device-Specific Capabilities**

To achieve a true 1:1 emulation, it's essential to replicate the unique capabilities of the donor device beyond basic PCIe interactions.

### **9.1 Implementing Advanced PCIe Capabilities**

Most PCIe devices support advanced features like **Advanced Error Reporting (AER)**, **Link Speed Negotiation**, and **Extended Capabilities**. Emulating these ensures the host system perceives the emulated device as identical to the donor.

1. **Advanced Error Reporting (AER)**
   - **Steps**:
     1. In `pcileech_pcie_cfg_a7.sv`, locate AER configurations.
     2. Enable AER if supported by the donor device.
        ```verilog
        AER_CAP_VERSION <= 4'b0001;  // Example version; replace with donor's AER version
        AER_CAP_NEXTPTR <= 8'h00;    // Set next pointer appropriately
        ```
     3. Implement error handling logic to manage AER-related events.

2. **Link Speed Negotiation**
   - **Steps**:
     1. Ensure the PCIe link speed and width negotiation matches the donor device.
     2. Adjust link speed settings as previously outlined in **8.1**.

3. **Extended Capabilities**
   - **Steps**:
     1. Identify any extended capabilities used by the donor device (e.g., Vendor-Specific Extended Capabilities, LTR, VSEC).
     2. Implement these capabilities within `pcileech_pcie_cfg_a7.sv` by defining the appropriate registers and logic.
        ```verilog
        // Example for Vendor-Specific Extended Capability
        VSEC_CAP_ID <= 16'hXXXX;       // Replace XXXX with vendor-specific ID
        VSEC_CAP_VERSION <= 8'hYY;    // Replace YY with version
        VSEC_CAP_NEXTPTR <= 8'hZZ;    // Next capability pointer
        ```

### **9.2 Emulating Vendor-Specific Features**

Some devices incorporate proprietary or vendor-specific features that must be accurately emulated to ensure seamless integration.

1. **Identify Vendor-Specific Features**
   - **Steps**:
     1. Use PCIe traffic analysis tools (e.g., Wireshark, Teledyne LeCroy) to monitor vendor-specific TLPs.
     2. Document unique registers, commands, or behaviors exhibited by the donor device.

2. **Implementing Vendor-Specific Logic**
   - **Steps**:
     1. In `pcileech_pcie_cfg_a7.sv`, add logic to handle vendor-specific features.
        ```verilog
        // Example: Vendor-Specific Register
        vendor_specific_reg <= 32'hXXXXXXXX;  // Replace with actual value
        ```
     2. Ensure that any proprietary commands or responses are accurately replicated.

3. **Testing Vendor-Specific Features**
   - **Steps**:
     1. Use vendor-specific drivers or applications to interact with the emulated device.
     2. Verify that all proprietary features function as expected.

---

## **10. Transaction Layer Packet (TLP) Emulation**

Accurate emulation of Transaction Layer Packets (TLPs) is vital for ensuring the FPGA-based device communicates seamlessly with the host system, mimicking the behavior of the donor device.

### **10.1 Understanding and Capturing TLPs**

TLPs are the fundamental units of PCIe communication, handling memory reads/writes, configuration accesses, and interrupt signaling.

1. **Capture TLPs from the Donor Device**
   - **Steps**:
     1. Use PCIe analysis tools like **Teledyne LeCroy’s Telescan PE** or **Wireshark** with PCIe support to monitor TLPs generated by the donor device.
     2. Record the structure, types, and patterns of TLPs used by the donor device during typical operations.

2. **Analyze TLP Structure**
   - **Components of a TLP**:
     - **Header Fields**: Define the type, format, address, and other control information.
     - **Data Payload**: The actual data being transferred.
     - **Tail Fields**: Additional information such as byte counts and sequence numbers.

   - **Example TLP Structure**:
     ```verilog
     tlps_static.tdata[127:0] = {TLP header fields, Data Payload};
     ```

3. **Emulating Legitimate Traffic**
   - **Steps**:
     1. Ensure that TLPs generated by the FPGA match those captured from the donor device in terms of type, address, length, and data.
     2. Implement logic to handle different types of TLPs, such as memory writes, memory reads, and configuration accesses.

### **10.2 Crafting Custom TLPs for Specific Operations**

To accurately mimic the donor device, you must craft custom TLPs that replicate its behavior during various operations.

1. **Memory Write TLP Example**
   - **Description**: Represents a write operation to system memory.
   - **Verilog Example**:
     ```verilog
     tlp_mem_write <= {
         1'b0,                    // Reserved
         7'b10_00000,             // TLP Type: Memory Write
         1'b0,                    // TLP Format
         address[31:0],           // Address being written to
         data[31:0]               // Data payload
     };
     ```

2. **Memory Read TLP Example**
   - **Description**: Represents a read operation from system memory.
   - **Verilog Example**:
     ```verilog
     tlp_mem_read <= {
         1'b0,                    // Reserved
         7'b00_00000,             // TLP Type: Memory Read
         1'b0,                    // TLP Format
         address[31:0],           // Address being read from
         tag[7:0]                 // Tag for transaction identification
     };
     ```

3. **Configuration Access TLP Example**
   - **Description**: Represents a configuration space access.
   - **Verilog Example**:
     ```verilog
     tlp_config_access <= {
         1'b0,                    // Reserved
         7'b01_00000,             // TLP Type: Configuration Read/Write
         1'b0,                    // TLP Format
         config_address[31:0],    // Configuration space address
         config_data[31:0]        // Configuration data payload
     };
     ```

4. **Interrupt Signaling TLP Example**
   - **Description**: Represents an interrupt signaling to the CPU.
   - **Verilog Example**:
     ```verilog
     tlp_interrupt <= {
         1'b0,                    // Reserved
         7'b11_00000,             // TLP Type: Interrupt
         1'b0,                    // TLP Format
         interrupt_address[31:0], // Address related to the interrupt
         interrupt_data[31:0]     // Interrupt data payload
     };
     ```

5. **Implement TLP Handlers**
   - **Steps**:
     1. In your firmware, implement handlers for different TLP types to ensure correct processing and response.
     2. Use state machines or logic blocks to manage TLP generation, processing, and response handling.

---

## **11. Building, Flashing, and Testing**

After customizing the firmware and ensuring all configurations align with the donor device, proceed to build, flash, and test the firmware on your FPGA device.

### **11.1 Synthesis and Implementation**

1. **Run Synthesis**
   - **Steps**:
     1. In Vivado, click on **Run Synthesis**.
     2. Monitor the synthesis process for any warnings or errors.
     3. Address any critical issues before proceeding.

2. **Run Implementation**
   - **Steps**:
     1. After successful synthesis, initiate **Run Implementation**.
     2. Ensure that the implementation phase completes without critical warnings.
     3. Review the implementation report for any potential issues.

3. **Generate Bitstream**
   - **Steps**:
     1. Once implementation is complete, click on **Generate Bitstream**.
     2. Confirm any prompts to generate the bitstream.
     3. Wait for the bitstream generation to finish successfully.

### **11.2 Flashing the Bitstream**

1. **Connect FPGA via JTAG**
   - **Steps**:
     1. Ensure your FPGA device is connected to the host system via the JTAG interface.
     2. Power on the FPGA device.

2. **Open Vivado Hardware Manager**
   - **Steps**:
     1. In Vivado, navigate to **Window > Hardware Manager**.
     2. Click **Open Target > Auto Connect** to detect the connected FPGA device.

3. **Program the FPGA**
   - **Steps**:
     1. In the Hardware Manager, right-click on the detected device and select **Program Device**.
     2. Browse to the generated bitstream file (`pcileech_squirrel_top.bit`).
     3. Click **Program** to flash the firmware onto the FPGA.
     4. Confirm successful programming via the Hardware Manager console.

### **11.3 Testing and Validation**

1. **Verify Device Detection**
   - **Steps**:
     1. Use **lspci** (on Linux) or **Device Manager** (on Windows) to verify that the FPGA is detected as the donor device.
     2. Confirm that the **Device ID**, **Vendor ID**, **Subsystem ID**, and **BARs** match the donor device's specifications.

   - **Example (Linux)**:
     ```bash
     lspci -vvv -s <PCI address>
     ```

2. **Memory Mapping Test**
   - **Steps**:
     1. Access the device's **BARs** to ensure correct memory mapping.
     2. Use memory access tools or simple read/write operations to test responsiveness.

3. **Interrupts Test**
   - **Steps**:
     1. Trigger interrupts through the emulated device.
     2. Verify that the host system correctly receives and handles these interrupts.
     3. Use system logs or diagnostic tools to confirm interrupt handling.

4. **Performance Testing**
   - **Steps**:
     1. Run DMA speed test tools to measure data transfer rates.
     2. Compare performance metrics against expected values to ensure firmware stability and efficiency.

   - **Example Tools**:
     - **PCILeech DMA Speed Test**: Available within the PCILeech toolset.
     - **Custom Benchmark Scripts**: Scripts that perform read/write operations to measure performance.

5. **Configuration Space Validation**
   - **Steps**:
     1. Use diagnostic tools to inspect the PCIe configuration space.
     2. Ensure all fields (Device ID, Vendor ID, BARs, Capabilities) are correctly set and match the donor device.

   - **Example (Linux)**:
     ```bash
     lspci -vvv -s <PCI address>
     ```

---

## **12. Advanced Debugging Techniques**

When developing custom firmware, encountering issues is common. Advanced debugging techniques can help identify and resolve these problems effectively.

### **12.1 Using Vivado's Integrated Logic Analyzer**

Vivado's **Integrated Logic Analyzer (ILA)** allows real-time monitoring of internal FPGA signals, aiding in debugging and verification.

1. **Set Up ILA Probes**
   - **Steps**:
     1. In Vivado, navigate to **Sources > Add Sources** and add an **Integrated Logic Analyzer**.
     2. Insert ILA probes at critical points in the PCIe communication path, such as TLP generators or BAR access controllers.
        ```verilog
        // Example: Adding ILA probe for TLP data
        wire [127:0] tlp_data;
        assign tlp_data = tlps_static.tdata[127:0];
        ila_0 probe (
            .clk(clk),
            .probe0(tlp_data)
        );
        ```

2. **Configure Triggers**
   - **Steps**:
     1. Open the **ILA** configuration dialog.
     2. Set trigger conditions based on specific events, such as TLP generation or memory access.
        ```verilog
        // Example trigger condition: Start of memory write TLP
        if (tlp_type == MEMORY_WRITE && tlp_valid) begin
            trigger_signal <= 1;
        end
        ```

3. **Analyze Signal Waveforms**
   - **Steps**:
     1. Run the FPGA with the ILA probes enabled.
     2. Use Vivado’s **Waveform Viewer** to examine captured signal waveforms.
     3. Identify timing issues, incorrect logic states, or unexpected behaviors.

   - **Benefits**:
     - Real-time visibility into internal signals.
     - Ability to capture and analyze transient issues during TLP processing.

### **12.2 PCIe Traffic Analysis Tools**

Beyond Vivado's ILA, external PCIe traffic analysis tools provide in-depth insights into PCIe communications between the FPGA and the host system.

1. **Wireshark with PCIe Extensions**
   - **Description**: Wireshark can capture and analyze PCIe traffic with the appropriate extensions or plugins.
   - **Steps**:
     1. Install Wireshark with PCIe support.
     2. Configure Wireshark to capture PCIe traffic.
     3. Analyze captured TLPs to ensure they align with expected donor device behavior.

2. **Teledyne LeCroy Telescan PE**
   - **Description**: A professional-grade PCIe traffic analysis tool offering comprehensive PCIe traffic monitoring and analysis capabilities.
   - **Steps**:
     1. Install Teledyne LeCroy’s Telescan PE.
     2. Connect it to your system to monitor PCIe traffic.
     3. Use it to capture and dissect TLPs exchanged between the FPGA and host system.

3. **Total Phase Beagle**
   - **Description**: A PCIe traffic analyzer that allows for real-time capture and analysis of PCIe communications.
   - **Steps**:
     1. Set up the Total Phase Beagle PCIe analyzer with your system.
     2. Configure it to monitor and capture PCIe traffic.
     3. Use its analysis features to verify TLP integrity and behavior.

**Benefits of Using PCIe Traffic Analysis Tools**:

- **Comprehensive TLP Analysis**: Detailed inspection of TLPs to ensure accurate emulation.
- **Error Detection**: Identify malformed TLPs or unexpected transaction patterns.
- **Performance Metrics**: Measure data transfer rates and identify bottlenecks.

---

## **13. Troubleshooting**

Encountering issues during firmware development is common. This section provides solutions to common problems you may face during the emulation process.

### **13.1 Device Detection Issues**

**Problem**: The host system fails to detect the FPGA as the donor device.

**Solutions**:

1. **Verify Device IDs**
   - **Steps**:
     1. Double-check that the **Device ID**, **Vendor ID**, and **Subsystem ID** in the firmware match those of the donor device.
     2. Ensure there are no typos or incorrect values in the configuration space.

2. **Check PCIe Link Training**
   - **Steps**:
     1. Use PCIe diagnostic tools to verify that the PCIe link is properly trained.
     2. Ensure that the link speed and width configurations match the donor device.

3. **Ensure Correct BAR Configuration**
   - **Steps**:
     1. Confirm that the **BAR sizes** and **address ranges** are accurately set.
     2. Ensure no overlapping or conflicting BAR configurations.

4. **Power and Connection Check**
   - **Steps**:
     1. Ensure the FPGA device is properly connected and powered.
     2. Re-seat the PCIe card to ensure a secure connection.

### **13.2 Memory Mapping and BAR Configuration Errors**

**Problem**: Incorrect memory mapping leads to failed or inaccurate memory access.

**Solutions**:

1. **Double-Check BAR Sizes and Addresses**
   - **Steps**:
     1. Verify that each BAR size in the firmware matches the donor device's configuration.
     2. Ensure that BAR address spaces are correctly set and do not overlap.

2. **Use Diagnostic Tools**
   - **Steps**:
     1. Utilize tools like **lspci** or **Arbor** to inspect the PCIe configuration space.
     2. Confirm that the BARs are correctly mapped and accessible.

3. **Adjust Memory Regions**
   - **Steps**:
     1. If memory regions are not accessible, adjust the BAR configurations to better match the system's memory map.
     2. Ensure that the firmware logic correctly handles memory read/write operations.

### **13.3 DMA Performance and TLP Errors**

**Problem**: Slow DMA performance or errors related to Transaction Layer Packets (TLPs).

**Solutions**:

1. **Optimize TLP Generation**
   - **Steps**:
     1. Ensure that TLPs are correctly formatted and free of errors.
     2. Use Vivado’s ILA and PCIe traffic analysis tools to identify and rectify malformed TLPs.

2. **Adjust Payload Sizes**
   - **Steps**:
     1. Set the maximum read request and payload sizes to 4KB or the highest supported by the donor device.
        ```verilog
        max_read_request_size <= 4;  // 4KB
        max_payload_size <= 4;       // 4KB
        ```
     2. Avoid setting payload sizes beyond what the donor device supports to prevent system instability.

3. **Check PCIe Link Settings**
   - **Steps**:
     1. Verify that the PCIe link speed and width are correctly configured.
     2. Ensure that the FPGA is negotiating the link parameters accurately with the host system.

4. **Firmware Integrity**
   - **Steps**:
     1. Review and validate all recent changes to the firmware to ensure no unintended modifications were introduced.
     2. Revert to a known stable firmware version if performance issues persist.

---

## **14. Emulation Accuracy and Optimizations**

Ensuring the emulation's accuracy is critical for seamless integration and undetectable behavior. This section outlines techniques to enhance emulation precision and optimize performance.

### **14.1 Techniques for Accurate Timing Emulation**

Matching the donor device's timing characteristics ensures that the host system interacts with the emulated device as if it were the original hardware.

1. **Use Matching Clock Domains**
   - **Steps**:
     1. Ensure that the FPGA’s clock matches the PCIe link’s clock rate.
     2. Synchronize internal clocks within the FPGA to align with PCIe timing requirements.

2. **Control Response Latency**
   - **Steps**:
     1. Implement registers or counters to manage response times for TLP acknowledgments and interrupt handling.
     2. Ensure that the latency in responses matches the donor device’s typical response times.

3. **Implement Pipeline Stages**
   - **Steps**:
     1. Use pipelining in the FPGA design to align with the donor device’s data processing stages.
     2. This reduces latency and ensures timely TLP generation and processing.

### **14.2 Dynamic Response to System Calls**

Emulating dynamic device behavior based on system interactions ensures the FPGA device responds appropriately under various conditions.

1. **Implement State Machines**
   - **Steps**:
     1. Design state machines within the FPGA to manage different operational states of the emulated device.
     2. Ensure transitions between states mimic the donor device’s behavior based on system calls and interactions.

2. **Track and Respond to System Requests**
   - **Steps**:
     1. Monitor incoming system requests and adjust the device’s responses dynamically.
     2. Ensure that the FPGA firmware can handle varying workloads and respond accurately to different types of TLPs.

3. **Handle Asynchronous Events**
   - **Steps**:
     1. Implement logic to manage asynchronous events such as interrupts or error conditions.
     2. Ensure that the firmware can generate and respond to these events in a manner consistent with the donor device.

---

## **15. Best Practices for Firmware Development**

Adhering to best practices ensures the development process is efficient, maintainable, and secure.

### **15.1 Continuous Testing and Documentation**

- **Test Frequently**
  - **Steps**:
    1. Conduct regular tests after each modification to ensure the firmware behaves as expected.
    2. Use automated scripts or test benches to validate firmware functionality continuously.

- **Document Changes**
  - **Steps**:
    1. Maintain detailed documentation for each change made to the firmware.
    2. Include explanations for why changes were made and their impact on the overall design.

### **15.2 Managing Firmware Versioning**

- **Use Version Control**
  - **Steps**:
    1. Implement a version control system (e.g., **Git**) to manage different iterations of the firmware.
    2. Commit changes regularly with descriptive messages to track the evolution of the project.

- **Branching Strategy**
  - **Steps**:
    1. Use branches to manage feature development, bug fixes, and experimental changes.
    2. Merge stable branches into the main branch only after thorough testing.

### **15.3 Security Considerations**

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

---

## **16. Additional Resources**

To further enhance your understanding and capabilities in developing custom firmware for device emulation, the following resources are invaluable:

- **PCILeech-FPGA Repository**
  - **Link**: [https://github.com/ufrisk/pcileech-fpga](https://github.com/ufrisk/pcileech-fpga)

- **Vivado FPGA Documentation**
  - **Link**: [Xilinx Vivado Documentation](https://www.xilinx.com/support/documentation.html)

- **PCI-SIG Specifications**
  - **Link**: [PCI-SIG](https://pcisig.com)

- **PCIe TLP Primer Tutorial**
  - **Link**: [PCIe TLP Primer](https://www.xillybus.com/tutorials/pci-express-tlp-pcie-primer-tutorial-guide-1)

- **Teledyne LeCroy Telescan PE Documentation**
  - **Link**: [Teledyne LeCroy Telescan PE](https://www.teledynelecroy.com/protocolanalyzer/pci-express/telescan-pe-software/resources/analysis-software)

- **Wireshark PCIe Extensions**
  - **Link**: [Wireshark Extensions](https://www.wireshark.org/docs/)

- **Field Programmable Gate Array (FPGA) Basics**
  - **Link**: [FPGA Basics](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2020_2/ug901-vivado-tutorial.pdf)

- **Arbor Software User Guide**
  - **Link**: [Arbor User Guide](https://www.mindshare.com/software/Arbor)

- **PCIe Specifications and Guides**
  - **Link**: [PCIe Specifications](https://pcisig.com/specifications)
