# static-site-s3-cloudfront
This repo is a template to help people set us static hosting using S3 and Cloudfront.  It also 
has a GitHub Actions workflow that will deploy any changes to S3 on a push event and then it 
will invalidate the Cloudfront cache so you can see the changes immediately.

## TODO:
- Explain how to set up S3 and Cloudfront
- Explain how to set the environment variables

## Future:
- Make a version of this template that can build Hugo files
