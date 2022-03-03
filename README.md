# CloudBolt Content Library Repository Template
This template repository may be used as a starting point to implement a separate
CMP Content Library repository for your organization.  To use it, create a new
repository in your GitHub account, using this repository as the template.  Then
follow the instructions in this README file in order to get your repository up
and running.

## High Level Summary
The CloudBolt Content Library is implemented using an AWS S3 storage bucket, with
AWS CloudFront as the front end distribution facility for content in that repository.
To host a CloudBolt Content Library instance, the following are required:

- A configured, licensed, and operational CloudBolt CMP or OneFuse server.
- An AWS account with administrative rights. (API Users, Roles, S3, and CloudFront)
- A GitHub account with administrative rights to create a new repository.

When correctly configured, content checked into your GitHub repository will
trigger a GitHub workflow that assembles your content into the correct format
for the Content Library and then publishes it to the configured S3 bucket
for consumption by your CloudBolt CMP or OneFuse server(s).

## Content Library Workflow Inputs And Secrets Quick Reference
The "generate-and-publish.yml" reusable workflow requires the following secret
and inputs in order to publish updated content to S3.  This is intended as
a quick reference, as well as a preview of what needs to be done to set up
a Content Library repository "fron scratch".


### Workflow Input: publish-to
The user friendly name of the "environment" the workflow publishes to.  Examples
include "Production", "Staging", and so on.   If this field is omitted, left
blank, or has the value "none" in any case, then the workflow will skip the
"publish" step and only build and check the content.  This might be useful for
automated checks on non-production and non-staging branches wher publication
is not needed.

### Workflow Input: aws-default-region
The default region for AWS API operations.  For example, "us-east-1".

### Workflow Input: aws-account-id
The AWS account ID for the API user created to publish and update content
in the Content Library S3 bucket.  Both the secret "aws-access-key-secret"
and the input "aws-access-key-id" below are created for this API
user ID.

### Workflow Input: aws-access-key-id
The AWS API access key ID used to authenticate to AWS as the account ID
specified above as "aws-account-id".  This access key is used to manage
the Content Library S3 bucket and CloudFront distribution on behalf of
the Content Library GitHub workflows and actions.

### Workflow Secret: aws-access-key-secret
The AWS API access key secret string provided by AWS when the the above
access key was created.  This is the only parameter that specifically MUST
be stored in the repository "secrets" cache since it is an actual authentication
secret granting read and write access to the Content Library content in AWS.

### Workflow Input: aws-content-library-role
The name of the restricted role to assume, granting the above user minimal
rights to publish or update content only in the S3 buckets and CloudFront
distributions for your Content Library repositories.  See the walkthrough
below for more information on configuring this role and its associated
S3 and CloudFront access privileges.  This can be any named role created in
your AWS environment that your API use can assume, but best practice would
be to create a specific use and role to strictly limit access to only that
which is needed by the Content Library.

### Workflow Input: aws-s3-bucket
The name of the S3 bucket to use for the Content Library.

### Workflow Input: aws-s3-backup-bucket
If specified, the name of an alternate S3 bucket to use to back up old
versions of content on each publication.  TBD - Do we want to disable
backups if unspecified or blank.  Alternately we could make this input
optional, and default to backing up in the "aws-s3-bucket" above.  Then,
if "none" (any case) is specified, turn off backups.

### aws-cloudfront-distribution-id
The "distribution ID" for the CloudFront front end of your repository.  This
is used to create cache invalidations to ensure new or updatd content becomes
available to consumers immediately.

## Setting Up A Content Library Repository From Scratch (Step by Step)

### Step 1 - Set up CloudBolt CMP.
A CloudBolt CMP server will be needed to access the new Content Library.
See the CloudBolt CMP documentation for instructions on getting it set
up, licensed, and running.

### Step 2 - AWS and GitHub access.
Create or obtain access to an AWS admin/root account and a GitHub account with
the ability to create a new repository.  The AWS account should have permission
to crate and manage S3 buckets, CloudFront distributions, and API users.
The process of obtaining or setting up AWS and GitHub access is beyond the
scope of this README.  Contact your system administrator, if needed.

The Content Library workflows will need the following:
- Input "aws-account-id": The AWS account ID. (From your AWS account)
- Input "aws-default-region": The AWS region. (e.g. "us-east-1")

