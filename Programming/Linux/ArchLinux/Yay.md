Related to [[Linux]], [[Arch Packages]]

#linux #yay_package #arch_packages 

# Summary
**`yay`** stands for **Yet Another Yogurt**. It is an **AUR helper** used on **Arch Linux** to easily install and manage packages from the **AUR (Arch User Repository)**.

Think of it like:

| Tool     | Purpose                                             |
| -------- | --------------------------------------------------- |
| `pacman` | Official Arch package manager (official repos only) |
| `yay`    | Wrapper that handles **both pacman + AUR packages** |

---

## Why `yay` exists

The **AUR** contains thousands of community-maintained packages (things not in official Arch repos), like:

- `1password`
    
- `jetbrains-toolbox`
    
- `google-chrome`
    
- `visual-studio-code-bin`
    
- many developer tools
    

Without `yay`, installing an AUR package requires several manual steps.


# Important Notes

- Yay will ask 
- 
- What the options mean

**[N]one (recommended most of the time)**

- Reuse the existing build directory
    
- Faster build
    
- Safe for normal updates
    

**[A]ll**

- Apply the operation to all listed packages

**[I]nstalled**

- Clean build files for packages you currently have installed
    

**[No]tInstalled**

- Clean build files for packages not installed
    

**[Ab]ort**

- Cancel the process

---
- A is the standard choice
--- 
Then when asked diffs to show: When `yay` asks about **showing diffs**, it’s asking if you want to see **changes in the PKGBUILD or install scripts** compared to the last time you built the package.

Example prompt you might see:

:: Diff between old and new PKGBUILDs?  
[Y]es [N]o [A]bort
# Commands
#yay_commands

```bash 

```