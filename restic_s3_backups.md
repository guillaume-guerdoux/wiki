# Setting up restic with Amazon S3

[Link 1](https://restic.readthedocs.io/en/latest/080_examples.html#preface)
[Link 2](https://linuxhint.com/how-to-install-and-use-restic-on-ubuntu-18-04/)
[Link 3](https://medium.com/@denniswebb/fast-and-secure-backups-to-s3-with-restic-49fd07944304)

Preface
=======

This tutorial will show you how to use restic with AWS S3. It will show you how
to navigate the AWS web interface, create an S3 bucket, create a user with
access to only this bucket, and finally how to connect restic to this bucket.

Installation
=============
`sudo apt update`
`sudo apt-get install restic`

Try if restic is installed with `restic`

Logging into AWS
================

Point your browser to
https://console.aws.amazon.com
and log in using your AWS account. You will be presented with the AWS homepage:

By using the "Services" button in the upper left corder, a menu of all services
provided by AWS can be opened:

For this tutorial, the Simple Storage Service (S3), as well as Identity and
Access Management (IAM) are relevant.

Creating the bucket
===================

First, a bucket to store your backups in must be created. Using the "Services"
menu, navigate to S3.

Click the "Create bucket" button and choose a name and region (frankfort) for your new
bucket. For the purpose of this tutorial, the bucket will be named
``restic-demo`` and reside in Frankfurt. Because the bucket name space is
shared among all AWS users, the name ``restic-demo`` may not be available to
you. Be creative and choose a unique bucket name.

It is not necessary to configure any special properties or permissions of the
bucket just yet. Therefore, just finish the wizard without making any further
changes:

Creating a user
===============

Use the "Services" menu of the AWS web interface to navigate to IAM. This will
bring you to the IAM homepage. To create a new user, click on the "Users" menu
entry on the left:

In case you already have set-up users with IAM before, you will see a list of
them here. Use the "Add user" button at the top to create a new user:

For this tutorial, the new user will be named ``restic-demo-user``. Feel free to
choose your own name that best fits your needs. This user will only ever access
AWS through the ``restic`` program and not through the web interface. Therefore,
"Programmatic access" is selected for "Access type":

During the next step, permissions can be assigned to the new user. To use this
user with restic, it only needs access to the ``restic-demo`` bucket. Select
"Attach existing policies directly", which will bring up a list of pre-defined
policies below. Afterwards, click the "Create policy" button to create a custom
policy:

A new browser window or tab will open with the policy wizard. In Amazon IAM,
policies are defined as JSON documents. For this tutorial, the "Visual editor"
will be used to generate a policy:

For restic to work, two permission statements must be created using the visual
policy editor. The first statement is set up as follows:

.. code::

   Service: S3
   Allow Actions: DeleteObject, GetObject, PutObject
   Resources: arn:aws:s3:::restic-demo/*

This statement allows restic to create, read and delete objects inside the S3
bucket named ``restic-demo``. Adjust the bucket's name to the name of the
bucket you created earlier. Next, add a second statement using the "Add
additional permissions" button:

.. code::

   Service: S3
   Allow Actions: ListBucket, GetBucketLocation
   Resource: arn:aws:s3:::restic-demo

Again, substitute ``restic-demo`` with the actual name of your bucket. Note
that, unlike before, there is no ``/*`` after the bucket name. This statement
allows restic to list the objects stored in the ``restic-demo`` bucket and to
query the bucket's region.

Continue to the next step by clicking the "Review policy" button and enter a
name and description for this policy. For this tutorial, the policy will be
named ``restic-demo-policy``. Click "Create policy" to finish the process:

Go back to the browser window or tab where you were previously creating the new
user. Click the button labeled "Refresh" above the list of policies to make
sure the newly created policy is available to you. Afterwards, use the search
function to search for the ``restic-demo-policy``. Select this policy using the
checkbox on the left. Then, continue to the next step.

The next page will present an overview of the user account that is about to be
created. If everything looks good, click "Create user" to complete the process:

After the user has been created, its access credentials will be displayed. They
consist of the "Access key ID" (think user name), and the "Secret access key"
(think password). Copy these down to a safe place.

You have now completed the configuration in AWS. Feel free to close your web
browser now.


Initializing the restic repository
==================================

Open a terminal and make sure you have the ``restic`` binary ready.

Using your favorite text editor create a file named .restic.env in your home directory. Inside this file, we are going to add lines for each of the 4 required environment variables. Use the example block below for yours, replacing the values with your own.

.. code-block:: console

   export AWS_ACCESS_KEY_ID="AKIAJNFRE7YAHGDUMD"
   export AWS_SECRET_ACCESS_KEY="EtE7mGPqcbjyPT4KLXX1qVDUkAVhA"
   export RESTIC_PASSWORD="!emP*#IWFEoy8%@zp9qQpIHo"
   export RESTIC_REPOSITORY="s3:https://s3.amazonaws.com/BUCKET_NAME"

Let’s protect this file from other users by changing the permissions on the file to read-only by your user with chmod 400 ~/.restic.env. To activate these environment variables into your current bash session run source ~/.restic.env. I don’t like having credentials in my shell all the time. This method allows me to only have them active when I need them.

Before Restic can backup anything, you must first initialize its repository. This only has to be done once. To do so run `source ~/.restic.env ; restic init`

Create a file `nano backupToS3.sh` with
```
echo 'Started'
date +'%a %b %e %H:%M:%S %Z %Y'
. /home/guillaume/.restic.env
dpkg --get-selections > dpkg.list
/usr/bin/restic backup -q /var/lib/postgresql/backup/XXXXX.bak
/usr/bin/restic forget -q --prune --keep-daily 3
date +'%a %b %e %H:%M:%S %Z %Y'
echo 'Finished'
```

Run `chmod +x backupToS3.sh`
Try it by doing `./backupToS3.sh`

Create a crontab : `sudo crontab -e`
Add `0 3 * * * sh /home/guillaume/backupToS3.sh > /home/guillaume/backupToS3.logs`

Restoring the restic repository
==================================

Create a restic.env file with the same

```
export AWS_ACCESS_KEY_ID="AKIAJNFRE7YAHGDUMD"
export AWS_SECRET_ACCESS_KEY="EtE7mGPqcbjyPT4KLXX1qVDUkAVhA"
export RESTIC_PASSWORD="!emP*#IWFEoy8%@zp9qQpIHo"
export RESTIC_REPOSITORY="s3:https://s3.amazonaws.com/BUCKET_NAME"
```

`source restic.env`
`restic init`
`restic snapshots`
`sudo ./restic restore 10fdbace --target restore`
