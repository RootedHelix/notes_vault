
**You do _not_ need external software!**  
The **Kali Linux installer provides everything you need** for dual booting with Windows (or another OS).

---

### **How Dual Booting with Kali Works (2025):**

#### **1. Prepare Your Computer**

- **If you have Windows installed:**
    
    - Use **Windows Disk Management** or **GParted** (on a live USB) to shrink your Windows partition and create unallocated space for Kali.
        
- **If you’re starting from scratch:**
    
    - You can just use the Kali installer to create all partitions.
        

#### **2. Boot from the Kali Linux Installer USB/DVD**

- Download the [Kali Linux ISO](https://www.kali.org/get-kali/), write it to a USB with [Rufus](https://rufus.ie/) (Windows) or `dd` (Linux/Mac), and boot from it.
    

#### **3. Use the Kali Installer**

- Choose “**Graphical install**” or “**Install**” in the boot menu.
    
- When you reach the partitioning section, select “**Guided – use the largest continuous free space**” (recommended for beginners) or manually partition if you prefer.
    
- The installer will detect Windows (if present) and set up **GRUB** automatically, so you can pick which OS to boot each time.
    

#### **4. Reboot**

- After installation, you’ll see a **boot menu** (GRUB) at startup. Choose Windows or Kali Linux.
    

---

## **You Do Not Need:**

- Any external bootloader (like EasyBCD, rEFInd, etc.)—unless you want advanced customization.
    
- Any paid software.
    
- Any extra partition software, unless you want to prepare partitions in advance (GParted is free and optional).
    

---

## **You _Do_ Need:**

- A backup of important data (just in case).
    
- Some free/unallocated space on your drive.
    
- The official Kali installer (which includes GRUB and partitioning tools).
    

---

### **TL;DR**

- **The Kali Linux installer provides everything you need.**
    
- Just follow the prompts; it will handle dual boot setup for you.
    
- No external/third-party dual boot software required.
    

---

If you want a **step-by-step guide** for your exact setup (Windows 10/11 + Kali, UEFI/Legacy BIOS, etc.), let me know your system details and I’ll write out the exact steps for you!

## Partitioning Drives

Perfect! Here’s a **step-by-step guide to partitioning your drive for dual booting Kali Linux with Windows 10**, with best practices and tips:

---

## **Step 1: Back Up Your Important Data**

- Before making any changes, back up documents, photos, and any files you can’t afford to lose (just in case).
    

---

## **Step 2: Shrink Your Windows 10 Partition**

1. **Open Disk Management:**
    
    - Press `Win + X` and choose **Disk Management** (or press `Win + R`, type `diskmgmt.msc`, hit Enter).
        
2. **Locate your Windows drive** (usually labeled C:).
    
    - Check how much free space you have available.
        
3. **Right-click the C: drive** and choose **Shrink Volume**.
    
4. **Enter the amount to shrink:**
    
    - Decide how much space to allocate for Kali Linux (e.g., 50 GB = 50,000 MB).
        
    - Enter this number and click **Shrink**.
        
5. **You’ll now see “Unallocated Space”** (black bar) equal to the size you chose.
    
    - **Important:** Do **not** format or assign a letter to this space!
        
    - Leave it “unallocated” for Kali Linux to use.
        

---

## **Step 3: (Optional) Defragment Your Drive**

- Not strictly necessary on SSDs, but if you’re on a hard disk, you can use Windows’ **Defragment and Optimize Drives** tool to consolidate free space before shrinking.
    

---

## **Step 4: Prepare Your Kali Linux Installation Media**

- Download the latest Kali ISO from [the official site](https://www.kali.org/get-kali/).
    
- Use [Rufus](https://rufus.ie/) to write the ISO to a USB drive.
    
- Plug in the USB, reboot, and enter your BIOS/UEFI (usually by pressing F2, Del, Esc, or a function key at boot).
    
- Set the USB to boot first.
    

---

## **Step 5: Install Kali Linux**

1. **Boot into the Kali Linux installer.**
    
2. Choose **Graphical Install** or **Install**.
    
3. Proceed until you reach the **Partitioning** step.
    
4. When asked about partitioning, choose:
    
    - **Guided – Use the largest continuous free space** _(recommended for beginners)_.
        
        - This will auto-create the needed partitions in the unallocated space.
            
    - Or **Manual** (“Something else”) if you want to create `/` (root), `/home`, and `swap` yourself.
        
5. Finish the installation.
    
    - The installer will install GRUB and detect both Windows and Kali, letting you pick at boot.
        

---

## **Step 6: First Reboot**

- When you reboot, you’ll see the **GRUB boot menu**.
    
- Choose either **Kali Linux** or **Windows Boot Manager**.
    

---

### **Quick Visual Reference**

```
Before:
[ Windows C:  |  Free space not available ]

After shrinking in Disk Management:
[ Windows C:  |  Unallocated Space ]

After Kali installation:
[ Windows C:  |  Kali /  |  Kali swap ]
```

---

## **Summary**

1. Back up your data.
    
2. Shrink Windows C: in Disk Management, leave new space “unallocated.”
    
3. Boot Kali Linux installer, install into the unallocated space.
    
4. GRUB bootloader lets you choose OS at boot.
    

---

If you want more detail, screenshots, or have a specific disk setup, let me know! If you hit any error or warning, ask before clicking through—partitioning is safe if you take it slow.
## 