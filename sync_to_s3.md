**Synchronize local files to s3 bucket**
----
_This tutorial shows how to synchronize local files to s3 bucket_

[Link 1](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html)
[Link 2](https://docs.aws.amazon.com/cli/latest/reference/s3/sync.html)

# 1. Bucket and user creation
First create a new bucket with encyrption and versionning.
Then create a new user with programmatic access
Use Attach existing policies and create a new policy

In the policy, put
```
Statement": [
    {
        "Effect": "Allow",
        "Action": "s3:*",
        "Resource": [
            "arn:aws:s3:::your-bucket",
            "arn:aws:s3:::your-bucket/*"
        ]
    }
]
```

Save the AWS key and the secret key

# 2. Synchronize all your files
First install aws cli
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Then configure AWS and add you api keys
```
aws configure
```
You can put json as default format

Create your root folder into your s3 bucket (root-folder)

Now sync all your folders to the s3 bucket
```
aws s3 sync root-folder/ s3://plateforme-retraite/root-folder/
```

It should be done
