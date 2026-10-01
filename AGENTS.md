# Workspace context: Alpha Data ADM-PCIE-9V3

The user is using an **Alpha Data ADM-PCIE-9V3** FPGA card to load and run Corundum. Treat this as the default hardware context for board-specific work in this repository. Ask which design variant or Ethernet speed is intended when it affects a change.

## Authoritative board references

- [Alpha Data product page](https://alpha-data.com/product/adm-pcie-9v3/) (links the current datasheet and manual).
- [ADM-PCIE-9V3 User Manual, revision 2.10, 17 July 2023](https://www.alpha-data.com/xml/user_manuals/adm-pcie-9v3%20user%20manual_v2_10.pdf), especially sections 3.1 (switches), 3.2 (clocks), 3.3 (PCIe), 3.5 (QSFP28), and 3.8–3.9 (USB/JTAG and configuration). Its appendix contains the complete pinout. Check the product page for a newer revision before relying on details that may change.
- [ADM-PCIE-9V3 datasheet, revision 3.3](https://alpha-data.com/xml/product_datasheets/adm-pcie-9v3_v3.3.pdf).
- Repository-specific support is indexed in `docs/source/devicelist.rst`, with board constraints in `fpga/mqnic/ADM_PCIE_9V3/fpga_25g/fpga.xdc` and `fpga/mqnic/ADM_PCIE_9V3/fpga_100g/fpga.xdc`. Use those XDC files alongside the vendor pinout, rather than guessing pins.

## Hardware facts relevant to Corundum

- Virtex UltraScale+ VU3P in FFVC1517 package; PCIe Gen3 x16 card; two front-panel QSFP28 cages. Each cage can support one 100GbE link or four 25GbE lanes, subject to the loaded design and optics/cable. Standard memory is two 8 GiB, 72-bit DDR4-2400 ECC banks (16 GiB total). The board also has an 8-lane SlimSAS connector.
- **Check the FPGA temperature grade on the physical card.** The vendor manual v2.10 specifies `XCVU3P-2FFVC1517E` as standard. Checked-in Corundum board READMEs, device list, and Makefiles specify `xcvu3p-ffvc1517-2-i`. In this workspace, the 25G `fpga/Makefile` has a pre-existing local change to `xcvu3p-ffvc1517-2-e`. Do not silently overwrite that change or assume the I and E parts are interchangeable for a specific board.
- The QSFP reference clock is programmable through the board's Si5338 clock generator; the manual lists **161.1328125 MHz as the factory default**, not a guaranteed current setting. Confirm the clock configuration when debugging transceiver links or changing PHY IP. PCIe uses a 100 MHz host reference clock.
- The manual specifies SW1-3 OFF for normal operation and SW1-4 OFF to route JTAG from the USB interface (ON selects the debug header). The front-panel micro-USB port provides Digilent USB-JTAG; a rear-edge USB port is present on board revision 7 / serial 306 and newer. Vivado Hardware Manager can configure the FPGA through JTAG. Power-on configuration comes from two 256-Mbit QSPI flash devices operated as SPI x8.
- For flash images, the manual's section 3.9.1.1 specifies a 64 MiB SPIx8 MCS image, positions `0x0000000` and optionally `0x2000000`, and Vivado flash part `mt25qu256-spi-x1_x2_x4_x8`. The board Makefiles include corresponding MCS and flash targets. Treat flash programming as a separate, persistent operation; inspect the exact image layout and target card before running it.
- Cooling matters for large designs. The manual says the system monitor clears FPGA configuration if core temperature exceeds 100 °C. Check airflow, power, and the board's status LEDs when debugging unexplained loss of configuration.

## Corundum paths and workflow

- Main network designs: `fpga/mqnic/ADM_PCIE_9V3/fpga_25g/` and `fpga/mqnic/ADM_PCIE_9V3/fpga_100g/`. Each contains an `fpga/` build directory, RTL, IP scripts, constraints, and a README. Additional 10G/TDMA variants live under these directories. The 25G README describes a 25G BASE-R PHY; the 100G README describes a Xilinx 100G CMAC.
- For the chosen design, the README's basic flow is: run `make` in its `fpga/` directory with Vivado on `PATH`; build the host driver in `modules/mqnic` and userspace utilities in `utils`; use `make program` for temporary JTAG programming. The board README then calls for a host reboot to re-enumerate PCIe, loading `mqnic.ko`, checking `dmesg`, and using `mqnic-dump -d /dev/mqnic0`.
- Repository documentation lists ADM-PCIE-9V3 board ID `0x41449003` and 25G/100G design variants in `docs/source/devicelist.rst`.
- Existing local Vivado outputs and untracked files under `fpga/mqnic/ADM_PCIE_9V3/fpga_25g/fpga/` may be the user's active work. Inspect `git status` before edits and preserve those files unless the user explicitly asks to clean them.
