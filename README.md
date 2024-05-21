# PCI-Passthrough
PCI passthrough allows you to use a physical PCI device (graphics card, network card) inside a VM (KVM virtualization only).

If you "PCI passthrough" a device, the device is not available to the host anymore. Note that VMs with passed-through devices cannot be migrated.


# Let's start step by step:


## 1- In your device BIOS make sure the following is enabled in the BIOS: Intel VT-d & VT-x – Intel Compatible list All AMD CPUs.


## 2-Enable IOMMU in GRUB (check Intel or AMD commands below - ** choose the right one) 

** For intel CPU command line: 
```bash
nano /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
```
** Or for AMD CPU command line:
```bash
nano /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt"
```

## 3-Verify that IOMMU is enabled by running and looking for a line indicating it is enabled
command line:
```bash
dmesg | grep -e DMAR -e IOMMU
```
## 4-run the command "update-grub" and reboot.
command line:
```bash
update-grub
reboot
```
## 5-Enable VFIO Modules add the following modules:
command line:
```bash
nano /etc/modules
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```
## 6- run the command update-initramfs -u -k all and reboot
command line:
```bash
update-initramfs -u -k all
reboot
```
## 7-Verify the modules are enabled with dmesg | grep -i vfio and checking the driver version line is present
command line:
```bash
dmesg | grep -i vfio
```
### 8-**FOR GPU Passthrough ONLY** Isolation From the Host (amend the below to include the IDs of the device you want to isolate)
Get device IDs command line:
```bash
lspci -nn
```
Example:
echo "options vfio-pci ids=????:????,????:???? disable_vga=1" > /etc/modprobe.d/vfio.conf
## 9-**FOR GPU Passthrough ONLY** Blacklist GPU drivers (here are all that you would ever need)
command line:
```bash
echo "blacklist radeon" >> /etc/modprobe.d/blacklist.conf 
echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf 
echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf 
echo "blacklist nvidiafb" >> /etc/modprobe.d/blacklist.conf
echo "blacklist nvidia_drm" >> /etc/modprobe.d/blacklist.conf 
```
## 10-Reboot your machine, be sure to use the web gui because with the gpu passed through you will no longer get video out. The start up will begin but then appear to hang even though proxmox has started fine.
