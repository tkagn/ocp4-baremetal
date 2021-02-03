# README.md

## Download the RHEL CoreOS 
```bash
mkdir {WorkingDir}
wget https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/latest/rhcos-installer.x86_64.iso
```

## Mount iso and copy to contents to working directory
```bash
mkdir {WorkingDIR}/ISO
mkdir /mnt/RHCOS
mount {WorkingDIR}/rhcos-installer.x86_64.iso /mnt/RHCOS
cp -R /mnt/RHCoreOS/* .{WorkingDir}/ISO
```
## Edit isolinux.cfg to reflect static IPs, Hostnames, DNS, and ignition file locations
isolinux.cfg should contain kernel cmd line entries for each machine and it's respective static ip
```bash
cd <WorkingDir>/ISO
vi isolinux/isolinux.cfg
```
contents:
```
# label linuxmaster<#x>
#   menu label ^Install RHEL CoreOS - Master<#x>
#   kernel /images/vmlinuz
#   append initrd=/images/initramfs.img nomodeset rd.neednet=1 coreos.inst=yes  ip=<static IP>::<GW>:<Mask>:<desired FQDN>::none nameserver=<DNS server> coreos.inst.install_dev=sda coreos.inst.image_url=http://<IP of FQDN of webserver hosting files>/rhcos-metal.x86_64.raw.gz coreos.inst.ignition_url=http://<IP of FQDN of webserver hosting files>/master.ign
```
Review [isolunx.cfg](isolinux.cfg) for an example.
## Compile new iso
```bash
cd <WorkingDir>/ISO
mkisofs -o <WorkingDir>/ocp4bootstrap.iso -rational-rock -J -joliet-long -eltorito-boot isolinux/isolinux.bin -eltorito-catalog isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table .
```
## Burn iso to cd/usb or use for VM booting.
