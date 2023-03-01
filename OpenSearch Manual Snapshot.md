# OpenSearch Manual Snapshot



1. Registration of snapshot repository can NOT be done through API call using Dev Tool plugin or simple `curl` command because they don't support AWS request signing. Instead, use Python client or Postman. 

```
PUT _snapshot/my-repo
{
  "type": "s3",
  "settings": {
    "bucket": "elasticsearch-snapshots-460453255610",
    "region": "ap-southeast-1",
    "role_arn": "arn:aws:iam::460453255610:role/ElasticSearchSnapshotLambdaRole",
    "server_side_encryption": true
  }
}
```

2. Verify snapshot repository.

```
GET _snapshot
```

