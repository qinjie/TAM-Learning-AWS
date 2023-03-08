# AWSCLI List EBS Volume



List all EBS volumes.

```
aws ec2 describe-volumes --query 'Volumes[*].[AvailabilityZone,CreateTime,Encrypted,SnapshotId,VolumeId,State,MultiAttachEnabled,VolumeType,Size,Iops]' --output text
```

List all EBS valumes of GP2 type.

```
aws ec2 describe-volumes --filter Name=volume-type,Values=gp2 --query 'Volumes[*].[AvailabilityZone,CreateTime,Encrypted,SnapshotId,VolumeId,State,MultiAttachEnabled,VolumeType,Size,Iops]' --output text
```





## GP2 Performance



### GP2 Throughput performance

`gp2` volumes deliver throughput between 128 MiB/s and 250 MiB/s, depending on the volume size. Throughput performance is provisioned as follows:

- Volumes that are 170 GiB and smaller deliver a maximum throughput of 128 MiB/s.
- Volumes larger than 170 GiB but smaller than 334 GiB can burst to a maximum throughput of 250 MiB/s.
- Volumes that are 334 GiB and larger deliver 250 MiB/s.

Throughput for a `gp2` volume can be calculated using the following formula, up to the throughput limit of 250 MiB/s:

```
Throughput in MiB/s = IOPS performance × I/O size in KiB
```

The **Maximum I/O Size** per I/O for SSD Volumes is `256 KiB/IO`.

#### Reference
- https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/general-purpose.html#gp2-performance
- https://www.saisci.com/aws/gp2-throughput-explained-ebs-volumes/
- https://www.saisci.com/aws/ebs-volumes-throughput-explained/#DataThroughputIOPS





## Info

Pricing https://aws.amazon.com/ebs/pricing/

Volume Types: https://aws.amazon.com/ebs/volume-types/

Old Generation Volume Types: https://aws.amazon.com/ebs/previous-generation/

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html




### Standard (Magnetic) Volume Info

| **Volume type**               | standard                                  |
| ----------------------------- | ----------------------------------------- |
| **Volume size**               | 1 GiB – 1 TiB                             |
| **Max throughput per volume** | 40 – 90 MiB/s                             |
| **Max IOPS per volume**       | 40 – 200                                  |
| **Use cases**                 | Workloads with infrequently accessed data |
| **Boot volume**               | Supported                                 |



#### Pricing

| Region         | Type                                | Price  |
| -------------- | ----------------------------------- | ------ |
| ap-southeast-1 | per GB-month of provisioned storage | $0.08  |
| ap-southeast-1 | per 1 million I/O requests          | $0.08  |
| eu-central-1   | per GB-month of provisioned storage | $0.059 |
| eu-central-1   | per 1 million I/O requests          | $0.059 |
| us-west-1      | per GB-month of provisioned storage | $0.08  |
| us-west-1      | per 1 million I/O requests          | $0.08  |






### Cold HDD (sc1) Volumes

Cold HDD is another type of Hard disk drive provided by AWS for less frequently accessed workloads but higher throughput. Cold HDD is different from Throughput optimized based on IOPS. It has lesser IOPS than Throughput optimized HDD. Like Throughput Optimized HDD, the Cold HDD also provides a baseline throughput of 12 MiB/s per TiB storage provisioned. Following is the performance chart of Cold HDD EBS volume.

|                               | **Cold HDD**                                                 |
| ----------------------------- | ------------------------------------------------------------ |
| **Volume type**               | sc1                                                          |
| **Volume size**               | 125 GiB – 16 TiB                                             |
| **Use cases**                 | Throughput oriented storage for less frequently accessed dataUse cases where lowest-cost storage required |
| **Durability**                | 99.8% – 99.9% durability                                     |
| **Amazon EBS Multi-attach**   | Not supported                                                |
| **Max IOPS per volume**       | 250                                                          |
| **Max throughput per volume** | 250 MiB/s                                                    |
| **Boot volume**               | Not supported                                                |



#### Pricing

| Region         | Price                                      |
| -------------- | ------------------------------------------ |
| us-west-1      | $0.018 per GB-month of provisioned storage |
| eu-central-1   | $0.018 per GB-month of provisioned storage |
| ap-southeast-1 | $0.018 per GB-month of provisioned storage |







https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-optimized.html#ebs-optimization-support

