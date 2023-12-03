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
 
looking glass config for very low latency output

ge9's iddsampledriver for not using hdmi dummy plug

remember if using pipewire then pulseaudio must be replaced with pipewire-pulse

```

```
enable rdp on windows 10 home with rdp-wrap

https://github.com/asmtron/rdpwrap/blob/master/binary-download.md

```
