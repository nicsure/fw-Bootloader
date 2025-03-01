# uboot upgrade agreement

---

| version number | Remark     |
| ------ | -------- |
| v1.1.2 | initial version |

## 一、uboot Project selection
According to the chip model used, select the corresponding uboot project. by AC632N For example, the project that needs to be opened is fw-Bootloader-main\user_boot\cpu\bd19\bd19_uboot.cbp。
As shown in the picture：

<br/>
<div align="center">
    <img src="./attch\uboot_select_project.png">
</div>
<br/>

## 
2. Upgrade mode selection
There are two types of uboot upgrade: "serial port upgrade mode" and "USB_HID upgrade mode". Choose according to your needs.
### 1.Serial port upgrade mode configuration
Open the Project build options settings, select Compiler settings, #defines settings, and add USB_MODE=0 to configure the uboot project into serial port upgrade mode. As shown in the picture:

<br/>
<div align="center">
    <img src="./attch\serial_update_mode_config.png">
</div>
<br/>


app\src\user.c file, ut_device_mode(tx, rx, bud) function sets the serial port TX pin, RX pin, and baud rate. As shown in the picture:

<br/>
<div align="center">
    <img src="./attch\serial_config.png">
</div>
<br/>

### 2.USB_HID Upgrade mode configuration
Open the Project build options setting, select Compiler settings, #defines settings, and add USB_MODE=1 to configure the uboot project into USB_HID upgrade mode. As shown in the picture:

<br/>
<div align="center">
    <img src="./attch\usb_hid_update_mode_config.png">
</div>
<br/>

When selecting the usb_hid upgrade mode, the usb_vid and usb_pid of the uboot project need to be consistent with the usb_vid and usb_pid of the usb_hid host computer.

The uboot project modifies usb_vid and usb_pid methods as shown in the figure:
<br/>
<div align="center">
    <img src="./attch\uboot_usb_vid_pid.png">
</div>
<br/>

usb_hid The method of modifying usb_vid and usb_pid on the host computer is as shown in the figure: (Open pc_demo\usb_hid\main.cpp to view)
<br/>
<div align="center">
    <img src="./attch\usb_hid_usb_vid_pid.png">
</div>
<br/>

## 3. Debugging function configuration
### 1. Enable debug printing
Open the Project build options settings, select Compiler settings, #defines settings, and add __DEBUG to enable the debug printing function. As shown in the picture:

<br/>
<div align="center">
    <img src="./attch\debug_enable.png">
</div>
<br/>

### 2. Debug printouts and baud rate configuration
Method 1: Use the pin and baud rate configuration in the isd_config.ini configuration file. As shown in the picture:

<br/>
<div align="center">
    <img src="./attch\isd_ini_config.png">
</div>
<br/>

Call the uart_init(uttx, ut_buad) function in the main function, as shown in the figure:

<br/>
<div align="center">
    <img src="./attch\isd_ini_config_code.png">
</div>
<br/>

Method 2: Set the print pin and baud rate directly in the code. Take PA5, 1000000 baud rate as an example. As shown in the picture:

<br/>
<div align="center">
    <img src="./attch\set_tx_and_baud.png">
</div>
<br/>

## 4. Upgrade triggering mode configuration
The upgrade trigger methods include I/O port detection trigger and SDK software reset trigger.
### 1. I/O oral trigger
Enter uboot, main function, and choose whether to jump to the upgrade process by detecting the level status of an I/O. As shown in the picture:

<br/>
<div align="center">
    <img src="./attch\io_detect.png">
</div>
<br/>

### 2.SDK Software reset trigger
In the user.h file, enable the USE_UPGRADE_MAGIC macro. As shown in the picture:

<br/>
<div align="center">
    <img src="./attch\sdk_softreset.png">
</div>
<br/>

In the sdk project, add the following code to implement SDK software reset trigger. (Anywhere)

    extern u32 nvram_list[];
    #define NV_RAM_LIST_ADDR nvram_list
    static u8 uboot_uart_upgrade_mode_magic[8] = {'u', 'b', 'o', 'o', 't', 0x5a, 's', 't', };
    static u8 uboot_uart_upgrade_succ_magic[8] = {'u', 'b', 'o', 'o', 't', 0xa5, 'o', 'k', };
    void check_uboot_uart_upgrade() //To check whether uboot upgrade is successful, this flag must be detected before memory_init();
    {
        if (memcmp((char *)NV_RAM_LIST_ADDR, uboot_uart_upgrade_succ_magic, sizeof(uboot_uart_upgrade_succ_magic)) == 0) {
            memset((char *)NV_RAM_LIST_ADDR, 0, sizeof(uboot_uart_upgrade_succ_magic));
            log_info("uboot uart upgrade succ\n");
        }
    }
    void hw_mmu_disable(void);
    void chip_reboot_entry_uboot_uart_upgrade_mode()    // uboot upgrade jump function
    {
        memcpy((char *)NV_RAM_LIST_ADDR, uboot_uart_upgrade_mode_magic, sizeof(uboot_uart_upgrade_mode_magic));
        hw_mmu_disable();
        cpu_reset();
    }

In the maskron_stubs.ld file, add the following code, the path is fw-Bootloader-main\user_boot\cpu\bd19\output. As shown in the picture:

    nvram_list = ABSOLUTE(0x800);

