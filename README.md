# static-site-s3-cloudfront
![This project is almost ready](https://img.shields.io/badge/Progress-Almost-There-orange)

This repo is a template to help people set us static hosting using S3 and Cloudfront.  It also 
has a GitHub Actions workflow that will deploy any changes to S3 on a push event and then it 
will invalidate the Cloudfront cache so you can see the changes immediately.

## TODO:
- Explain how to set the environment variables

## Future:
- Make a version of this template that can build Hugo files

# Setting this up on AWS:

These instructions assume you have an AWS account and your domain is hosted in Route 53.  Let me know if you'd like details on moving your domain to Route 53 and I'll write up some instructions for that too.

## Step 1: Create a Bucket in S3

1. Go to S3 and click on "Create a bucket".
2. Enter a name for the bucket. I recommend using the hostname of your site.
3. Leave all other options as is, and click "Create Bucket".

## Step 2: Create an IAM Policy and User

First, we'll create a policy that allows a user to access the bucket.

1. Go to IAM and click on "Policies".
2. Click "Create Policy".
3. Select JSON and replace all the existing JSON with the following:


```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "SyncAccess",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObjectAcl",
                "s3:GetObject",
                "s3:ListBucket",
                "s3:DeleteObject",
                "s3:PutObjectAcl"
            ],
            "Resource": [
                "arn:aws:s3:::hello.internection.com",
                "arn:aws:s3:::hello.internection.com/*"
            ]
        }
    ]
}
```

4. Click Next and name your policy.  I like to name these like this:  hello.internection.com-GitHub-Actions-Sync
5. Add any tags you might want to apply, and then click "Create Policy"

Now we'll create a user and assign this policy to them:

1. Click "Users" on the left sidebar and click "Add users".
2. Enter a user name, like helloinxn.
3. Click "Next", select "Attach policies directly", and find the policy you just created.
4. Check the box next to your policy, add any tags you'd like, and click "Next".
5. Click "Create user".

## Step 3: Generate Access Keys

1. Click the user you just created from the list.
2. Click the "Security Credentials" tab and find the "Access Keys" section.
3. Click "Create access key", select "Application running outside AWS", and click "Next".
4. Enter a tag if you'd like, then click "Create access key".
5. Copy both keys and store them securely.
6. Click "Done".


## Step 4: Add a Placeholder to the Bucket

Add a placeholder file, such as index.html containing "Hello!" to your S3 bucket. You can use an S3 client like Transmit to move the file to the bucket using the keys from Step 3. Without a placeholder, Cloudfront and Route 53 may encounter issues later.


## Step 5: Create a Cloudfront Distribution

1. Go to Cloudfront and click "Create distribution".
2. Under "Origin domain", select your S3 bucket.
3. For "Origin access", click "Origin access control settings", then "Create control setting". Leave the options as they are and click "Create".
   - Don't worry about the bucket policy warning; we'll address it later.
4. Set "Viewer protocol policy" to "Redirect HTTP to HTTPS" and leave all other settings as is.
5. For "Web Application Firewall (WAF)", choose "Do not enable security protections" unless you plan to use WAF.
6. In "Settings", add all the desired Fully Qualified Domain Names (FQDNs) for your site, such as: mysite.com, w<span>ww.</span>mysite.com, mysite.net, and w<span>ww.</span>mysite.net
7. Click "Request certificate" under "Custom SSL certificate".
   - Select "DNS validation". AWS will add the validation records to your zone.
   - Once the certificate is "Issued", return to the Cloudfront tab.
8. Select the certificate you created under "Custom SSL certificate". Click the refresh button if you don't see it.
9. Enter your "Default root object" (typically "index.html").
10. Click "Create distribution".
11. On the next screen, click "Copy policy" to address the bucket policy warning.


## Step 6: Update S3 Bucket Policy

1. Go to S3, click your bucket's name, and navigate to the "Permissions" tab.
2. Under "Bucket policy", click "Edit" and paste the policy you copied from Cloudfront.
3. Click "Save changes".


## Step 7: Point FQDN to Cloudfront

1. Go to Route 53, click "Hosted Zones", then click your domain.
2. Edit or create a record for your FQDN.
   - Enter your hostname next to the domain to specify the FQDN.
   - Click the Alias slider, and select "Alias to Cloudfront distribution" under "Route traffic to".
3. Click on the box below that to select the Cloudfront distribution you created in Step 5
4. Leave the other options alone and click "Create records"

Now you should be able to visit the FQDN and see the placeholder. If you encounter issues, review these settings, paying special attention to the "Default root object".