### Step 3 - Create the primary S3 bucket
The Content Library uses S3 buckets to store its data, with CloudFront as
a content distribution front end.  Also, an optional "backup" S3 bucket can
be created separately to allow the Content Library workflows to automatically
back up previous versions of content on update.  The "backup" bucket does
not need a CloudFront distribution.

The Content Library workflows will need the following:
- Input "aws-s3-bucket": For this demo, "content-library-demo".

Follow these steps as an example:
- Log into the AWS management console.
- Use the "search" box next to the "aws" and "Services" button in the upper left corner.
- Search for "S3" and click on S3 to open it.
- Note: S3 may also already be in your favorites or recently used list.
- In the left side bar, make sure "Buckets" is selected.
- In the "Buckets" panel, click the "Create bucket" button.
- Select a name for the bucket.
- For this example, we will name the bucket "content-library-demo".
- Select an AWS region for the bucket.
- For this example, we will use "us-east-1".
- Under "Object Ownership" make sure "ACLs disable" is checked.
- Under "Block Public Access settings..." make sure "Block all public access" is checked.
- Under "Bucket Versioning" make sure "Disable" is checked.
- Under "Tags", there do not need to be any tags.
- Under "Default encryption" make sure "Enable" is checked.
- Check encryption key type "Amazon S3-managed keys (SSE-S3)"
- Under "Advanced settings" check "Disable" for "Object Lock".
- Click "Create Bucket"

### Step 4 - Create the backup S3 bucket (Optional)
The backup bucket can be the same bucket as the main Content Library bucket.
For this example, we will use a separate backup bucket.
TBD - Do we want to allow "none" (see above) to mean "don't backup"?

The Content Library workflows will need the following:
- Input "aws-s3-backup-bucket": For this demo, "content-library-demo-backup".

Follow the instructions from Step 3 above using the backup bucket name
instead of the main bucket name.  All other S3 bucket options should be the same.

### Step 5 - Create a CloudFront Distribution

The Content Library workflows will need the following:
- Input "aws-cloudfront-distribution-id": The CloudFront distribution ID noted below.

To use the new content Library, CMS will need:
- The origin domain obtained below.

Follow these steps as an example:
- Log into the AWS management console.
- Use the "search" box next to the "aws" and "Services" button in the upper left corner.
- Search for "CloudFront" and click on CloudFront to open it.
- Note: CloudFront may also already be in your favorites or recently used list.
- In the left side bar, make sure "Distributions" is selected.
- In the "Distributions" panel, click the "Create distribution" button.
- Click on the "Origin domain" search box.  This will open a pop up list.
- Select the domain containing the Bucket name (e.g. "content-library-demo") from above.
- The selected domain will look similar to "content-library-demo.s3.us-east-1.amazonaws.com".
- Leave "Origin path" blank.
- The "Name" field should already be the same as "Origin domain".  Leave it as is.
- Under "S3 bucket access", check "Yes use OAI...".
- Click the "Create new OAI" button.  Accept the defaut name.
- OAI name will look like "access-identity-content-library-demo.s3.us-east-1.amazonaws.com".
- Under Bucket policy, check "No, I will update the bucket policy".
- No custom headers need to be added.
- Under "Enable Origin Shield" check "No".
- No changes are needed to "Additional settings".
- Leave all other settings on defaults.
- Click "Create Distribution".
- Note the distribution ID for workflow input "aws-cloudfront-distribution-id".

### Step 6 - Add AWS API User

The Content Library GitHub workflows require an AWS user identity with a minimal
set of permissions restricted to your S3 bucket and CloudFront distribution in order
to be able to publish content.  The example steps below set up a minimal per-bucket
user for this purpose.  The resulting IAM user can be additionally configured according
to site needs and policies as long as it has the minimum access to S3 and CloudFront.

Note: Don't forget to copy the access key secret somewhere after the user creation
succeeds, since AWS will never display it again after leaving that page.  If you
forget to do this, however, you can always generate a new access key and secret
for the user.

The workflow Secret "aws-access-key-secret" is the one piece of information that
must be stored in your GitHub repository's "secrets" cache. (See below)

The Content Library workflows will need the following:
- Input "aws-access-key-id": The AWS access key ID for the user created here.
- Secret "aws-access-key-secret": The secret for the above access key.

