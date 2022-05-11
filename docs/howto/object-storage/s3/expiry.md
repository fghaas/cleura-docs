# Object expiry

You can set a bucket's lifecycle configuration such that it automatically 
deletes objects after a certain number of days. 

First, you need to create a JSON file, `lifecycle.json`, that contains
the lifecycle configuration rule. Be sure to set `Days` to your
desired value:

```json
{
  "Rules": [{
    "ID": "cleanup",
    "Status": "Enabled",
    "Prefix": "",
    "Expiration": {
        "Days": 5
    }
  }]
}
```

Then, apply this lifecycle configuration to your bucket using one of
the following commands:

=== "aws"
    ```bash
    aws --endpoint-url=https://s3-<region>.{{extra.brand_domain}}:8080 \
      s3api put-bucket-lifecycle-configuration \
	  --lifecycle-configuration file://lifecycle.json \ 
	  --bucket <bucket-name>
    ```
=== "mc"
    ```bash
    mc ilm import <bucket-name> < lifecycle.json
    ```
=== "s3cmd"
    ```bash
	s3cmd setlifecycle lifecycle.json s3://<bucket-name>
    ```
