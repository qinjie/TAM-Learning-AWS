# Learn AWS Data Analysis

### Upload Data to S3

Create new S3 bucket in a region.

```
aws s3 mb s3://<bucket-name> --region <region>
```

Upload a file to s3 bucket in a storage class, e.g. STANDARD_IA.

```
aws s3 cp <file-path> s://<bucket-name>/<prefix> --storage-class <class-name>
```

Upload a folder to s3 bucket.

```
aws s3 cp <folder-path> s3://<bucket-name>/<prefix> --recursive
```

Count number of lines in a file.

```
wc -l <filename>
```

Split large file into smaller ones.

* `-d -a 4` append 4 digits at the back of file name.
* `-l 10000` max 10000 lines in a file.

```
split -d -a 4 -l 10000
```

