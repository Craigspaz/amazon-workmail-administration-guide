# Data protection in Amazon WorkMail<a name="data-protection"></a>

The AWS [shared responsibility model](http://aws.amazon.com/compliance/shared-responsibility-model/) applies to data protection in Amazon WorkMail\. As described in this model, AWS is responsible for protecting the global infrastructure that runs all of the AWS Cloud\. You are responsible for maintaining control over your content that is hosted on this infrastructure\. This content includes the security configuration and management tasks for the AWS services that you use\. For more information about data privacy, see the [Data Privacy FAQ](http://aws.amazon.com/compliance/data-privacy-faq)\. For information about data protection in Europe, see the [AWS Shared Responsibility Model and GDPR](http://aws.amazon.com/blogs/security/the-aws-shared-responsibility-model-and-gdpr/) blog post on the *AWS Security Blog*\.

For data protection purposes, we recommend that you protect AWS account credentials and set up individual users with AWS IAM Identity Center \(successor to AWS Single Sign\-On\) or AWS Identity and Access Management \(IAM\)\. That way, each user is given only the permissions necessary to fulfill their job duties\. We also recommend that you secure your data in the following ways:
+ Use multi\-factor authentication \(MFA\) with each account\.
+ Use SSL/TLS to communicate with AWS resources\. We recommend TLS 1\.2 or later\.
+ Set up API and user activity logging with AWS CloudTrail\.
+ Use AWS encryption solutions, along with all default security controls within AWS services\.
+ Use advanced managed security services such as Amazon Macie, which assists in discovering and securing sensitive data that is stored in Amazon S3\.
+ If you require FIPS 140\-2 validated cryptographic modules when accessing AWS through a command line interface or an API, use a FIPS endpoint\. For more information about the available FIPS endpoints, see [Federal Information Processing Standard \(FIPS\) 140\-2](http://aws.amazon.com/compliance/fips/)\.

We strongly recommend that you never put confidential or sensitive information, such as your customers' email addresses, into tags or free\-form text fields such as a **Name** field\. This includes when you work with Amazon WorkMail or other AWS services using the console, API, AWS CLI, or AWS SDKs\. Any data that you enter into tags or free\-form text fields used for names may be used for billing or diagnostic logs\. If you provide a URL to an external server, we strongly recommend that you do not include credentials information in the URL to validate your request to that server\.

## How Amazon WorkMail uses AWS KMS<a name="workmail-kms"></a>

Amazon WorkMail transparently encrypts all messages in the mailboxes of all Amazon WorkMail organizations before the messages are written to disk, and it transparently decrypts the messages when users access them\. You can't disable encryption\. To protect the encryption keys that protect the messages, Amazon WorkMail is integrated with AWS Key Management Service \(AWS KMS\)\.

Amazon WorkMail also provides an option for enabling users to send signed or encrypted email\. This encryption feature does not use AWS KMS\. For more information, see [Enabling signed or encrypted email](enable_encryption.md)\.

**Topics**
+ [Amazon WorkMail encryption](#workmail-kms-encryption)
+ [Authorizing use of the CMK](#workmail-kms-authorizing-cmk)
+ [Amazon WorkMail encryption context](#workmail-encryption-context)
+ [Monitoring Amazon WorkMail interaction with AWS KMS](#workmail-kms-cloudtrail-logs)

### Amazon WorkMail encryption<a name="workmail-kms-encryption"></a>

In Amazon WorkMail, each organization can contain multiple mailboxes, one for each user in the organization\. All messages, including email and calendar items, are stored in the user's mailbox\.

To protect the contents of the mailboxes in your Amazon WorkMail organizations, Amazon WorkMail encrypts all mailbox messages before they are written to disk\. No customer\-provided information is stored in plaintext\.

Each message is encrypted under a unique data encryption key\. The message key is protected by a mailbox key, which is a unique encryption key that is used only for that mailbox\. The mailbox key is encrypted under an AWS KMS customer master key \(CMK\) for the organization that never leaves AWS KMS unencrypted\. The following diagram shows the relationship of the encrypted messages, encrypted message keys, encrypted mailbox key, and the CMK for the organization in AWS KMS\.

![\[Encrypting your Amazon WorkMail mailboxes\]](http://docs.aws.amazon.com/workmail/latest/adminguide/images/service-workmail.png)

#### Setting a CMK for the organization<a name="workmail-cmk"></a>

When you create an Amazon WorkMail organization, you have the option to select an AWS KMS customer master key \(CMK\) for the organization\. This CMK protects all mailbox keys in that organization\.

You can select the default AWS managed CMK for Amazon WorkMail, or you can select an existing customer managed CMK that you own and manage\. For more information, see [customer master keys \(CMKs\)](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) in the *AWS Key Management Service Developer Guide*\. You can select the same CMK or a different CMK for each of your organizations, but you cannot change the CMK once you select it\.

**Important**  
Amazon WorkMail supports only symmetric CMKs\. You cannot use an asymmetric CMK\. For help determining whether a CMK is symmetric or asymmetric, see [Identifying symmetric and asymmetric CMKs](https://docs.aws.amazon.com/kms/latest/developerguide/find-symm-asymm.html) in the *AWS Key Management Service Developer Guide*\.

To find the CMK for your organization, use the AWS CloudTrail log entry that records calls to AWS KMS\.

#### A unique encryption key for each mailbox<a name="workmail-mailbox-kms-key"></a>

When you create a mailbox, Amazon WorkMail generates a unique 256\-bit [Advanced Encryption Standard](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) \(AES\) symmetric encryption key for the mailbox, known as its *mailbox key*, outside of AWS KMS\. Amazon WorkMail uses the mailbox key to protect the encryption keys for each message in the mailbox\.

To protect the mailbox key, Amazon WorkMail calls AWS KMS to encrypt the mailbox key under the CMK for the organization\. Then it stores the encrypted mailbox key in the mailbox metadata\. 

**Note**  
Amazon WorkMail uses a symmetric mailbox encryption key to protect message keys\. Previously, Amazon WorkMail protected each mailbox with an asymmetric key pair\. It used the public key to encrypt each message key and the private key to decrypt it\. The private mailbox key was protected by the CMK for the organization\. Older mailboxes may use an asymmetric mailbox key pair\. This change does not affect the security of the mailbox or its messages\.

#### Encrypting each message<a name="workmail-message-kms-key"></a>

When a user adds a message to a mailbox, Amazon WorkMail generates a unique 256\-bit AES symmetric encryption key for the message outside of AWS KMS\. It uses this *message key* to encrypt the message\. Amazon WorkMail encrypts the message key under the mailbox key and stores the encrypted message key with the message\. Then, it encrypts the mailbox key under the CMK for the organization\.

#### Creating a new mailbox<a name="workmail-kms-create-mailbox"></a>

When Amazon WorkMail creates a mailbox, it uses the following process to prepare the mailbox to hold encrypted messages\.
+ Amazon WorkMail generates a unique 256\-bit AES symmetric encryption key for the mailbox outside of AWS KMS\.
+ Amazon WorkMail calls the AWS KMS [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) operation\. It passes in the mailbox key and the identifier of the customer master key \(CMK\) for the organization\. AWS KMS returns a ciphertext of the mailbox key encrypted under the CMK\.
+ Amazon WorkMail stores the encrypted mailbox key with the mailbox metadata\.

#### Encrypting a mailbox message<a name="workmail-kms-message-encrypt"></a>

To encrypt a message, Amazon WorkMail uses the following process\.

1. Amazon WorkMail generates a unique 256\-bit AES symmetric key for the message\. It uses the plaintext message key and the Advanced Encryption Standard \(AES\) algorithm to encrypt the message outside of AWS KMS\. 

1. To protect the message key under the mailbox key, Amazon WorkMail needs to decrypt the mailbox key, which is always stored in its encrypted form\.

   Amazon WorkMail calls the AWS KMS [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation and passes in the encrypted mailbox key\. AWS KMS uses the CMK for the organization to decrypt the mailbox key and it returns the plaintext mailbox key to Amazon WorkMail\.

1. Amazon WorkMail uses the plaintext mailbox key and the Advanced Encryption Standard \(AES\) algorithm to encrypt the message key outside of AWS KMS\.

1. Amazon WorkMail stores the encrypted message key in the metadata of the encrypted message so it is available to decrypt it\. 

#### Decrypting a mailbox message<a name="workmail-kms-decrypt"></a>

To decrypt a message, Amazon WorkMail uses the following process\.

1. Amazon WorkMail calls the AWS KMS [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation and passes in the encrypted mailbox key\. AWS KMS uses the CMK for the organization to decrypt the mailbox key and it returns the plaintext mailbox key to Amazon WorkMail\.

1. Amazon WorkMail uses the plaintext mailbox key and the Advanced Encryption Standard \(AES\) algorithm to decrypt the encrypted message key outside of AWS KMS\. 

1. Amazon WorkMail uses the plaintext message key to decrypt the encrypted message\.

#### Caching mailbox keys<a name="workmail-kms-mailbox-key-cache"></a>

To improve performance and minimize calls to AWS KMS, Amazon WorkMail caches each plaintext mailbox key for each client locally for up to one minute\. At the end of the caching period, the mailbox key is removed\. If the mailbox key for that client is required during the caching period, Amazon WorkMail can get it from the cache instead of calling AWS KMS\. The mailbox key is protected in the cache and is never written to disk in plaintext\.

### Authorizing use of the CMK<a name="workmail-kms-authorizing-cmk"></a>

When Amazon WorkMail uses a customer master key \(CMK\) in cryptographic operations, it acts on behalf of the mailbox administrator\.

To use the AWS KMS customer master key \(CMK\) for a secret on your behalf, the administrator must have the following permissions\. You can specify these required permissions in an IAM policy or key policy\.
+ `kms:Encrypt`
+ `kms:Decrypt`
+ `kms:CreateGrant`

To allow the CMK to be used only for requests that originate in Amazon WorkMail, you can use the [kms:ViaService](https://docs.aws.amazon.com/kms/latest/developerguide/policy-conditions.html#conditions-kms-via-service) condition key with the `workmail.<region>.amazonaws.com` value\.

You can also use the keys or values in the [encryption context](#workmail-encryption-context) as a condition for using the CMK for cryptographic operations\. For example, you can use a string condition operator in an IAM or key policy document or use a grant constraint in a grant\.

**Key policy for the AWS managed CMK**

The key policy for the AWS managed CMK for Amazon WorkMail gives users permission to use the CMK for specified operations only when Amazon WorkMail makes the request on the user's behalf\. The key policy does not allow any user to use the CMK directly\.

 This key policy, like the policies of all [AWS managed keys](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys), is established by the service\. You can't change the key policy, but you can view it at any time\. For details, see [Viewing a key policy](https://docs.aws.amazon.com/kms/latest/developerguide/key-policy-viewing.html) in the *AWS Key Management Service Developer Guide*\.

The policy statements in the key policy have the following effect:
+ Allow users in the account and Region to use the CMK for cryptographic operations and to create grants, but only when the request comes from Amazon WorkMail on their behalf\. The `kms:ViaService` condition key enforces this restriction\.
+ Allows the AWS account to create IAM policies that allow users to view CMK properties and revoke grants\.

The following is a key policy for an example AWS managed CMK for Amazon WorkMail\.

```
{
  "Version" : "2012-10-17",
  "Id" : "auto-workmail-1",
  "Statement" : [ {
    "Sid" : "Allow access through WorkMail for all principals in the account that are authorized to use WorkMail",
    "Effect" : "Allow",
    "Principal" : {
      "AWS" : "*"
    },
    "Action" : [ "kms:Decrypt", "kms:CreateGrant", "kms:ReEncrypt*", "kms:DescribeKey", "kms:Encrypt" ],
    "Resource" : "*",
    "Condition" : {
      "StringEquals" : {
        "kms:ViaService" : "workmail.us-east-1.amazonaws.com",
        "kms:CallerAccount" : "111122223333"
      }
    }
  }, {
    "Sid" : "Allow direct access to key metadata to the account",
    "Effect" : "Allow",
    "Principal" : {
      "AWS" : "arn:aws:iam::111122223333:root"
    },
    "Action" : [ "kms:Describe*", "kms:List*", "kms:Get*", "kms:RevokeGrant" ],
    "Resource" : "*"
  } ]
}
```

**Using grants to authorize Amazon WorkMail**

In addition to key policies, Amazon WorkMail uses grants to add permissions to the CMK for each organization\. To view the grants on the CMK in your account, use the [ListGrants](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListGrants.html) operation\.

Amazon WorkMail uses grants to add the following permissions to the CMK for the organization\.
+ Add the `kms:Encrypt` permission to allow Amazon WorkMail to encrypt the mailbox key\.
+ Add the `kms:Decrypt` permission to allow Amazon WorkMail to use the CMK to decrypt the mailbox key\. Amazon WorkMail requires this permission in a grant because the request to read mailbox messages uses the security context of the user who is reading the message\. The request does not use the credentials of the AWS account\. Amazon WorkMail creates this grant when you select a CMK for the organization\.

To create the grants, Amazon WorkMail calls [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) on behalf of the user who created the organization\. Permission to create the grant comes from the key policy\. This policy allows account users to call `CreateGrant` on the CMK for the organization when Amazon WorkMail makes the request on an authorized user's behalf\.

The key policy also allows the account root to revoke the grant on the AWS managed key\. However, if you revoke the grant, Amazon WorkMail can't decrypt the encrypted data in your mailboxes\.

### Amazon WorkMail encryption context<a name="workmail-encryption-context"></a>

An encryption context is a set of key\-value pairs that contain arbitrary nonsecret data\. When you include an encryption context in a request to encrypt data, AWS KMS cryptographically binds the encryption context to the encrypted data\. To decrypt the data, you must pass in the same encryption context\. For more information, see [Encryption context](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#encrypt_context) in the *AWS Key Management Service Developer Guide*\.

Amazon WorkMail uses the same encryption context format in all AWS KMS cryptographic operations\. You can use the encryption context to identify a cryptographic operation in audit records and logs, such as [AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html), and as a condition for authorization in policies and grants\.

In its [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) and [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) requests to AWS KMS, Amazon WorkMail uses an encryption context where the key is `aws:workmail:arn` and the value is the Amazon Resource Name \(ARN\) of the organization\. 

```
"aws:workmail:arn":"arn:aws:workmail:region:account ID:organization/organization-ID"
```

For example, the following encryption context includes an example organization ARN in the Europe \(Ireland\) \(`eu-west-1`\) Region\.

```
"aws:workmail:arn":"arn:aws:workmail:eu-west-1:111122223333:organization/m-a123b4c5de678fg9h0ij1k2lm234no56"
```

### Monitoring Amazon WorkMail interaction with AWS KMS<a name="workmail-kms-cloudtrail-logs"></a>

You can use AWS CloudTrail and Amazon CloudWatch Logs to track the requests that Amazon WorkMail sends to AWS KMS on your behalf\.

#### Encrypt<a name="workmail-kms-cloudtrail-encrypt"></a>

When you create a mailbox, Amazon WorkMail generates a mailbox key and calls AWS KMS to encrypt the mailbox key\. Amazon WorkMail sends an [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) request to AWS KMS with the plaintext mailbox key and an identifier for the CMK of the Amazon WorkMail organization\.

The event that records the `Encrypt` operation is similar to the following example event\. The user is the Amazon WorkMail service\. The parameters include the CMK ID \(`keyId`\) and the encryption context for the Amazon WorkMail organization\. Amazon WorkMail also passes in the mailbox key, but that is not recorded in the CloudTrail log\.

```
{
    "eventVersion": "1.05",
    "userIdentity": {
        "type": "AWSService",
        "invokedBy": "workmail.eu-west-1.amazonaws.com"
    },
    "eventTime": "2019-02-19T10:01:09Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "Encrypt",
    "awsRegion": "eu-west-1",
    "sourceIPAddress": "workmail.eu-west-1.amazonaws.com",
    "userAgent": "workmail.eu-west-1.amazonaws.com",
    "requestParameters": {
        "encryptionContext": {
            "aws:workmail:arn": "arn:aws:workmail:eu-west-1:111122223333:organization/m-a123b4c5de678fg9h0ij1k2lm234no56"
        },
        "keyId": "arn:aws:kms:eu-west-1:111122223333:key/1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d"
    },
    "responseElements": null,
    "requestID": "76e96b96-7e24-4faf-a2d6-08ded2eaf63c",
    "eventID": "d5a59c18-128a-4082-aa5b-729f7734626a",
    "readOnly": true,
    "resources": [
        {
            "ARN": "arn:aws:kms:eu-west-1:111122223333:key/1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d",
            "accountId": "111122223333",
            "type": "AWS::KMS::Key"
        }
    ],
    "eventType": "AwsApiCall",
    "recipientAccountId": "111122223333",
    "sharedEventID": "d08e60f1-097e-4a00-b7e9-10bc3872d50c"
}
```

#### Decrypt<a name="workmail-kms-cloudtrail-decrypt"></a>

When you add, view, or delete a mailbox message, Amazon WorkMail asks AWS KMS to decrypt the mailbox key\. Amazon WorkMail sends a [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) request to AWS KMS with the encrypted mailbox key and an identifier for the CMK of the Amazon WorkMail organization\.

The event that records the `Decrypt` operation is similar to the following example event\. The user is the Amazon WorkMail service\. The parameters include the encrypted mailbox key \(as a ciphertext blob\), which is not recorded in the log, and the encryption context for the Amazon WorkMail organization\. AWS KMS derives the ID of the CMK from the ciphertext\.

```
{
    "eventVersion": "1.05",
    "userIdentity": {
        "type": "AWSService",
        "invokedBy": "workmail.eu-west-1.amazonaws.com"
    },
    "eventTime": "2019-02-20T11:51:10Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "Decrypt",
    "awsRegion": "eu-west-1",
    "sourceIPAddress": "workmail.eu-west-1.amazonaws.com",
    "userAgent": "workmail.eu-west-1.amazonaws.com",
    "requestParameters": {
        "encryptionContext": {
            "aws:workmail:arn": "arn:aws:workmail:eu-west-1:111122223333:organization/m-a123b4c5de678fg9h0ij1k2lm234no56"
        }
    },
    "responseElements": null,
    "requestID": "4a32dda1-34d9-4100-9718-674b8e0782c9",
    "eventID": "ea9fd966-98e9-4b7b-b377-6e5a397a71de",
    "readOnly": true,
    "resources": [
        {
            "ARN": "arn:aws:kms:eu-west-1:111122223333:key/1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d",
            "accountId": "111122223333",
            "type": "AWS::KMS::Key"
        }
    ],
    "eventType": "AwsApiCall",
    "recipientAccountId": "111122223333",
    "sharedEventID": "241e1e5b-ff64-427a-a5b3-7949164d0214"
}
```