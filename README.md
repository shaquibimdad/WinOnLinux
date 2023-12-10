# WinOnLinux
```
//check virtualization
lscpu | grep Virtualization
VT-x = Intel
AMD-v = AMD

//get dedicated gfx pci ids
lspci -nn | grep -E "NVIDIA"

pcibus    ids 
1:00.0 = 10de:1f99
1:00.1 = 10de:10fa

//add iommu and vfio in kernel cmdline (grub or systemd)

//for intel
intel_iommu=on iommu=pt vfio-pci.ids=10de:1f99,10de:10fa
//for amd
amd_iommu=on iommu=pt vfio-pci.ids=10de:1f99,10de:10fa

//edit or add this file /etc/modprobe.d/vfio.conf
options vfio-pci ids=10de:1f99,10de:10fa
softdep nvidia pre: vfio-pci

get your kernel name with uname -r command
//run below command
sudo mkinitcpio -p linux 

// reboot system

//create vm
...vmconfigs
...virtio.iso inside windows vm
sudo strings /sys/firmware/acpi/tables/MSDM //get oem key attached to your motherboard
...passing smbios as sysinfo for windows activation inside vm with oem keys

--------------------------------------------------------------------------------------

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

-------------------------------------------------------

<os firmware="efi">
    ...
    <smbios mode="sysinfo"/>
</os>

----------------------------------------------------

<features>
    ...
    <kvm>
      <hidden state="on"/>
    </kvm>
    ...
</features>

----------------------------------------------------

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





Looking-glass kvmfr setup Arch
===

More in-depth setup-guide for usage of kvmfr (see https://looking-glass.io/docs/B6/module/)

This setup was done on EndeavourOs (Arch) on Kernel 6.1.12-arch1-1. <br>
QEMU-version 7.2 and libvirt version 9.0.0 . <br>
Running cGroups Policy (if you have AppArmor, some things are different)

Check respective sections in looking-glass docs for other versions (specifcally QEMU <6.2 and libvirt <7.9)

<br>

# 1. Prerequisites
## 1.1. VM
You need a VM which is fully set-up with graphics-driver and the looking-glass host-program.

Documentation about that can be found here: https://looking-glass.io/docs/B6/install/

<br>

## 1.2. Packages
You need the following packages
- `sudo` (or do things as root)
- `git`
- `dkms`
- `linux-headers` (same version as kernel, probably already installed)

<br>

## 1.3. MEM-size 

This guide uses the default 32MB size, which is fine for 1080p.

For 1440p or 2160p you need to use 64MB.

See [this section](https://looking-glass.io/docs/B6/install/#libvirt-determining-memory) to determine how much you need.

<br>

Formula
```
pixel size for SDR is 4
pixel size for HDR is 8

width x height x pixel size x 2 = frame bytes
frame bytes / 1024 / 1024 = frame megabytes
frame megabytes + 10 MiB = total megabytes

Round up to nearest power of 2. (32, 64, 128, ..)
```

<br>

# 2. Setup
## 2.1. kvmfr
Clone repository and enter module folder
```sh
git clone https://github.com/gnif/LookingGlass
cd LookingGlass/module
```

Compile and install module with dmks
```sh
sudo dkms install "."
```

Setup udev-rule for module to set permissions<br>
Deviate from docs to make user-independent
```sh
sudo cat > /etc/udev/rules.d/99-kvmfr.rules <<EOF
SUBSYSTEM=="kvmfr", OWNER="libvirt-qemu", GROUP="kvm", MODE="0666"
EOF
```

Setup auto-load of module
```sh
sudo cat > /etc/modules-load.d/kvmfr.conf <<EOF
# 3. KVMFR Looking Glass module
kvmfr
EOF

sudo cat > /etc/modprobe.d/kvmfr.conf <<EOF
#KVMFR Looking Glass module
options kvmfr static_size_mb=32
EOF
```

<br>

## 3.1. libvirt-qemu
### 3.1.1. Setup cgroups-policy to allow qemu access to device
1. Open `/etc/libvirt/qemu.conf`
2. Find `cgroup_device_acl`
3. Uncomment everything in that block and add `"/dev/kvmfr0"`

Looks something like this:
```sh
# 4. This is the basic set of devices allowed / required by
# 5. all virtual machines.
#
# 6. As well as this, any configured block backed disks,
# 7. all sound device, and all PTY devices are allowed.
#                                                                                                          
# 1. This will only need setting if newer QEMU suddenly
# 2. wants some device we don't already know about.
#
cgroup_device_acl = [
    "/dev/null", "/dev/full", "/dev/zero",
    "/dev/random", "/dev/urandom",
    "/dev/ptmx", "/dev/kvm",
    "/dev/kvmfr0"
]
```

4. Save file and restart libvirtd <br>
`sudo systemctl restart libvirtd`

<br>

### 2.0.1. Setup VM to use MEM-device
1. Open virtual machine manager
2. Your VM -> Details -> XML

3. First line has to be
```xml
<domain xmlns:qemu="http://libvirt.org/schemas/domain/qemu/1.0" type="kvm">
```

4. Go to the bottom and add this block
```xml
<qemu:commandline>
  <qemu:arg value='-device'/>
  <qemu:arg value='{"driver":"ivshmem-plain","id":"shmem0","memdev":"looking-glass"}'/>
  <qemu:arg value='-object'/>
  <qemu:arg value='{"qom-type":"memory-backend-file","id":"looking-glass","mem-path":"/dev/kvmfr0","size":33554432,"share":true}'/>
</qemu:commandline>
```

`size` is your MEM-size in bytes ([looking-glass docs](https://looking-glass.io/docs/B6/module/#libvirt))<br>
Formula: `MEM-size x 1024 x 1024`

<br>

5. Save and check if everything is still there<br>
libvirt might replace `"` with `&quot;`, thats fine

<br>

## 2.1. Looking-Glass Client
You might not have to specify the mem-device as it already detects the correct one

Feels free to add a config-file or manually add the `-f` Option if not

<br>

# 3. Testing
1. Load module manually
```sh
sudo modprobe kvmfr static_size_mb=32
```

2. Check kernel-log for output of module
```sh
sudo dmesg | grep kvmfr
```
Should return this:
> kvmfr: creating 1 static devices

<br>

3. Check created node and permissions
```sh
ls -la /dev/kvmfr*
```
Should return this:
> crw-rw-rw- 1 libvirt-qemu kvm 510, 0 23. Feb 10:00 /dev/kvmfr0

<br>

4. Now try to start the VM and then looking-glass
