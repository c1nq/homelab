# Proxmox Storage Setup

## Storage mounten
`ash
mkfs.ext4 /dev/sdb
mkfs.ext4 /dev/sdc
mkdir -p /mnt/storage-2tb
mkdir -p /mnt/storage-7tb
echo "UUID=  /mnt/storage-2tb  ext4  defaults  0  2" >> /etc/fstab
echo "UUID=  /mnt/storage-7tb  ext4  defaults  0  2" >> /etc/fstab
systemctl daemon-reload && mount -a
`

## Storage overzicht
| Device | Mountpoint | Grootte | Filesystem |
|--------|-----------|---------|------------|
| /dev/sda | / (Proxmox OS) | 930 GB | ext4/LVM |
| /dev/sdb | /mnt/storage-2tb | 2.7 TB | ext4 |
| /dev/sdc | /mnt/storage-7tb | 7.3 TB | ext4 |
