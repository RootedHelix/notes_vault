Related to [[Embedded]], [[STM32 Week1]], [[CLI]]
#embedded_workflow #embedded #CLI

We create a central hub for flashing and building on the secondary Computer (2019 Build) using WSL. This way I am able to program externally and then centralize the build process on a physical environment elsewhere that better supports the tools and software necessary to develop for the MCU. 

> See [[WSL Embedded Logs]] for more information on how to use picocom to view embedded logs 

# How to Setup



## Necessary Packages

**Windows** : `usbipd`

**WSL/Ubuntu** : `make`, `openocd`, `stlink-tools 
- `sudo apt install gdb-multiarch`



Since you’re already comfortable with Linux tooling, WSL gives you:

- Real `make`    
- Real GCC
- Real package manager
- Clean CLI workflow
- No Windows toolchain headaches

Let’s design this correctly.

---

## 🧠 What Changes With WSL?

Your new architecture becomes:
#### PC B (Windows + WSL)

- Windows handles:
    - USB connection to STM32
- WSL handles:
    - Git
        
    - Build system
        
    - ARM GCC
        
    - Make
        
    - Flashing tool (via USB pass-through)
        

So yes — you can install almost everything inside WSL.

But there is one important detail about USB.

---

## ⚠️ Important: USB Access in WSL

WSL2 does **not automatically see USB devices**.

To flash STM32 from WSL, you need:

🔹 usbipd-win

This allows USB devices (like ST-Link) to attach to WSL.

Without this, WSL cannot talk to your board.

---

## 🧱 Clean Setup Plan (Recommended)

### Step 1 — Install Everything Inside WSL

Inside Ubuntu (WSL):

```bash
sudo apt update
sudo apt install build-essential git make
```

Then install ARM toolchain:

```bash
sudo apt install gcc-arm-none-eabi
```

Verify:

```bash
arm-none-eabi-gcc --version
```

---

### Step 2 — Install Flashing Tool in WSL

You have two options.

---

### ✅ Option A (Easiest in WSL): OpenOCD

Install:

🔹 OpenOCD

Inside WSL:

```bash
sudo apt install openocd
```

---

### ✅ Option B: STM32CubeProgrammer (Linux version)

Install Linux version of:

🔹 STM32CubeProgrammer

This works fine in WSL once USB is passed through.

---

## 🧩 Step 3 — Attach STM32 USB to WSL

On Windows (PowerShell as Admin):

```powershell
usbipd list
```

Find your ST-Link device.

> ** THIS  BELOW MUST BE DONE EVERY TIME IF USING WSL AND NOT NATIVE LINUX**

Then:

```
usbipd bind --busid X-X
usbipd attach --wsl --busid X-X
```

Replace `1-3` with your actual BUSID.

This tells Windows:

> “This device can be shared with WSL.”

You usually only need to bind once unless unplugged/reset.

Now inside WSL:

```bash
lsusb
```

You should see STMicroelectronics.

Now flashing works.


USB detection ≠ Debugger usability.

Now we check whether user-space tools can access it.

---

## 🔬 Test 1 — Using stlink tools (if installed)

`st-info --probe`

If working, you’ll see something like:

`Found 1 stlink programmers   serial: ...   flash: 524288 (pagesize: 16384)   sram: 131072`

---

## 🔬 Test 2 — Using OpenOCD

`openocd -f interface/stlink.cfg -f target/stm32f4x.cfg`

If successful, you’ll see:

`Info : STLINK V2J... Info : Target voltage: 3.3V Info : stm32f4x.cpu: hardware has 6 breakpoints...`

That means the debugger is talking to the MCU.

---

### 🧪 Step 4 — Full Workflow in WSL

Inside WSL:

```bash
git clone your_repo
cd project
make
make flash
```

Completely Linux-style.

---

## 🚀 Why WSL Is Better For You

Given your background (NGINX, Linux, memory pools, low-level systems):

WSL gives you:

- Same workflow as real embedded Linux teams
    
- Clean Makefiles
    
- Easy scripting
    
- Easy CI mirroring
    

It also aligns with how embedded firmware teams operate.

---

# After Generating the Code from CubeMX

## 🧭 What You Should Do

Go back to the project root (not inside `cmake/`):
[[HAL Library]]
`cd ~/Projects/embedded_collection/stm32/F446RE_Test`

---

### 🏗 Step 1 — Create a Build Directory

Never build in-source.

`mkdir build cd build`

---

### 🧠 Step 2 — Configure CMake with the ARM Toolchain


Because this is a cross-compile project, you must tell CMake which toolchain file to use.

Run:

>`cmake -DCMAKE_TOOLCHAIN_FILE=cmake/gcc-arm-none-eabi.cmake ..`

If your ARM GCC is installed properly in WSL (`arm-none-eabi-gcc --version` works), this should configure successfully.

---

### 🏗 Step 3 — Build

Now run:

`make`

or

`cmake --build .`

After successful build you should see:

`F446RE_Test.elf F446RE_Test.bin F446RE_Test.hex`

inside `build/`.


### Viable mention:

#### Configure once
cmake -S . -B build -DCMAKE_TOOLCHAIN_FILE=cmake/gcc-arm-none-eabi.cmake

#### Build anytime
cmake --build build

---
## Flash Using OpenOCD
#flash_stm32_command
>From inside the `build/` directory Run this:

```bash
sudo openocd \
-f interface/stlink.cfg \
-f target/stm32f4x.cfg \
-c "program F446RE_Test.elf verify reset exit"

```


When flashing if you see :

`** Programming Finished **`
`** Verified OK **`

That means:
- ELF was written into flash  
- Flash was read back  
- It matched byte-for-byte

> You are now running your new firmware.

---

# This is not what I did. This is an Alternate Option
### Alternative (Even Simpler) But less professional

You can:

- Build in WSL
    
- Flash using Windows STM32CubeProgrammer
    

Example:

```bash
make
```

Then in PowerShell:

```powershell
STM32_Programmer_CLI -c port=SWD -w build/firmware.elf -rst
```

No USB passthrough needed.


---
		
Yes — you can install almost everything through WSL.

The only tricky part is USB access, which requires usbipd.

## Debug
[[Debugging With GDB]]