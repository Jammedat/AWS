The Nautilus DevOps team has been tasked with creating an internal information portal for public access. As part of this project, they need to host a static website on AWS using an S3 bucket. The S3 bucket must be configured for public access to allow external users to access the static website directly via the S3 website URL.

Task Requirements:

Create an S3 bucket named devops-web-1903.
Configure the S3 bucket for static website hosting with index.html as the index document.
Allow public access to the bucket so that the website is publicly accessible.
Upload the index.html file from the /root/ directory of the AWS client host to the S3 bucket.
Verify that the website is accessible directly through the S3 website URL.


### Solution
1. Create a bucket

2. To configure the s3 bucket for static website hosting go to properties section of the newly created bucket and enable the S3 static website hosting in Static Website Hosting section.

3. Add the following policy in permission section of the bucket:
```bash

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::devops-web-1903/*"
    }
  ]
}
```

4. Upload the file as :  
` aws s3 cp /root/index.html s3://devops-web-1903 `

5. Verify by visiting the bucket website endpoint which is presented at the end of the properties section of the bucket.