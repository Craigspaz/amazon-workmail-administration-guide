# Logging and monitoring in Amazon WorkMail<a name="monitoring-overview"></a>

Monitoring your email flow is important for maintaining the health of your Amazon WorkMail organization\. Monitoring the email sending activity for your organization helps protect your domain reputation\. Monitoring can also help you track emails that are sent and received\. For more information about how to enable email event logging, see [Enabling event logging](tracking.md)\.

AWS provides the following monitoring tools to watch Amazon WorkMail, report when something is wrong, and take automatic actions when appropriate:
+ *Amazon CloudWatch* monitors your AWS resources and the applications you run on AWS in real time\. For example, when you enable email event logging for Amazon WorkMail, CloudWatch can track emails sent and received for your organization\. For more information about monitoring Amazon WorkMail with CloudWatch, see [Monitoring Amazon WorkMail with Amazon CloudWatch](monitoring-workmail-cloudwatch.md)\. For more information about CloudWatch, see the [Amazon CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/)\.
+ *Amazon CloudWatch Logs* enables you to monitor, store, and access your email event logs for Amazon WorkMail when email event logging is enabled in the Amazon WorkMail console\. CloudWatch Logs can monitor information in the log files, and you can archive your log data in highly durable storage\. For more information about tracking Amazon WorkMail messages using CloudWatch Logs, see [Enabling event logging](tracking.md)\. For more information about CloudWatch Logs, see the [Amazon CloudWatch Logs User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/)\.
+ *AWS CloudTrail* captures API calls and related events made by or on behalf of your AWS account, and delivers the log files to an Amazon S3 bucket that you specify\. You can identify which users and accounts called AWS, the source IP address from which the calls were made, and when the calls occurred\. For more information, see [Logging Amazon WorkMail API calls with AWS CloudTrail](logging-using-cloudtrail.md)\.

**Topics**
+ [Monitoring Amazon WorkMail with Amazon CloudWatch](monitoring-workmail-cloudwatch.md)
+ [Logging Amazon WorkMail API calls with AWS CloudTrail](logging-using-cloudtrail.md)
+ [Enabling event logging](tracking.md)