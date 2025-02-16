# GPU Passthrough (Ubuntu)

### Operating System & Software
- Proxmox VM
- Ubuntu 22.04
- Nvidia Drivers

---

### Edit Grub On Proxmox Server
```bash
nano /etc/default/grub
```
Intel
```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
```
AMD
```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on"
```

### Run After Editing Grub
```bash
update-grub
```

### Edit Modules
```bash
cat << EOF >> /etc/modules
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
EOF
```

### Add To Blacklist
```bash
cat << EOF >> /etc/modprobe.d/blacklist.conf
blacklist radeon
blacklist nouveau
blacklist nvidia
EOF
```

### Disable GPU From Host
Check vendor GPU ID(s) of your vga card.
```bash
lspci -nn | grep -e 'VGA.*NVIDIA' -e 'Audio.*NVIDIA' | sed 's/.*\[\([^]]*\)\].*/\1/g'
```
Create vender GPU IDs file.
```bash
lspci -nn | grep -e 'VGA.*NVIDIA' -e 'Audio.*NVIDIA' | sed 's/.*\[\([^]]*\)\].*/\1/g' | xargs -n 2 bash -c 'echo "options vfio-pci ids=$0,$1 disable_vga=1"> /etc/modprobe.d/vfio.conf'
```
Check output of file ```/etc/modprobe.d/vfio.conf```.
```bash
cat /etc/modprobe.d/vfio.conf
```
Reboot system.

---

## Create VM (Ubuntu). Install and configure driver for nvidia

### Install Nvidia Driver
```bash
ubuntu-drivers devices
sudo ubuntu-drivers autoinstall
```
or
```bash
ubuntu-drivers devices
sudo apt install nvidia-driver-515
sudo reboot
```
or
```bash
sudo apt install software-properties-common -y
sudo add-apt-repository ppa:graphics-drivers/ppa -y
sudo apt update
ubuntu-drivers devices
sudo ubuntu-drivers autoinstall
```

### Test Nvidia Driver
```bash
nvidia-smi
```

### Additional Software
```bash
sudo apt install qemu-guest-agent
```

## Troubleshooting

### Disable Nouveau Driver
```bash
sudo bash -c "echo blacklist nouveau > /etc/modprobe.d/blacklist-nvidia-nouveau.conf"
sudo bash -c "echo options nouveau modset=0 >> /etc/modprobe.d/blacklist-nvidia-nouveau.conf"
sudo update-initramfs -u
```
### Edit Proxmox Config File
```bash
vim /etc/pve/qemu-server/100.conf
cpu: host,hidden=1
```
