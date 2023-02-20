# EC2 Linux Upsize EBS Volume



If an EC2 instance is running out of disk space, we first add more space to its underlying EBS volume, then we need to extend the OS file system.

In this example, we increase the disk space size of an Ubuntu EC2 instance from 8GB to 16GB.

Note: This process does not need to detach the disk or restart the instance.



1. Create snapshot for the EC2 instance.

2. Modify the EBS volume and add more space to it.

3. In the terminal, check the current partition, which still shows the old space size of 8GB.

   ```bash
   df -h
   ```

4. List all partitions in the in EBS, which shows the disk size and its partitions. The disk is currently 16GB, where its first partition is allocated with 8GB.

   ```bash
   lsblk
   ```

5. Allocate the un-used space to partition 1.

   * `/dev/nvme0n1` is the identifier of the disk, where `/dev/` is prefixed to the disk name `nvme0n1`.
   * `1` is referring to the partition 1.

   ```
   sudo growpart /dev/nvme0n1 1
   ```

6. After the partition is allocated with more space, we need to update OS volume info.

   * `/dev/nvme0n1p1` is the identifier of the partition, where `/dev/` is prefixed to the partition name `nvme0n1p1`.

   ```bash
   sudo resize2fs /dev/nvme0n1p1
   ```

7. Check the volume size again. 

   ```
   df -h
   ```