Follow these steps as an example:
- Log into the AWS management console.
- Use the "search" box next to the "aws" and "Services" button in the upper left corner.
- Search for "IAM" and click on "IAM" to open it.
- Note: IAM may also already be in your favorites or recently used list.
- In the left side bar, make sure "Users" is selected.
- Click the "Add new users" button.
- Select a user name.  For this demo, we will use "ContentLibraryDemo".
- Under "Select AWS credential type" check "Access key..."
- Leave "Password..." unchecked -- User does not require console or UI access.
- Click the "Next: Permissions" button at the bottom of the page.
- Nothing needs to be done for "Set permissions" or "Set permission boundary".
- Click the "Next: Tags" button at the bottom of the page.
- No tags are needed.
- Click the "Next: Review" button at the bottom of the page.
- A warning on the page will indicate the user has no permissions.  This is expected.
- Permissions will be added later.
- Click the "Create user" button at the bottom of the page.
- Don't forget to save the resulting "Access key ID" and "Secret access key" values.

### Step 7 - Create AWS Policy for S3 and CloudFront Permissions

Follow these steps as an example:
- Log into the AWS management console.
- Use the "search" box next to the "aws" and "Services" button in the upper left corner.
- Search for "IAM" and click on "IAM" to open it.
- Note: IAM may also already be in your favorites or recently used list.
- In the left side bar, make sure "Policies" is selected.
- Click the "Create Policy" button.
- Within the "Visual Editor" tab, expand "Service"
- Inside the "Select a service" pane, click "Choose a service".
- Enter "S3" in the "Find a service" search box.
- Click "S3"
- Scroll down the page a bit.
- Within the "Visual Editor" tab, expand "Actions".  This will unexpand "Service".
- Leave "Filter actions" blank.
- Leave "All S3 actions" unchecked.
- Expand "List" and check ONLY "ListBucket".
- Expand "Read" and check ONLY "GetBucketLocation" and "GetObject".
- Expand "Write" and check ONLY "DeleteObject" and "PutObject".
- Scroll down the page a bit.
- Within the "Visual Editor" tab, expand "Resources".  This will unexpand "Actions".
- Make sure "Specific" is checked and "All resources" is unchecked. 
- Under "bucket", enter an ARN like "arn:aws:s3:::ContentLibraryDemo".
- For the above step, change "ContentLibraryDemo" to your bucket name if different.
- Under "object", enter an ARN like "arn:aws:s3:::ContentLibraryDemo/\*".
- Again, for the above step, change "ContentLibraryDemo" if different.
- Scroll down the page a bit.
- Expand "Request conditions".
- Leave "MFA required" and "Source IP" unchecked.
- This completes permissions for S3.  We must also add one permission for CloudFront.
- Within the "Visual Editor" pane, unexpand "S3" to free up room on the page.
- At the bottom of the visual editor pane, click "Add additional permissions".
- In the newly created permission box, click "Choose a service".
- Enter "CloudFront" in the text search box.
- Click CloudFront.  This will rename the permission box to "CloudFront".
- The "Actions" section should already be expanded.
- Leave "Filter actions" blank.
- Leave "All CloudFront actions" unchecked.
- Under "Access level", Expand "Write" and check ONLY "CreateInvalidation".
- Scroll down the page a bit.
- Within the "CloudFront" permission box, expand "Resources".  This will unexpand "Actions".
- Make sure "Specific" is checked and "All resources" is unchecked. 
- Under "distribution", click "Add ARN".
- This will cause a popup dialog to appear.
- Enter the distribution ID for the CloudFront distribution created earlier.
- Click "Add" to complete the popup dialog.
- Scroll down the page a bit.
- Expand "Request conditions".
- Leave "MFA required" and "Source IP" unchecked.
- This completes permissions for CloudFront.  We must also add one permission for CloudFront.
- At the bottom of the page, click the "Next: Tags" button.
- No tags are needed.
- At the bottom of the page, click the "Next: Review" button.
- Fill in a name for the policy.  For this demo we use "ContentLibraryDemoPolicy".
- Fill in the optional description, if desired.
- At the bottom of the page, click the "Create policy" button.

### Step 8 - Create Role Mapping Policy to API User.

Need a custom trust policy:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:user/ContentLibraryDemo"
            },
            "Action": "sts:AssumeRole",
            "Condition": {}
        }
    ]
}
```

### Step 9 - Create GitHub Content Library Repository.

### Step 10 - Create Workflow(s).

### Step 11 - Test Content Creations.



