
# Upgrade Validator Disk

- Time 19.00 - 21.00
- Date 22 Nov 2021

Step (POC)
1. Create VM Sync full
2. Create DataDisk 
3. Mount Data Disk to `/bkc-node` to Data Disk
```bash
// Create folder
sudo mkdir /bkc-node
// Format disk to ext4
sudo mkfs -t ext4 /dev/vdb
// Mount Volumn
sudo mount /dev/vdb /bkc-node/
// Change Owner 
sudo chown -R $USER /bkc-node
// Verify Disk Size
cd /bkc-node &&  df -h .
// BackUp file system file mount
cp /etc/fstab /etc/fstab.bak
// Add mount dist to fstab
sudo nano /etc/fstab
/dev/vdb       /bkc-node   ext4    defaults,nofail        0       0
```
4. With for Sync Success
5. Create Snapshot at 18.30 (ref1)


# Implementation Steps - Validators
## Pre-implementation
1. Create Data Disk from Snapshot from Ref1 and attach to the servers
2. Verify block devices (Disk Sequence)
```
lsblk
```
#### Remark: If /dev/vdb does not match, then unmount the disk and reboot.
#
## Implementation Steps
### 1. Create new folder
```bash
mkdir /bkc-node
```

### 2. Backup and update `/etc/fstab` file
```bash
cp -p /etc/fstab /etc/fstab.bak
nano /etc/fstab

/dev/vdb       /bkc-node   ext4    defaults,nofail        0       0
```

### 3. Mount all filesystem and change permission
```bash
mount -a
chown -R $USER /bkc-node
```


### 4. Verify Disk Size (Should be 2TB)
```bash
cd /bkc-node &&  df -h .
```

### 5. Create new directory for geth
```bash
mkdir -p /bkc-node/mainnet/data/geth
mkdir -p /bkc-node/mainnet/data/keystore
```

### 6. Copy Keystore to new destination
```bash
cp -p /root/bkc-node/bkc-mainnet/node0/keystore/* /bkc-node/mainnet/data/keystore/
```
### 7. Copy nodekey to new destination
```
cp -p /root/bkc-node/bkc-mainnet/node0/geth/nodekey /bkc-node/mainnet/data/geth/nodekey
```

### 8. Backup `startnode0.sh` before update flags
```bash
cp -p /root/bkc-node/bkc-mainnet/node0/startnode0.sh /root/bkc-node/bkc-mainnet/node0/startnode0.sh.bak
```
```bash
nano /root/bkc-node/bkc-mainnet/node0/startnode0.sh
```

- `--datadir`
```
/bkc-node/mainnet/data
```

- `--snaphot`
```
--snapshot=false
```

- `--cache`
```
8192
```

### 9. Reload systemd configuration and restart the service.
```bash
systemctl daemon-reload
systemctl restart geth.service
```
### 10. Verify and monitor (geth.service) logs
```bash
journalctl -fu geth.service
```
#
## Rollback Steps
### 1. Stop geth.service
```bash
systemctl stop geth.service
```
### 2. Revert backup configuration
```bash
mv /root/bkc-node/bkc-mainnet/node0/startnode0.sh.bak /root/bkc-node/bkc-mainnet/node0/startnode0.sh
```
### 3. Remove new disk from /etc/fstab
```bash
nano /etc/fstab
```

### 4. Start geth.service
```bash
systemctl start geth.service
```