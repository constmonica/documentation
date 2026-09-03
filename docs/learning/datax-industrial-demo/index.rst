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

- no-OS branch: `AD-APARD32690-SL firmware <https://github.com/GanscaTudor/no-OS/tree/apardspoe-color-sensor>`__
- pyadi-iio branch: `AD-SWIOT1L-SL control <https://github.com/constmonica/pyadi-iio/tree/swiot>`__
- Kuiper Linux branch: :git-kuiper:`AD-RPI-T1LPSE-SL board support <kuiper-AD-RPI-T1LPSE-SL:>`
- Demo application: `Conveyor belt control panel <https://github.com/GanscaTudor/industrial-demo/tree/main/RPI_T1LPSE>`__
- Claude-based AI agent: ``adi_datax_10baset1l_demo``

Block Diagram
-------------------------------------------------------------------------------

.. figure:: block.drawio.png
   :align: center
   :width: 900

Demo Description
-------------------------------------------------------------------------------

The Raspberry Pi 4 with the AD-RPI-T1LPSE-SL hat runs Kuiper Linux and serves as the main
controlling unit to three network segments. The AD-SWIOT1L-SL board is connected to the main
RPi using T1L-to-USB on eth1, the two AD-APARD32690 with AD-APARDSPOE-SL shield and
AD-APARDPFWD-SL shield are daisy-chained and share eth2 on the same subnet.
The second RPi with EVAL-CN0575-RPIZ connects via eth3.
All links use 10BASE-T1L for industrial connectivity.

Reference Design
-------------------------------------------------------------------------------

The system uses a central control hub architecture and distributed nodes connected via
10BASE-T1L, carrying power and data over a single twisted pair.

Required Hardware
-------------------------------------------------------------------------------

.. csv-table::
   :file: hardware_table.csv
   :widths: 30, 55, 15
   :header-rows: 1

Software Setup
-------------------------------------------------------------------------------

System Setup
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Build the custom RPI-T1LPSE Kuiper image:

   .. code-block:: bash

      git clone https://github.com/analogdevicesinc/kuiper -b kuiper-AD-RPI-T1LPSE-SL
      cd kuiper
      sudo ./build-docker.sh

   You will find the image in */kuiper-volume*. Flash it to the SD card using your preferred
   method. Insert the SD card into the main Raspberry Pi 4.

#. Set up the device-tree overlay.

   In order to enable the T1L PHY on the RPI-T1LPSE-SL board, you need to add the following
   line to the */boot/config.txt* file on the Raspberry Pi 4, then reboot the system for the
   changes to take effect.

   .. code-block:: bash

      dtoverlay=rpi-tl1pse-class12

#. Set up the EVAL-CN0575-RPIZ.

   To use the EVAL-CN0575-RPIZ board with Raspberry Pi 4, download and flash a prebuilt
   Kuiper image to an SD card and insert it into the second Raspberry Pi 4.

   .. admonition:: Download

      Go to `GitHub Actions`_ and download it.

   The *rpi-cn0575-adxl355* is a custom overlay (not part of the standard Kuiper overlay
   set). It needs to be compiled from source before it can be enabled. To do this, follow
   the steps below:

   .. code-block:: bash

      git clone https://github.com/ganscatudor/industrial-demo
      dtc -@ -I dts -O dtb -o rpi-cn0575-adxl355-overlay.dtbo RPI_CN0575/rpi-cn0575-adxl355-overlay.dts
      sudo cp rpi-cn0575-adxl355-overlay.dtbo /boot/overlays/

   Add the following line to the */boot/config.txt* file on the second Raspberry Pi 4, then
   reboot the system for the changes to take effect.

   .. code-block:: bash

      dtoverlay=rpi-cn0575-adxl355

.. _GitHub Actions: https://github.com/analogdevicesinc/kuiper/actions/workflows/kuiper2_0-build.yml?query=branch:main

Install Prerequisites
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Install build tools, libraries and SDKs on the main Raspberry Pi 4.

#. **Build tools and ARM cross-compiler**

   .. code-block:: bash

      sudo apt update
      sudo apt install -y git make gcc-arm-none-eabi libnewlib-arm-none-eabi

#. **MaximSDK** (headers and libraries only)

   .. code-block:: bash

      git clone https://github.com/analogdevicesinc/msdk.git ~/MaximSDK

   Create a GNUTools symlink so the no-OS build system can find the compiler.

   .. code-block:: bash

      mkdir -p ~/MaximSDK/Tools/GNUTools/10.3/bin
      ln -s /usr/bin/arm-none-eabi-* ~/MaximSDK/Tools/GNUTools/10.3/bin

   Add the environment variable to the *~/.bashrc* file.

   .. code-block:: bash

      echo 'export MAXIM_LIBRARIES=~/MaximSDK/Libraries' >> ~/.bashrc
      source ~/.bashrc

#. **no-OS**

   Clone the GitHub repository:

   .. code-block:: bash

      git clone --recursive https://github.com/GanscaTudor/no-OS ~/no-OS

