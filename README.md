# WinOnLinux
```
//check virtualization
lscpu | grep Virtualization
VT-x = Intel
AMD-v = AMD
```
```
//get dedicated gfx pci ids
lspci -nn | grep -E "NVIDIA"
```
```
pcibus    ids 
1:00.0 = 10de:1f99
1:00.1 = 10de:10fa
```
```
//add iommu and vfio in kernel cmdline (grub or systemd)

//for intel
intel_iommu=on iommu=pt vfio-pci.ids=10de:1f99,10de:10fa
//for amd
amd_iommu=on iommu=pt vfio-pci.ids=10de:1f99,10de:10fa
```
```
//edit or add this file /etc/modprobe.d/vfio.conf
options vfio-pci ids=10de:1f99,10de:10fa
softdep nvidia pre: vfio-pci
blacklist nouveau
options nouveau modeset=0
```

```
get your kernel name with uname -r command
//run below command
sudo mkinitcpio -p linux 

// reboot system
```
```
//create vm
...vmconfigs
...virtio.iso inside windows vm
sudo strings /sys/firmware/acpi/tables/MSDM //get oem key attached to your motherboard
...passing smbios as sysinfo for windows activation inside vm with oem keys
```
```
//use dmidecode to get values and replace all EDIT THIS with dmidecode values
<domain xmlns:qemu="http://libvirt.org/schemas/domain/qemu/1.0" type="kvm">
...
<sysinfo type="smbios">
    <bios>
      <entry name="vendor">EDIT THIS</entry>
    </bios>
    <system>
      <entry name="manufacturer">EDIT THIS</entry>
      <entry name="product">EDIT THIS</entry>
      <entry name="version">EDIT THIS</entry>
      <entry name="serial">EDIT THIS</entry>
      <entry name="uuid">EDIT THIS</entry>
      <entry name="sku">EDIT THIS</entry>
      <entry name="family">EDIT THIS</entry>
    </system>
    <baseBoard>
      <entry name="manufacturer">EDIT THIS</entry>
      <entry name="product">EDIT THIS</entry>
      <entry name="version">EDIT THIS</entry>
      <entry name="serial">EDIT THIS</entry>
    </baseBoard>
    <chassis>
      <entry name="manufacturer">EDIT THIS</entry>
      <entry name="version">EDIT THIS</entry>
      <entry name="serial">EDIT THISr</entry>
    </chassis>
  </sysinfo>
...
</domain>

```
```

<os firmware="efi">
    ...
    <smbios mode="sysinfo"/>
</os>

```
```

<features>
    ...
    <kvm>
      <hidden state="on"/>
    </kvm>
    ...
</features>

```
```
enable rdp in windows vm and connect to it via  freerdp on linux host
xfreerdp -grab-keyboard /v:192.168.122.107 /u:shaquib /p:123456 /size:100% /dynamic-resolution /gfx-h264:avc444 +gfx-progressive

then install nvidia drivers via geforce experience app in vm and reboot 
```

```
/configuring ivshmem or kvmfr for igpu on host
https://looking-glass.io/docs/B6/module/#  //for kvmfrsetup

 
looking glass config for very low latency output

ge9's iddsampledriver for not using hdmi dummy plug

remember if using pipewire then pulseaudio must be replaced with pipewire-pulse

```

```
enable rdp on windows 10 home with rdp-wrap

https://github.com/asmtron/rdpwrap/blob/master/binary-download.md

```

```
//setup-kvmfr
https://gist.github.com/Ruakij/dd40b3d7cacf5d0f196d1116771b6e42

cd into cloned looking_glass_client dir and go to module directory

now open terminal in that dir and execute make command (manual method)
and do sudo modprobe kvmfr static_size_mb=32

or using dkms just execute below command
sudo dkms install "."


then create a udev rule
sudo cat > /etc/udev/rules.d/99-kvmfr.rules <<EOF
SUBSYSTEM=="kvmfr", OWNER="libvirt-qemu", GROUP="kvm", MODE="0666"
EOF

then setup auto loading during boot

Setup auto-load of module

sudo cat > /etc/modules-load.d/kvmfr.conf <<EOF
# KVMFR Looking Glass module
kvmfr
EOF

sudo cat > /etc/modprobe.d/kvmfr.conf <<EOF
#KVMFR Looking Glass module
options kvmfr static_size_mb=32
EOF


finally open /etc/libvirt/qemu.conf

and add /dev/kvmfr0 to this section

cgroup_device_acl = [
    "/dev/null", "/dev/full", "/dev/zero",
    "/dev/random", "/dev/urandom",
    "/dev/ptmx", "/dev/kvm",
    "/dev/kvmfr0"
]
```

```
enabling samba service
install samba package 'pacman -S samba'
copy below config to /etc/samba/smb.conf

#---------------------------------------

[global]

# workgroup = NT-Domain-Name or Workgroup-Name, eg: MIDEARTH
   workgroup = MYGROUP

# server string is the equivalent of the NT Description field
   server string = Samba Server

# this tells Samba to use a separate log file for each machine
# that connects
;   log file = /usr/local/samba/var/log.%m

# Put a capping on the size of the log files (in Kb).
   max log size = 50

# Windows Internet Name Serving Support Section:
# WINS Support - Tells the NMBD component of Samba to enable it's WINS Server
   wins support = yes

# sticky bit set on it to prevent abuse. Obviously this could be extended to
# as many users as required.
[myshare]
   comment = Holy Storage
   path = /media/shaquib
   public = no
   writable = yes
   browsable = yes


#---------------------------------------------------------------

then generate sambapasswd for current user
sudo smbpasswd -a shaquibimdad

finally eanble samba smb service
sudo systemctl enable --now smb.service

check your current ip in network in virt-manager

and connect with that in windows by mapping a new network device 
```