<br/>
<div align="center">
    <img src="./attch\maskrom_stubs.ld.png">
</div>
<br/>

Different chip models correspond to different parameters:

    BD19: nvram_list = ABSOLUTE(0x800);
    BR23: nvram_list = ABSOLUTE(0x10800);
    BR25: nvram_list = ABSOLUTE(0x10880);
    BR28: nvram_list = ABSOLUTE(0x180800);
    BR30: nvram_list = ABSOLUTE(0x28800);
    BR34: nvram_list = ABSOLUTE(0x28800);
    SH54: nvram_list = ABSOLUTE(0x7ee4);
    SH55: nvram_list = ABSOLUTE(0x4ee4);
    

Where uboot upgrade needs to be performed, just call the chip_reboot_entry_uboot_uart_upgrade_mode() function.

After the upgrade is completed, the sdk calls the check_uboot_uart_upgrade() function to check whether the upgrade is successful (it needs to be placed before memory_init()).

## 5. Use of host computer
There are three types of host computer tools: win-uart, win-usb_hid, android-usb_hid, which are placed under the fw-Bootloader\update_tools\tools path and are open source.

### 1.The serial port upgrade host computer interface description is as follows:

    1.Select the corresponding serial port
    2.Set the baud rate
    3.Upgrade uboot (usually this option is not checked)
    4.Communication encryption key (uboot default key is 12345678 (decimal), the key can be modified in the user.c file, communication_key variable)
    5.Refresh the serial port
    6.Upgrade file selection
    7.Start upgrading
    Note: When selecting serial port upgrade, the packet length is customized, and the default length is 4K.

<br/>
<div align="center">
    <img src="./attch\serial_upper_host_machine.png">
</div>
<br/>

### 2.win-USB_HID The instructions for upgrading the host computer are as follows: (There is no graphical interface at the moment)

    1. Open the fw-Bootloader-main\update_tools\tools\win-usb_hid\build-out-bin folder;
    2. Copy the generated jl_isd.bin file to the folder;
    3. Open the Powershell window;
    4. Enter .\UbootHid.exe and press Enter to execute;
    5. After the upgrade is completed, reset;

<br/>
<div align="center">
    <img src="./attch\usb_hid_build_out_bin.png">
</div>
<br/>

<br/>
<div align="center">
    <img src="./attch\open_powershell_window.png">
</div>
<br/>

<br/>
<div align="center">
    <img src="./attch\UbootHid_exe.png">
</div>
<br/>

<br/>
<div align="center">
    <img src="./attch\update_ok.png">
</div>
<br/>

### 3.win-USB_HID The instructions for upgrading the host computer interface are as follows:：
    1. After connecting the small machine to the mobile phone, open the APP and display Device: online to indicate successful connection;
    2. Click Select file and select the jl_isd.bin file to be upgraded;
    3. Click Upgrade and wait until the upgrade is completed. Success will be prompted;

<br/>
<div align="center">
    <img src="./attch\android_usb_hid.png">
</div>
<br/>
<br/>
<div align="center">
    <img src="./attch\android_usb_hid_update_success.png">
</div>
<br/>

## 6、Test process
    1. Build the uboot project and generate a new uboot.boot file with the path cpu\bd19\output;
    2. Copy the uboot.boot file to the sdk download directory, that is, the \cpu\bd19\tools folder;
    3. Compile and download the sdk to the minicomputer. At this time, the jl_isd.bin file is generated as program A. Back up program A first;
    4. Modify the sdk (such as modifying some printing), then compile and download it to the minicomputer. At this time, the jl_isd.bin file is generated as program B;
    5. Through the above steps, we get program B running on the small computer and program A to be upgraded;
    6. The PC and the minicomputer are connected through the serial port, the minicomputer is powered on, triggers the upgrade and enters the upgrade mode to wait for the upgrade (serial port upgrade or USB_HID upgrade);
    7. Set the corresponding parameters (com port, baud rate, secret key, upgrade file) on the PC host computer;
    8. Click Start Update to start the upgrade;

## 7. Precautions
    1. The uboot.boot used by the program running on the minicomputer and the uboot.boot used by the file to be upgraded need to be consistent;
    2. It is recommended to force the program to 4K alignment and add the following code to the isd_config.ini file;
    SPECIAL_OPT=0; 
    FORCE_4K_ALIGN=YES;
    3. If 4k alignment cannot be used (the code space is not enough), please ensure that the bin file used for upgrade is generated when using the forced upgrade tool to connect to the prototype to download the code;
    4. If the isd_config.ini file has EOFFSET=1; configuration, you need to add GENERATE_TWO_BIN = YES in the isd_config.ini file to generate 0K/4K files;
        Then according to upgrade_eoffset (in the uboot code), if it is equal to 4k, use jl_isd_4K.bin, otherwise use jl_isd_0K.bin. If there is no EOFFSET=1; just use the jl_isd.bin file directly;
    5. When choosing usb_hid to upgrade, since the maximum length of the hid transmission packet is 64 Byte, the command to write flash will also occupy some Bytes, so the actual data length written to flash = (64 - the length of the write flash command); as shown in the figure: (Open pc_demo\usb_hid\main.cpp to view)
<br/>
<div align="center">
    <img src="./attch\write_flash_max_len.png">
</div>
<br/>



