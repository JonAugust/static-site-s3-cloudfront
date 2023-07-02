# static-site-s3-cloudfront
![This project is almost ready](https://img.shields.io/badge/Progress-Almost_There-orange)

This repo is a template to help people set us static hosting using S3 and CloudFront.  It also 
has a GitHub Actions workflow that will deploy any changes to S3 on a push event and then it 
will invalidate the CloudFront cache so you can see the changes immediately.

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
3. Select JSON and replace all the existing JSON with the following (replace the resources with your own bucket ARN):


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

Add a placeholder file, such as index.html containing "Hello!" to your S3 bucket. You can use an S3 client like Transmit to move the file to the bucket using the keys from Step 3. Without a placeholder, CloudFront and Route 53 may encounter issues later.


## Step 5: Create a CloudFront Distribution

1. Go to CloudFront and click "Create distribution".
2. Under "Origin domain", select your S3 bucket.
3. For "Origin access", click "Origin access control settings", then "Create control setting". Leave the options as they are and click "Create".
   - Don't worry about the bucket policy warning; we'll address it later.
4. Set "Viewer protocol policy" to "Redirect HTTP to HTTPS" and leave all other settings as is.
5. For "Web Application Firewall (WAF)", choose "Do not enable security protections" unless you plan to use WAF.
6. In "Settings", add all the desired Fully Qualified Domain Names (FQDNs) for your site, such as: mysite.com, w<span>ww.</span>mysite.com, mysite.net, and w<span>ww.</span>mysite.net
7. Click "Request certificate" under "Custom SSL certificate".
   - Select "DNS validation". AWS will add the validation records to your zone.
   - Once the certificate is "Issued", return to the CloudFront tab.
8. Select the certificate you created under "Custom SSL certificate". Click the refresh button if you don't see it.
9. Enter your "Default root object" (typically "index.html").
10. Click "Create distribution".
11. On the next screen, make note of your CloudFront Distribution ID.  You'll need this to grant invalidation permissions to your user.
12. Then, click "Copy policy" from inside the bucket policy warning. 

## Step 6: Update S3 Bucket Policy

1. Go to S3, click your bucket's name, and navigate to the "Permissions" tab.
2. Under "Bucket policy", click "Edit" and paste the policy you copied from CloudFront.
3. Click "Save changes".

## Step 7: Create an Invalidation Policy

1. Go to IAM and click on "Policies".
2. Click "Create Policy".
3. Select JSON and replace all the existing JSON with the following (replace the resources with your own bucket ARN):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "cloudfront:CreateInvalidation",
            "Resource": "arn:aws:cloudfront::987654321:distribution/ABCDEFGHIJKLMNO"
        }
    ]
}
```
4. Replace the 'ABCDEFGHIJKLMNO' with the CloudFront Distribution you recorded from Step 6. Replace the '987654321' with your AWS Account ID.
5. Click Next and name your policy.  I like to name these like this:  hello.internection.com-CloudFront-Invalidation
6. Add any tags you might want to apply, and then click "Create Policy"

## Step 8: Attach the Invalidation Policy to the User
1. Click "Users" on the left sidebar and click on your S3 user.
2. Click "Add Permissions" and select "Add Permissions"
3. Select "Attach policies directly", and find the Invalidation Policy you just created.
4. Check the box next to your policy and click "Next".
5. Click "Add Permission".

## Step 9: Point FQDN to CloudFront

1. Go to Route 53, click "Hosted Zones", then click your domain.
2. Edit or create a record for your FQDN.
   - Enter your hostname next to the domain to specify the FQDN.
   - Click the Alias slider, and select "Alias to CloudFront distribution" under "Route traffic to".
3. Click on the box below that to select the CloudFront distribution you created in Step 5
4. Leave the other options alone and click "Create records"

Now you should be able to visit the FQDN and see the placeholder. If you encounter issues, review these settings, paying special attention to the "Default root object".

Now we'll set the environment variables in GitHub so the GitHub Actions Workflow can sync your changes to S3 upon a push event.

----
# Setting up GitHub for your Workflow
1. Go to the GitHub repository you'll be syncing to your S3 / CloudFront instance. Click on the Settings Tab at the top.
2. Click on "Secrets and Variables" and then "Actions"
3. For each of the variables below, you'll click on the green "New repository secret" button. Enter the name and the secret for each and then click "Add secret"
   -  Be sure to replace the dummy values with your own

| Name                       | Sample Value                        | Note                                          |
|----------------------------|-------------------------------------|-----------------------------------------------|
| AWS_ACCESS_KEY_ID          | JQ469BPXZN31                        | Created in Step 3 of "Setting this up on AWS" |
| AWS_SECRET_ACCESS_KEY      | PZd4w6CMYA6ssnj4+WDUp+8gbXEJFhBVqQG | Created in Step 3 of "Setting this up on AWS" |
| AWS_S3_BUCKET              | static.mysite.com                   | Use the name of your bucket                   |
| AWS_REGION                 | us-east-1                           | Use your AWS region                           |
| CLOUDFRONT_DISTRIBUTION_ID | XXXXXXXXXXXXX                       | Noted in Step 5 of "Setting this up on AWS"   |

----

Your GitHub Action is ready!  Now when you push the main branch to your repo, the workflow will trigger.  The first part 
of the workflow will sync the changes to your S3 bucket. When that has completed successfully, the second job will 
invalidate your CloudFront distribution so you can see the changes immediately. You can monitor the progress of the 
workflow by clicking on the Actions tab at the top of your repository.