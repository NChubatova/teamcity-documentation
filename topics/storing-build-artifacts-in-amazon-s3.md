[//]: # (title: Storing Build Artifacts in Amazon S3)
[//]: # (auxiliary-id: Storing Build Artifacts in Amazon S3)

TeamCity comes bundled with the [Amazon S3 Artifact Storage](https://plugins.jetbrains.com/plugin/9623-aws-s3-artifact-storage) plugin which allows storing build artifacts in an Amazon S3 bucket.

It is possible to replace the TeamCity built-in artifacts' storage with [Amazon S3](https://aws.amazon.com/s3/) at a project level. When an S3 artifacts storage is configured, it:
* allows uploading to, downloading, and removing artifacts from S3;
* handles resolution of artifact dependencies as well as clean-up of artifacts;
* displays artifacts located externally in the TeamCity UI.

<include src="configuring-artifacts-storage.md" include-id="artifactMigrationToS3"/>

## Configuring Amazon S3 Artifacts Storage

To enable an external artifact storage in an AWS S3 bucket:
1. Open __Project Settings | Artifacts Storage__. The built-in TeamCity artifacts storage is displayed by default and marked as active.
2. Click __Add new storage__. Select _S3 Storage_ as the storage type.
3. Enter an optional name for your storage.
4. Select the AWS environment and provide the required settings.
5. Provide your AWS security credentials.
6. Specify an existing S3 bucket to store artifacts.
7. Save your settings.

The configured S3 storage will appear on the __Artifacts Storage__ page. To enable it in this project, change its state to _Active_.

Now, new artifacts produced by builds of this project and its subprojects will be stored in the specified AWS S3 bucket.

<anchor name="pathPrefix"/>

## Path Prefix

You can set an S3 [path prefix](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/using-folders.html). This allows using the same S3 bucket for all TeamCity projects and configure prefix-based permissions.

<anchor name="forceVirtualHostAddressing"/>

## Virtual Host Addressing

You can enable the virtual host addressing for S3 buckets. Currently, both hosted-style and path-style requests are supported by TeamCity. Note that Amazon [stopped supporting path-style access](https://docs.aws.amazon.com/AmazonS3/latest/dev/VirtualHosting.html#path-style-access) for new buckets since September 2020.

## Permissions

When the "_Use Pre-Signed URLs for upload_" option is enabled, the provided AWS credentials or IAM role on the TeamCity server should have permissions: `DeleteObject, ListAllMyBuckets, GetBucketLocation, GetObject, ListBucket, PutObject`.

When the "Use Pre-Signed URLs for upload" option is disabled:
* the provided AWS credentials or IAM role on the TeamCity server should have permissions: `DeleteObject, ListAllMyBuckets, GetBucketLocation, GetObject`
* either AWS credentials should be specified and have `ListBucket, PutObject` permissions, or IAM role on all the TeamCity agents should have permissions: `ListBucket, PutObject`

## Transferring Artifacts via CloudFront
{id="CloudFrontSettings" auxiliary-id="CloudFrontSettings"}

[Amazon CloudFront](https://aws.amazon.com/cloudfront/) is a content delivery network that offers low latency and high transfer speeds. Enabling its support for an S3 storage will allow TeamCity to transfer artifacts through the closest CloudFront server. If your S3 bucket is located in a different region than your TeamCity infrastructure, this could significantly speed up the artifacts' upload/download and reduce expenses.

>If you use [EC2 build agents](setting-up-teamcity-for-amazon-ec2.md) located in the same region as the target S3 bucket, these agents will communicate with the bucket directly, omitting CloudFront.

### Prerequisites

TeamCity can [set up CloudFront integration for you](#Automatic+CloudFront+Setup), or you can set up all the settings [manually](#Manual+CloudFront+Setup).

The CloudFront integration requires configuring:
* an [OAI](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html) user
* a [CloudFront distribution](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/distribution-working-with.html)
* a [trusted key group](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-trusted-signers.html#choosing-key-groups-or-AWS-accounts)
* an SSH-2 RSA [key pair](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-trusted-signers.html#private-content-creating-cloudfront-key-pairs) (public key + private key) in PEM format

### CloudFront Settings

To enable the CloudFront support for the current S3 bucket, activate the _Use CloudFront to transport artifacts_ option in the storage settings.

In the _Distribution_, choose an available distribution, if it has been [manually created](#Manual+CloudFront+Setup) in your Amazon profile, or click ![magic-wand.png](magic-wand.png) to _[configure settings automatically](#Automatic+CloudFront+Setup)_.

#### Automatic CloudFront Setup

TeamCity can configure the settings automatically. This involves:
* Generating a key pair and uploading a public key to CloudFront.
* Creating a new key group in CloudFront.
* Creating a new distribution with:
  * the __Use all edge locations__ price class
  * a new [Origin Access Identity](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html) that can access the current bucket
  * a single default behavior with:
    * `GET`, `HEAD`, `OPTIONS`, `PUT`, `POST`, `PATCH`, `DELETE` as allowed HTTP methods
    * a new [key group](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-trusted-signers.html#private-content-adding-trusted-signers) with the viewer access
    * a custom [cache policy](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/using-managed-cache-policies.html) that allows passing all query strings
* Adding a new policy to an S3 bucket to allow the new distribution accessing it. See the [policy example](#S3+Policy+Example).

#### Manual CloudFront Setup

To configure a distribution in CloudFront:
1. Generate a key pair in [SSH-2 RSA key format](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-trusted-signers.html#private-content-creating-cloudfront-key-pairs).
2. Upload the public key from the pair to CloudFront.
3. [Add a new key group](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-trusted-signers.html#private-content-adding-trusted-signers) in CloudFront and add the created public key to this group.
4. Create a new cache policy with __Cache key settings | Query strings__ set to _All_.
5. If you use a private bucket, create a new [OAI](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html) user.
6. [Create a distribution](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/distribution-web-creating.html) and attach your key group to it:
   * Make sure to choose the same S3 bucket as specified in TeamCity.
   * __Allowed HTTP methods__: `GET`, `HEAD`, `OPTIONS`, `PUT`, `POST`, `PATCH`, `DELETE`
   * __Restrict viewer access__: _yes_
   * __Trusted authorization type__: _trusted key groups_
   * __Cache key and origin requests__: _Cache policy and origin request policy_
   * __Cache key settings | Query strings__: All
   * For private buckets, enable the _use OAI_ option and configure OAI with the following setting:
     * __Bucket policy__: _No, I will update the bucket policy_
   * For public buckets, disable the __Block public access__ option.
7. [Add a new policy](https://docs.aws.amazon.com/AmazonS3/latest/userguide/add-bucket-policy.html) to your S3 bucket. See the [policy example](#S3+Policy+Example).

When configured, the distribution should automatically appear in the _CloudFront distribution_ drop-down menu:
1. Select the target CloudFront distribution.
2. In _CloudFront public key_, select the public key associated with this distribution.
3. In _Private SSH key_, upload the private key from the pair.
4. Save the storage settings.

#### S3 Policy Example

For accessing a private bucket with OAI:

```Plain Text
{
"Sid": "1",
"Effect": "Allow",
"Principal": {
      "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity <OAI ID"
  },
  "Action": [
      "s3:GetObject",
      "s3:PutObject",
      "s3:DeleteObject"
  ],
  "Resource": "arn:aws:s3:::<S3 bucket name/*"
}
```

For accessing a public bucket:

```Plain Text
{
"Sid": "PublicRead",
"Effect": "Allow",
"Principal": "*",
  "Action": [
        "s3:GetObject",
        "s3:GetObjectVersion"
    ],
  "Resource": "arn:aws:s3::<BUCKET_NAME>/*"
}
```

<include src="configuring-artifacts-storage.md" include-id="S3multipartUpload"/>