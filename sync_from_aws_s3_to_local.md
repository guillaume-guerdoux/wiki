**Synchronize s3 bucket to local **
----
_This tutorial shows how to synchronize s3 bucket to local files_


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
aws configure --profile NAME_PROFILE
```
You can put json as default format

Create your root folder into your s3 bucket (root-folder)

Now sync all your folders to the s3 bucket
```
export AWS_PROFILE=NAME_PROFILE
aws s3 sync  s3://BUCKER folder
```

It should be done
