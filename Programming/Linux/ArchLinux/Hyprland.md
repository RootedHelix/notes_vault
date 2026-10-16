Related to [[Arch Packages]], [[ArchLinux]], [[Linux]]
#hyprland #linux #arch_linux

Hyprland is a window manager and works with the wayland compositor to give a seemless user interface to linux

I recently encountered an issue with linux when I downloaded NVIDIA driver packages that made the os/hyprland detect two monitors and the fix was to explicitly disable the undesired monitor in the `hyprland.conf` file. 

## Hyprland Commands

Check all active monitors: `hyprctl monitors all`