# OGC.Engineering
#### Yocto development on the STM32MP157F-DK2
developmer contact - dustin ( at ) ogc.engineering

---

## Repository Usage
```
git clone https://github.com/OGC-dustin/ogc_yocto_exploration__stm32mp1.git
git submodule update --init
```

## Description
* ST STM32MP157D-DK2 - Discovery kit with STM32MP157F MPU
    * https://www.st.com/en/evaluation-tools/stm32mp157f-dk2.html#overview
```
    Common features:
        STM32MP157 Arm®-based dual Cortex®‑A7 800 MHz 32 bits + Cortex®‑M4 32 bits MPU in a TFBGA361 package
        ST PMIC STPMIC1
        4-Gbit DDR3L, 16 bits, 533 MHz
        1-Gbit/s Ethernet (RGMII) compliant with IEEE-802.3ab
        USB OTG HS
        Audio codec
        4 user LEDs
        2 user and reset push-buttons, 1 wake-up button
        5 V / 3 A USB Type-C® power supply input (not provided)
        Board connectors:
            Ethernet RJ45
            4 × USB Host Type-A
            USB Type-C® DRP
            MIPI DSI®
            HDMI®
            Stereo headset jack including analog microphone input
            microSD™ card
            GPIO expansion connector (Raspberry Pi® shield capability)
            ARDUINO® Uno V3 expansion connectors
        On-board ST-LINK/V2-1 debugger/programmer with USB re-enumeration capability: Virtual COM port and debug port
        STM32CubeMP1 and full mainline open-source Linux® STM32 MPU OpenSTLinux Distribution (such as STM32MP1Starter) software and examples
        Support of a wide choice of Integrated Development Environments (IDEs) including IAR Embedded Workbench®, MDK-ARM, and STM32CubeIDE
    Board-specific features:
        4" TFT 480×800 pixels with LED backlight, MIPI DSI® interface, and capacitive touch panel
        Wi‑Fi® 802.11b/g/n
        Bluetooth® Low Energy 4.1
```

---

## Build one of the core images as proof of concept
* Create folders and get repositories ( already done if repository cloned and submodules initialized/updated )
```
mkdir sources
cd sources
git clone -b scarthgap git://git.yoctoproject.org/poky.git
git clone -b scarthgap git://git.openembedded.org/meta-openembedded
git clone -b scarthgap https://github.com/STMicroelectronics/meta-st-stm32mp.git
cd ..
```
* Add layers to the build
```
. sources/poky/oe-init-build-env
bitbake-layers add-layer ../sources/meta-openembedded/meta-oe
bitbake-layers add-layer ../sources/meta-openembedded/meta-python
bitbake-layers add-layer ../sources/meta-st-stm32mp
echo "MACHINE ?= \"stm32mp1\"" >> conf/local.conf
```

## Build a core image ( skip sourcing of environment if above steps manually executed )
```
. sources/poky/oe-init-build-env
time bitbake core-image-full-cmdline --runall=fetch
time bitbake core-image-full-cmdline
```

## Create SD card and boot development kit
* SD Card image creation scripts exist in generated files at tmp/deploy/images/stm32mp1/scripts
* Flash layout examples exist in generated files at tmp/deploy/images/stm32mp1/flashlayout_<image_name>
```
cd tmp/deploy/images/stm32mp1/scripts
./create_sdcard_from_flashlayout.sh ../flashlayout_core-image-full-cmdline/extensible/FlashLayout_sdcard_stm32mp157f-dk2-extensible.tsv 
```
* Example writing an SD Card at /dev/mmcblk0
```
sudo dd if=../FlashLayout_sdcard_stm32mp157f-dk2-extensible.raw of=/dev/mmcblk0 bs=8M conv=fdatasync status=progress oflag=direct
sync
```
* Insert SD Card into development kit, connect micro USB to CN11 ST-LINK and use your favorite terminal to connect
```
minicom -b 115200 -D /dev/ttyACM0
CTRL-a q to quit, CTRL-a z to get menu
```

## Notes found during core image build
* No HDMI output, boot messages seem to indicate no /dev/fb reference used by command line ot x-windows in core-image-sato build