#. **libiio** (v0 branch)

   .. code-block:: bash

      sudo apt-get install -y libxml2 libxml2-dev bison flex libcdk5-dev cmake \
          libaio-dev libusb-1.0-0-dev libserialport-dev libavahi-client-dev
      git clone https://github.com/analogdevicesinc/libiio.git --branch libiio-v0 ~/libiio
      cd ~/libiio && mkdir build && cd build
      cmake .. -DPYTHON_BINDINGS=ON
      make -j && sudo make install
      sudo ldconfig

#. **pyadi-iio**

   .. code-block:: bash

      sudo apt-get install -y python3 libatlas-base-dev
      git clone https://github.com/constmonica/pyadi-iio.git ~/pyadi-iio
      cd pyadi-iio
      git checkout swiot
      sudo python3 -m pip install -r requirements_prod_test.txt
      sudo pip install .

#. **network-manager**

   .. code-block:: bash

      sudo apt-get install -y network-manager
      sudo systemctl enable NetworkManager
      sudo systemctl start NetworkManager

#. **OpenOCD**

   .. code-block:: bash

      sudo apt-get install build-essential libtool pkg-config libusb-1.0-0-dev \
          libhidapi-dev libgpiod-dev
      git clone https://github.com/analogdevicesinc/openocd -b "0.12.0-1.1.1.2" --depth 1 --recurse-submodules
      cd openocd
      ./bootstrap
      ./configure --enable-cmsisdap --enable-linuxgpiod --disable-werror
      make -j
      sudo make install

Building the Firmware
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In this setup phase, you need to build the firmware for the two AD-APARD32690-SL with different
configurations for each shield (AD-APARDSPoE-SL and AD-APARDPFW-SL). You also need to download
the AD-SWIOT1L-SL static IP firmware.

#. **AD-APARD32690 #1 with AD-APARDPFWD-SL shield** (ip: 192.168.98.50)

   .. code-block:: bash

      cd ~/no-OS/projects/apardpfwd/src/examples
      make clean
      make RELEASE=y EXAMPLE=forward_packets_example -j
      cp build/apardspoe.elf /home/analog/apardspoe.elf

#. **AD-APARD32690-SL #2 with AD-APARDSPOE-SL shield** (ip: 192.168.98.60)

   .. code-block:: bash

      cd ~/no-OS/projects/apardspoe/src/examples
      make clean
      make RELEASE=y EXAMPLE=color_sensors_example -j
      cp build/apardspoe.elf /home/analog/apardspoe.elf

#. **AD-SWIOT1L-SL** (ip: 192.168.97.40)

   .. code-block:: bash

      wget -O /home/analog/swiot1l_static_ip.hex https://github.com/analogdevicesinc/no-OS/releases/download/swiot1l-v1.1.0/swiot1l_maxim_swiot1l_static_ip.hex

Flashing the Firmware
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. important::

   All the boards must be flashed with the firmware before customizing the network settings.

All the boards are flashed using OpenOCD with MAXIM CMSIS-DAP programmer.
Connect the CMSIS-DAP programmer to the Raspberry Pi 4 and connect the target board to the
programmer using the SWD interface.

#. For AD-APARD32690-SL with AD-APARDPFWD-SL shield, run the following command:

   .. code-block:: bash

      openocd -f interface/cmsis-dap.cfg -f target/maxim_max32690.cfg -c "program /home/analog/apardpfwd.elf verify reset exit"

#. For AD-APARD32690-SL with AD-APARDSPOE-SL shield, run the following command:

   .. code-block:: bash

      openocd -f interface/cmsis-dap.cfg -f target/maxim_max32690.cfg -c "program /home/analog/apardspoe.elf verify reset exit"

#. For AD-SWIOT1L-SL, run the following command:

   .. code-block:: bash

      openocd -f interface/cmsis-dap.cfg -f target/maxim_max32690.cfg -c "program /home/analog/swiot1l_static_ip.hex verify reset exit"

Network Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This demo uses multiple network interfaces and subnets. The main Raspberry Pi 4 is connected to the AD-SWIOT1L-SL board via eth1,
to the two AD-APARD32690-SL boards via eth2, and to the second Raspberry Pi 4 with EVAL-CN0575-RPIZ via eth3.

.. tip::

   The network configuration is done using NetworkManager. You can use the ``nmcli``
   command to configure the network interfaces and assign static IP addresses.

The AD-RPI-T1LPSE-SL hat creates an Ethernet interface for the 10BASE-T1L network. Assign static IP addresses to the network interface connected to the AD-APARD32690-SL boards.

#. Find the interface name:

   .. code-block:: bash

      nmcli device status

#. If the APARD32690-SL boards are connected to eth2, assign the static IP address to match
   the subnet of the boards (192.168.98.x):

   .. code-block:: bash

      sudo nmcli connection add type ethernet con-name t1l-apard \
          ifname eth2 \
          ipv4.method manual \
          ipv4.addresses 192.168.98.1/24
      sudo nmcli connection up t1l-apard

