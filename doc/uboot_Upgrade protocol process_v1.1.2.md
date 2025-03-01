# U-Boot Upgrade Protocol

---

| Version Number | Remarks     |
| --------------- | ----------- |
| v1.1.2         | Initial Version |

## I. Implementation Process
1. Main Flowchart

<br/>
<div align="center">
    <img src="./attch/uboot_update_0.jpg">
</div>
<br/>

2. U-Boot Upgrade Flowchart

<br/>
<div align="center">
    <img src="./attch/uboot_update_1.jpg">
</div>
<br/>

## II. Protocol Description

## 1. Packet Format

Multi-byte data: low byte first, high byte last, i.e. little-endian format.  
CRC16 Standard: CRC-CCITT (XModem).  
Responses use the same command.

| Byte[0]         | Byte[1]         | Byte[3~2] | Byte[4] | Byte[5]    | Byte[6~n-2]       | Byte[n~n-1] |      |
| --------------- | --------------- | --------- | ------- | ---------- | ----------------- | ----------- | ---- |
| syncdata0(0xAA) | syncdata1(0x55) | cmd_len   | cmd     | rsp_status | param (some commands have none) | crc16       |      |

Parameter Description:  
syncdata0:  Fixed to 0xAA  
syncdata1:  Fixed to 0x55  
cmd_len:  Length of cmd + rsp_status + union data  
cmd:  Operation command  
rsp_status:  Response status  
param:   Parameters corresponding to the operation command (some commands have no parameters)  
crc16:  CRC16 result of the entire command packet (excluding the CRC itself)

## 2. Response Status

| Status                   | Value | Description       |
| ------------------------ | ----- | ----------------- |
| JL_SU_CMD_SUSS          | 0x0   | Command executed successfully |
| JL_SU_CMD_CRC_ERROR     | 0x1   | CRC error         |
| JL_SU_CMD_SDK_ID_ERROR  | 0x2   | ID error          |
| JL_SU_CMD_OTHER_ERROR   | 0x3   | Other errors      |

        enum {  
         JL_SU_CMD_SUSS, // Command success  
         JL_SU_CMD_CRC_ERROR, // CRC error  
         JL_SU_CMD_SDK_ID_ERROR, // ID error  
         JL_SU_CMD_OTHER_ERROR, // Other errors  
        };

## 3. Operation Commands, Parameters, and Data Direction

### 1. JL_SU_CMD_DEVICE_INIT

| Defined Value | Description                                    |
| --------------| ---------------------------------------------- |
| ------        | -                                              |