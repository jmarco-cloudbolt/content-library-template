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