#. The CN0575-RPIZ board is connected to the second Raspberry Pi 4 via eth3. Assign a static
   IP address to the network interface connected to the CN0575-RPIZ board (192.168.10.x
   subnet):

   .. code-block:: bash

      sudo nmcli connection add type ethernet con-name t1l-cn0575 \
          ifname eth3 \
          ipv4.method manual \
          ipv4.addresses 192.168.10.1/24
      sudo nmcli connection up t1l-cn0575

#. The AD-SWIOT1L-SL board is connected via T1L-to-USB on eth1. Assign a static IP address
   to the network interface connected to the AD-SWIOT1L-SL board (192.168.97.x subnet):

   .. code-block:: bash

      sudo nmcli connection add type ethernet con-name t1l-swiot1l \
          ifname eth1 \
          ipv4.method manual \
          ipv4.addresses 192.168.97.1/24
      sudo nmcli connection up t1l-swiot1l

#. Verify the network configuration by pinging the boards from the main Raspberry Pi 4:

   .. code-block:: bash

      ping -c 3 192.168.98.50  # AD-APARD32690-SL #1 (PFWD)
      ping -c 3 192.168.98.60  # AD-APARD32690-SL #2 (SPoE)
      ping -c 3 192.168.97.40  # AD-SWIOT1L-SL
      ping -c 3 192.168.10.2   # Raspberry Pi 4 + EVAL-CN0575-RPIZ

.. tip::

   A successful check ends with ``3 packets transmitted, 3 received, 0% packet loss``.
   If you see 100% packet loss, the addresses above are specific to this setup — confirm
   the board's actual IP, netmask, and gateway, which are printed on its serial console
   at boot via the debug adapter.

Run the Demo
-------------------------------------------------------------------------------

The demo application has a single control panel for the 10BASE-T1L industrial demo. It brings 5
subsystems into one window: a vibration monitor, a color-based sorting line, temperature monitoring,
a fan PWM controller - all reached over the T1L network.

On the main Raspberry Pi 4, run the demo script to start the conveyor belt application and observe the system operation.

.. code-block:: bash

   git clone https://github.com/ganscatudor/industrial-demo ~/industrial-demo
   cd ~/industrial-demo
   sudo pip install matplotlib pyadi-iio numpy

.. code-block:: bash

   python3 main_app.py --demo # synthetic data, # no hardware needed
   python3 main_app.py --adxl-host 192.168.10.2 # points to a remote ADXL355 server

**Demo mode** is the fastest way to see the whole application running without any hardware.
Responses match the real device protocol exactly, so every panel, graph and the sorting logic
behave as they do against real hardware.

Control Panel
-------------------------------------------------------------------------------

.. figure:: main_app.png
   :align: center
   :width: 900

Every panel has a **Status** row with a **Test** (or **Connect**) button that
verifies the link to its board. The header's theme toggle switches light/dark at
any time. **Test All** in the log header tests every board at once.

Panels
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. **ADXL355 - Predictive maintenance**

   Streams three-axis vibration and plots a live FFT.

   - **Start Servers** - brings up the Pi-side data server over SSH so you don't
     need a terminal on the second Raspberry Pi. You will be prompted once for the password and
     the server will run until the end of the session. This starts both the CN0575 temperature
     monitor and the ADXL355 vibration monitor.
   - **Start** / **Stop** - begin and end acquisition
   - **Axis health** - shows the current status of the three axes (``OK``, ``WARNING``,
     ``ALARM``) and live RMS for X, Y, Z.

#. **Color sensor - Sorting line**

   Reads the TCS34725 data over IP and sorts objects by color. The demo shows a conveyor belt with three colored objects (red, green, blue)
   that are sorted into bins based on the detected color.
   Before starting the sorting, ensure calibration is complete:

   #. **Calibrate White** - place a white reference under the sensor and click the button.
      The status reads *Calibrated* when done. **Reset Cal** undoes it.
   #. **Cal Red**, **Cal Green**, **Cal Blue** - place the matching cube under the sensor
      and click. The label tracks which cubes are calibrated.

   **Sorting**

   **Start sorting** arms detection; the label changes to *Sorting running* and
   the button becomes **Stop Sorting**.

   A reading is only treated as a cube when a channel exceeds 500 counts. Each cube is sorted into the bin with the highest color match.
   The log shows the detected color and the bin it was sorted into.

#. **Servo control**

   Direct manual servo control, independent of the sorting line.

#. **CN0575 - Temperature monitor**

   - **Read Temp** - single ADT75 reading, shown in °C
   - **Live (5s)** - poll every 5 seconds and plot the temperature over time
   - **Clear** - clear the graph

#. **AD-SWIOT1L-SL - Fan control**

   - **Connect** - open the connection (requires ``pyadi-iio``).
   - **Duty (%)** - enter a duty cycle and press **Set**; **Stop** halts the fan.
   - The readout shows duty cycle, estimated RPM, and ``RUNNING`` / ``STOPPED``.

   The fan animation indicates only *that* the fan is commanded on - it does not spin
   proportionally to the duty cycle.

#. **Communication log**

   Every command, response, error and detection event is timestamped and tagged with
   the board name. This is the first place to check for errors.


Control Panel running
-------------------------------------------------------------------------------

.. figure:: running_app.png
   :align: center
   :width: 900