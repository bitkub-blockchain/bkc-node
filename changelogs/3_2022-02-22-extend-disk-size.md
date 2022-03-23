# 3_2022-02-22-extend-disk-size

## Topic
- Extend the file system

## Detail
- Extend volumn `/dev/vdb` size `2Gb` to 

## Timeline
- 28-29 Mar 2022

## Step
1. Switch Admin User
```sh
su -u
cd ~
```

2. Display all disk device
```
lsblk
```

3. Extend the file system
```
resize2fs /dev/vdb
```

## Reference
- https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/recognize-expanded-volume-linux.html