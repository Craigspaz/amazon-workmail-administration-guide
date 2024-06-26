# Working with access control rules<a name="access-rules"></a>

Access control rules for Amazon WorkMail allow administrators to control how their organization's users and impersonation roles are granted access to Amazon WorkMail\. Each Amazon WorkMail organization has a default access control rule that grants mailbox access to all users and impersonation roles added to the organization, no matter which access protocol or IP address they use\. Administrators can edit or replace the default rule with one of their own, add a new rule, or delete a rule\.

**Warning**  
If an administrator deletes all access control rules for an organization, Amazon WorkMail blocks all access to the organization's mailboxes\.

Administrators can apply access control rules that allow or deny access based on the following criteria:
+ **Protocols** – The protocol used to access the mailbox\. Examples include **Autodiscover**, **EWS**, **IMAP**, **SMTP**, **ActiveSync**, **Outlook for Windows**, and **Webmail**\. 
+ **IP addresses** – The IPv4 CIDR ranges used to access the mailbox\.
+ **Amazon WorkMail users** – The users in your organization that are used to access the mailbox\.
+ **Impersonation roles** – The impersonation roles in your organization that are used to access the mailbox\. For more information, see [Managing impersonation roles](managing-impersonation-roles.md)\.

Administrators apply access control rules in addition to the user's mailbox and folder permissions\. For more information, see [Working with mailbox permissions](mail_perms_overview.md) and [Sharing folders and folder permissions](https://docs.aws.amazon.com/workmail/latest/userguide/share-folders.html) in the *Amazon WorkMail User Guide*\.

**Note**  
When you are enabling access for Outlook for Windows, it is recommended to also enable access for Autodiscover and EWS\.
Access control rules do not apply to Amazon WorkMail console or SDK access\. Use AWS Identity and Access Management \(IAM\) roles or policies instead\. For more information, see [Identity and access management for Amazon WorkMail](security-iam.md)\.

## Creating access control rules<a name="create-acr"></a>

Create new access control rules from the Amazon WorkMail console\.

**To create a new access control rule**

1. Open the Amazon WorkMail console at [https://console\.aws\.amazon\.com/workmail/](https://console.aws.amazon.com/workmail/)\.

   If necessary, change the AWS Region\. In the bar at the top of the console window, open the **Select a Region** list and choose a Region\. For more information, see [Regions and endpoints](http://docs.aws.amazon.com/general/latest/gr/index.html?rande.html) in the *Amazon Web Services General Reference*\.

1. In the navigation pane, choose **Organizations**, and then choose the name of your organization\.

1. Choose **Access control rules**\.

1. Choose **Create rule**\.

1. For **Description**, enter a description for the rule\.

1. For **Effect**, choose **Allow** or **Deny**\. This allows or denies access based on the conditions that you select in the following step\.

1. For **This rule applies to requests that \.\.\.**, select the conditions to apply to the rule, such as whether to include or exclude specific protocols, IP addresses, or users, or impersonation roles\.

1. \(Optional\) If you enter IP address ranges, users, or impersonation roles, choose **Add** to add them to the rule\.

1. Choose **Create rule**\.

## Editing access control rules<a name="edit-acr"></a>

Edit new and default access control rules from the Amazon WorkMail console\.

**To edit an access control rule**

1. Open the Amazon WorkMail console at [https://console\.aws\.amazon\.com/workmail/](https://console.aws.amazon.com/workmail/)\.

   If necessary, change the AWS Region\. In the bar at the top of the console window, open the **Select a Region** list and choose a Region\. For more information, see [Regions and endpoints](http://docs.aws.amazon.com/general/latest/gr/index.html?rande.html) in the *Amazon Web Services General Reference*\.

1. In the navigation pane, choose **Organizations**, and then choose the name of your organization\.

1. Choose **Access control rules**\.

1. Select the rule to edit\.

1. Choose **Edit rule**\.

1. Edit the description, effect, and conditions, as needed\.

1. Choose **Save changes**\.

**Important**  
When you change an access rule, the affected mailboxes can take five minutes to follow the updated rule\. Clients that access the affected mailboxes may show inconsistent behavior during that time\. However, you will immediately see correct behavior when you test your rules\. For more information about testing rules, see the steps in the next section\.

## Testing access control rules<a name="test-acr"></a>

To see how your organization's access control rules are applied, test the rules from the Amazon WorkMail console\.

**To test access control rules for your organization**

1. Open the Amazon WorkMail console at [https://console\.aws\.amazon\.com/workmail/](https://console.aws.amazon.com/workmail/)\.

   If necessary, change the AWS Region\. In the bar at the top of the console window, open the **Select a Region** list and choose a Region\. For more information, see [Regions and endpoints](http://docs.aws.amazon.com/general/latest/gr/index.html?rande.html) in the *Amazon Web Services General Reference*\.

1. In the navigation pane, choose **Organizations**, and then choose the name of your organization\.

1. Choose **Access control rules**\.

1. Choose **Test rules**\.

1. For **Request context**, select the protocol to test for\.

1. For **Source IP address**, enter the IP address to test for\.

1. For **Request performed by**, choose **User** or **Impersonation role** to test for\.

1. Select **User** or **Impersonation role** to test for\.

1. Choose **Test**\.

The test results appear under **Effect**\.

## Deleting access control rules<a name="delete-acr"></a>

Delete access control rules that you no longer require from the Amazon WorkMail console\.

**Warning**  
If an administrator deletes all access control rules for an organization, Amazon WorkMail blocks all access to the organization's mailboxes\.

**To delete an access control rule**

1. Open the Amazon WorkMail console at [https://console\.aws\.amazon\.com/workmail/](https://console.aws.amazon.com/workmail/)\.

   If necessary, change the AWS Region\. In the bar at the top of the console window, open the **Select a Region** list and choose a Region\. For more information, see [Regions and endpoints](http://docs.aws.amazon.com/general/latest/gr/index.html?rande.html) in the *Amazon Web Services General Reference*\.

1. In the navigation pane, choose **Organizations**, and then choose the name of your organization\.

1. Choose **Access control rules**\.

1. Select the rule to delete\.

1. Choose **Delete rule**\.

1. Choose **Delete**\.