# static-site-s3-cloudfront
![This project still in the works](https://img.shields.io/badge/Progress-Still_in_the_works-red)

This repo is a template to help people set us static hosting using S3 and Cloudfront.  It also 
has a GitHub Actions workflow that will deploy any changes to S3 on a push event and then it 
will invalidate the Cloudfront cache so you can see the changes immediately.

## TODO:
- Explain how to set up S3 and Cloudfront
- Explain how to set the environment variables

## Future:
- Make a version of this template that can build Hugo files

# Setting this up on AWS:

These instructions assume you have an AWS account and your domain is hosted in Route 53.  Let me know if you'd like details on moving your domain to Route 53 and I'll write up some instructions for that too.

1. Go to S3.  Create a bucket.  Enter a name for the bucket.  I like to name the bucket after the hostname of the site.  Leave all the other options alone and click Create Bucket.


2. We're going to create a user, but first we need to create a policy for the user so it can only access this bucket.  Go to IAM and click on Policies.
    - Then click "Create Policy"
    - select JSON and replace all the existing JSON with this:

```
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

- Click Next and name your policy.  I like to name these like this:  hello.internection.com-GitHub-Actions-Sync
- Add any tags you might want to apply, and then click "Create Policy"

Now click on users on the left side and click "Add users".  I like to name the users something similar to the site:  helloinxn
- Click Next
- Select "Attach policies directly"
- Locate the policy you just created in the list.  The search is helpful for this.
- Check the box next to your policy and then click Next
- Add any tags you may want to apply and then click "Create user"

3. Now click the user you just created from the list of users.  Now we're going to make an access and secret key for the s3 bucket.
- Click on the Securty Credentials tab
- Scroll a little bit until you see Access Keys.  Click "Create access key" and select "Application running outside AWS".  Click Next
- Enter a tag if you'd like and then click "Create access key"
- Make sure you copy each of the keys using the overlapping squares icon and store them in a safe place for now.
- Click Done.  You can continue if it warns you that you haven't viewed the secret key.


4. Put a placeholder in the bucket.  I usually put an index.html in there with just "Hello!" in it.  You can use an S3 client like Transmit to move the file to the bucket using the keys from the last step.  Without a placeholder, we may have a problem with Cloudfront and Route 53 later.


5. Now we're going to make a Cloudfront distribution.  Go to Cloudfront.
- Click "Create distribution"
- Click on the Origin domain box and select your s3 bucket from the list.
- Under Origin access, click on Origin access control settings.  Then click on "Create control setting".  Leave the options as is and click Create.
      
  - You'll see a warning about applying a bucket policy.  Don't worry, we'll handle that in a minute.
- Leave all the settings in "Origin" and "Default cache behavior" as they are except "Viewer protocol policy".  Change that to "Redirect HTTP to HTTPS"
  - Leave the settings in "Function associations" alone.
  - Under "Web Application Firewall (WAF)" click "Do not enable security protections" unless you want to pay for WAF
  - In Settings, click Add item to add all the various FQDNs you'd want this site to have.  For example:
      - mysite.com
      - www.mysite.com
      - mysite.net
      - www.mysite.net
      
      If you have only one FQDN, enter just that one here.
  - Click "Request certificate" Under Custom SSL certificate.
      - Click Next when you see "Request a public certificate"
      - Enter the main FQDN in the box for Fully Qualified Domain Name
      - Click "Add another name to this certificate" for all the names you added in the last step.
      - Select DNS validation.  AWS will add the validation records to your zone for you.
      - Add any tags you might want and then click Request.
      - On the next screen click the refresh button to see your Pending Certificate.  Click on the certificate ID to see the details about validation.
          - Click "Create records in Route 53".  And then click "Create records" on the next screen
          - Got back to Certificates screen and wait patiently.  You can click refresh to update the status.
          - It should only take a few minutes.  Once the certificate is "Issued" we can got back to the Cloudfront tab.  Don't worry that the certificate says "Inelligible".  It'll become eligible once it's tied to a service like Cloudfront.
  - Now you can select the certificate you created under "Custom SSL certificate".  You might need to click the refresh icon if you don't see it in the list
  - Enter your "Default root object" - typically, this is "index.html"
  - Click "Create distribution"
  - On the next screen you'll see a blue banner saying the bucket policy needs to be updated.  Click "Copy policy"

6. Go to S3 and click on your bucket's name.
    - Go the the permissions tab.  Under Bucket policy, click "Edit" and paste the policy you copied in the last step
    - Click "Save changes" at the bottom right of the screen


7. Now we need to point your FQDN to the Cloudfront distribution.  Go to Route 53 and click on "Hosted Zones".  Click on your domain to see your DNS records.
    - If you have an existing record for your FQDN, you'll need to edit it.  Otherwise, click "Create record"
    - Add the host name to the domain to specify the FQDN.  Click the Alias slider and under "Route traffic to", select "Alias to Cloudfront distribution"
    - Then click on the box below that to select the Cloudfront distribution
    - Leave the other options alone and click "Create records"

Now you should be able to visit the FQDN and see the placeholder. On a couple of occasions, I've seen the "Default root object" not set once I've completed these steps.  If you have any problems, I recommend reviewing all these settings, especially that one.
