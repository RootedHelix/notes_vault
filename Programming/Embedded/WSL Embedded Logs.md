Related to [[Embedded Workflow]], [[Embedded]], [[WSL]]
#wsl_embedded_logs #embedded

By default the USB access is owned by windows. You must ensure that access to USB has been shared using `usbipd` similar to the process laid out in [[Embedded Workflow]].

Command to list Device Files, Specifically ones pertaining to USB virtual serial port (commonly MCU's): 

```bash
/dev/ttyACM
```
**More info on this^**
AI RESPONSE
- `/dev/` → directory containing device nodes (interfaces to hardware)
- `tty` → teletype (terminal interface)
- `ACM` → **Abstract Control Model** (USB CDC class for serial communication)
- `0` → first device of this type

So `/dev/ttyACM0` is:
**The first USB virtual serial port exposed by a device (commonly microcontrollers).**

## Confirm These before using picomon

- The 115200 there is the [[Baud Rate]]. The baud rate determines the communication layer that both devices are using 

You must confirm that the Source Code that you have has the baud rate that you set to the `picocom` 

You can find this in the `MX_USART2_UART_Init()` function in the HAL source. The function lives in `main.c`

The command to open the `picocom`(See [[WSL Packages]]) serial window: 

```bash
picocom -b 115200 /dev/ttyACM0
```




