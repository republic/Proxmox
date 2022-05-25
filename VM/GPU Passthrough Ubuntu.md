### Setup Ubuntu VM

Edit grub
```
vim /etc/default/grub
```

Additional Software
```
sudo apt install qemu-guest-agent
```

Change the folowing lines:
```
GRUB_CMDLINE_LINUX_DEFAULT=""
#to
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
#or
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on"
```

Apply changes
```
update-grub
#or
pve-efiboot-tool refresh
```

Edit modules
```
cat << EOF >> /etc/modules
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
EOF
```

Add to blacklist
```
cat << EOF >> /etc/modprobe.d/blacklist.conf
blacklist radeon
blacklist nouveau
blacklist nvidia
EOF
```

Disable GPU from host
```
lspci -nn | grep -e 'VGA.*NVIDIA' -e 'Audio.*NVIDIA'
```
Insert vender IDs for the GPU and update
```
echo "options vfio-pci ids=****:****,****:*** disable_vga=1"> /etc/modprobe.d/vfio.conf
update-initramfs -u
```

### Create VM (Ubuntu 20.04). Install and configure driver for nvidia

Disable nouveau driver
```
sudo bash -c "echo blacklist nouveau > /etc/modprobe.d/blacklist-nvidia-nouveau.conf"
sudo bash -c "echo options nouveau modset=0 >> /etc/modprobe.d/blacklist-nvidia-nouveau.conf"
sudo update-initramfs -u
```

Install nvidia driver
```
sudo apt update
sudo apt install build-essential libglvnd-dev pkg-config
wget https://us.download.nvidia.com/XFree86/Linux-x86_64/455.23.04/NVIDIA-Linux-x86_64-455.23.04.run
chmod +x NVIDIA-Linux-x86_64-455.23.04.run
sudo ./NVIDIA-Linux-x86_64-455.23.04.run
```

Edit ProxMox config file
```
vim /etc/pve/qemu-server/100.conf
cpu: host,hidden=1
```

Test VM is nvidia driver is working
```
nvidia-smi
```
