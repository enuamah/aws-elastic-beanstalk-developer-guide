# Elastic Beanstalk Environment Notifications with Amazon SNS<a name="using-features.managing.sns"></a>

You can configure your AWS Elastic Beanstalk environment to use Amazon Simple Notification Service \(Amazon SNS\) to notify you of important events that affect your application\. Specify an email address during or after environment creation to receive emails from AWS when an error occurs or when your environment's health changes\.

**Note**  
Elastic Beanstalk uses Amazon SNS for notifications\. For details about Amazon SNS pricing, see [https://aws\.amazon\.com/sns/pricing/](https://aws.amazon.com/sns/pricing/)\.

When you configure notifications for your environment, Elastic Beanstalk creates an Amazon SNS topic for your environment\. To send messages to an Amazon SNS topic, Elastic Beanstalk must have the required permissions\. Elastic Beanstalk automatically attaches the required permissions to the topic's policy\.

When a notable [event](using-features.events.md) occurs, Elastic Beanstalk sends a message to the topic\. Amazon SNS relays messages it receives to the topic's subscribers\. Notable events include environment creation errors and all changes in [environment and instance health](health-enhanced.md)\. Events for Amazon EC2 Auto Scaling operations \(adding and removing instances from the environment\) and other informational events don't trigger notifications\.

![\[Amazon SNS notification email\]](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/sns-notification-email.png)

The Elastic Beanstalk console lets you enter an email address during or after environment creation to create an Amazon SNS topic and subscribe to it\. Elastic Beanstalk manages the lifecycle of the topic, and deletes it when your environment is terminated or when you remove your email address in the [environment management console](environments-console.md)\.

The `aws:elasticbeanstalk:sns:topics` namespace provides options for configuring an Amazon SNS topic by using configuration files, or by using a CLI or SDK\. These methods let you configure the type of subscriber and the endpoint, so that the subscriber can be an Amazon SQS queue or HTTP URL\.

You can only turn Amazon SNS notifications on or off\. Depending on the size and composition of your environment, the frequency of notifications sent to the topic can be high\. For notifications that are sent only under specific circumstances, you can [configure your environment to publish custom metrics](health-enhanced-cloudwatch.md) and [set Amazon CloudWatch alarms](using-features.alarms.md) to notify you when those metrics reach an unhealthy threshold\.

## Configuring Notifications<a name="configuration-notifications-console"></a>

The Elastic Beanstalk console lets you enter an email address to create an Amazon SNS topic for your environment\.

**To configure notifications in the Elastic Beanstalk console**

1. Open the [Elastic Beanstalk console](https://console.aws.amazon.com/elasticbeanstalk)\.

1. Navigate to the [management page](environments-console.md) for your environment\.

1. Choose **Configuration**\.

1. On the **Notifications** configuration card, choose **Modify**\.

1. Enter an email address\.  
![\[Elastic Beanstalk notifications configuration page\]](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-config-sns.png)

1. Choose **Save**, and then choose **Apply**\.

When you enter an email address for notifications, Elastic Beanstalk creates an Amazon SNS topic for your environment and adds a subscription\. Amazon SNS sends an email to the subscribed address to confirm the subscription\. You must click the link in the confirmation email to activate the subscription and receive notifications\.

## The aws:elasticbeanstalk:sns:topics Namespace<a name="configuration-notifications-namespace"></a>

Use the options in the [aws:elasticbeanstalk:sns:topics`` namespace](command-options-general.md#command-options-general-elasticbeanstalksnstopics) to configure Amazon SNS notifications for your environment\. You can set these options by using [configuration files](ebextensions.md), a CLI, or an SDK\.

`Notification Endpoint` – Email address, Amazon SQS queue, or URL to send notifications to\. Setting this option creates an SQS queue and a subscription for the specified endpoint\. If the endpoint is not an email address, you must also set the `Notification Protocol` option\. SNS validates the value of `Notification Endpoint` based on the value of `Notification Protocol`\. Setting this option multiple times creates additional subscriptions to the topic\. Removing this option deletes the topic\.

`Notification Protocol` – The protocol used to send notifications to the `Notification Endpoint`\. This option defaults to `email`\. Set this option to `email-json` to send JSON\-formatted emails, `http` or `https` to post JSON\-formatted notifications to an HTTP endpoint, or `sqs` to send notifications to an SQS queue\.

**Note**  
AWS Lambda notifications are not supported\.

`Notification Topic ARN` – After setting a notification endpoint for your environment, read this setting to get the ARN of the SNS topic\. You can also set this option to use an existing SNS topic for notifications\. A topic that you attach to your environment by using this option is not deleted when you change this option or terminate the environment\.

**Note**  
When you specify an existing SNS topic, be sure the topic has the required permissions\. The following policy allows Elastic Beanstalk to send messages to the topic\.  

```
{
  "Version": "2012-10-17",
  "Statement": [{   
    "Sid": "AWSElasticBeanstalkSNSPublishStatement-20170914",
    "Effect": "Allow",   
    "Principal": {
      "Service": "elasticbeanstalk.amazonaws.com"
    },
    "Action": "sns:Publish",   
    "Resource": "arn:aws:sns:region:topic-owner-account-id:topic-name"
  }]
}
```
Open the [Amazon SNS console](https://console.aws.amazon.com/sns/v2/home)\.
Choose **Topics**, and then choose the topic with the policy you want to update\.
Choose **Other topic actions**, and then choose **Edit topic policy**\.
Choose **Advanced view**, and then add the statement specified in this section, using the appropriate values for the region, account ID, and topic name\. Specify a policy statement `Sid` that is unique within the topic's policy\. For example, you can use today's date as part of the name\.
Choose **Update policy**\.

`Notification Topic Name` – Set this option to customize the name of the Amazon SNS topic used for environment notifications\. If a topic with the same name already exists, Elastic Beanstalk attaches that topic to the environment\.

**Warning**  
If you attach an existing SNS topic to an environment with `Notification Topic Name`, Elastic Beanstalk will delete the topic if you terminate the environment or change this setting\.

Changing this option also changes the `Notification Topic ARN`\. If a topic is already attached to the environment, Elastic Beanstalk deletes the old topic, and then creates a new topic and subscription\.

The The EB CLI and Elastic Beanstalk console apply recommended values for the preceding options\. You must remove these settings if you want to use configuration files to configure the same\. See [Recommended Values](command-options.md#configuration-options-recommendedvalues) for details\.