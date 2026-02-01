# Install & Configure AWS

```bash
aws configure
```

# Setup S3

```bash
# Create bucket
aws s3api create-bucket 
    --bucket my-unique-angular-bucket 
    --region region-name 
    --create-bucket-configuration LocationConstraint=region-name

# Disable Public Access
aws s3api put-public-access-block \
    --bucket my-unique-angular-bucket \
    --public-access-block-configuration "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"    
```

# Setup Cloudfront

```bash
# Create the Origin Access Control (OAC) so that Cloudfront talks to S3
# Copy the Id (OAC_ID) from the output JSON
aws cloudfront create-origin-access-control \
    --origin-access-control-config Name="MyAngularOAC",Description="Access for Angular S3",SigningProtocol="sigv4",SigningBehavior="always",OriginAccessControlOriginType="s3"

# Create Cloudfront distribution
aws cloudfront create-distribution \
    --origin-domain-name my-unique-angular-bucket.s3.amazonaws.com \
    --default-root-object index.html \
    --origin-access-control-id <OAC_ID>    
```

To make S3 allow Cloudfront, create policy.json

```bash
# Apply the policy
aws s3api put-bucket-policy --bucket my-unique-angular-bucket --policy file://policy.json
```

Configure Cloudfront error pages

```bash
HTTP Error code: 403: Forbidden
Customize error response: Yes
Response page path: /index.html
HTTP Response code: 200: OK

HTTP Error code: 404: Not Found
Customize error response: Yes
Response page path: /index.html
HTTP Response code: 200: OK
```

Now build your angular application, upload files to s3 using:

```bash
# From root of the project (If used Create React App)
aws s3 sync build s3://my-unique-angular-bucket --delete

# From root of the project (If used Vite)
aws s3 sync dist s3://my-unique-angular-bucket --delete
```

```bash
# Invalidate cloudfront
aws cloudfront create-invalidation --distribution-id <YOUR_DISTRIBUTION_ID> --paths "/*"
```

