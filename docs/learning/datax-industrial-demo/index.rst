DEMO ADI DataX AI-driven 10BASE-T1L Deployment
===============================================================================

Deploying a functional 10BASE-T1L network for industrial applications requires navigating firmware builds, board-specific configurations, IP assignment, and multi-device coordination - an expertise barrier that constrains adoption despite the technology's inherent advantages in delivering power and data over a single twisted pair. ADI addresses this with a complete reference platform combining hardware, software, and agentic AI to streamline industrial network deployment through intelligent automation.

This demonstration presents a complete 10BASE-T1L reference architecture spanning five board types across four network subnets. ADI DataX components such as no-OS, libiio, pyadi-iio, integrated with a Claude-based AI agent, automate end-to-end system bring-up, substantially reducing manual integration overhead through a structured, reproducible workflow:

- AD-RPI-T1L-PSE provides power sourcing and serves as the network entry point

- AD-SWIOT1L-SL operates as the T1L field device with configurable I/O and sensor interfaces

- AD-APARD32690-SL with AD-APARDSPOE-SL and AD-APARDPFW-SL shields provides application processing, power forwarding, and network extension

- CN0575 extends the network as an additional T1L node

- The AI agent orchestrates firmware builds, flash sequencing, IP assignment, interface configuration, and connectivity verification across all nodes

As a proof of concept, a table-top scale sorting conveyor belt is designed to operate on the fully configured network, validating the deployment stack from unconfigured hardware to a running industrial application – highlighting a repeatable, production-ready path to 10BASE-T1L adoption.

Resources
-------------------------------------------------------------------------------

- no-OS branch: :git-no-os:`adi_datax_10baset1l_demo <adi_datax_10baset1l_demo:>`
- pyadi-iio branch: :git-pyadi-iio:`adi_datax_10baset1l_demo <adi_datax_10baset1l_demo:>`
- Claude-based AI agent: :git-ai-agent:`adi_datax_10baset1l_demo <adi_datax_10baset1l_demo:>`
- Application code for conveyor belt demo: :git-conveyor:`adi_datax_10baset1l_demo <adi_datax_10baset1l_demo:>`
- Kuiper Linux branch: :git-kuiper:`adi_datax_10baset1l_demo <adi_datax_10baset1l_demo:>`

Block diagram
-------------------------------------------------------------------------------
.. figure:: block.drawio.png
   :align: center
   :width: 900

Demo description
-------------------------------------------------------------------------------
The Raspberry Pi 4 with the AD-RPI-T1LPSE-SL hat runs Kuiper Linux and serves as the main 
controlling unit to three network segments. The AD-SWIOT1L-SL board is connected to the main
RPi using T1L-to-USB on eth1, the two AD-APARD32690 with AD-APARDSPOE-SL shield and AD-
APARDPFWD-SL shield are daisy-chained and share eth2 on the same subnet. T
he second RPi with EVAL-CN0575-RPIZ conects via eth3. 
All links use 10BASE-T1L for industrial connectivity.

System Capabilities
-------------------------------------------------------------------------------

Required Hardware
-------------------------------------------------------------------------------

SD Card Configuration
----------------------------------------------------------------------------

Software Setup
-------------------------------------------------------------------------------

Demo Features
-------------------------------------------------------------------------------
