Related to [[Embedded]], [[Embedded Workflow]], [[GDB]]
#embedded #gdb_debug

Next step:

Start OpenOCD **without the program command**:

`sudo openocd \` 
`-f interface/stlink.cfg \` 
`-f target/stm32f4x.cfg`

Leave that terminal running.

Then open a second terminal and run:

`gdb-multiarch F446RE_Test.elf`

Inside GDB:

`target remote localhost:3333`
`monitor reset halt`
`break main` 
`continue`

When it stops at `main`, type:

`print/x RCC->CFGR`

Now you will be reading live hardware.

## Useful Commands 

`info registers`

`info breakpoints`

`delete 'breakpoint number'`

`bt`