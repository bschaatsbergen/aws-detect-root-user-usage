# detect and alert AWS root user usage

**Note:** Don't deploy this stack yet, it's under development

When you create a new AWS account it's important that you immediately secure the root user as it has full access to everything in your AWS account. It's very important that you prevent anyone from using this account unless there's a specific task that requires you to use root privilege.

This repository contains a `stack.yaml` that you can deploy to your AWS account in order to capture and alert any root user usage. It contains an EventBridge rule, SNS topic and SNS subscription.

## Deploy

To deploy the stack we simply issue the `cloudformation deploy` command.

```sh
EmailAddress="brunoschaatsbergen@binx.io"
aws cloudformation deploy --stack-name root-user-usage-stack \
    --template-file stack.yaml \
    --parameter-overrides EmailAddress=${EmailAddress} \
    --region eu-central-1
```

## EventBridge

The stack contains an EventBridge rule using a event pattern that captures
any console sign in and API calls tracked via CloudTrail from the root user.

```json
{
	"detail-type": [
      "AWS API Call via CloudTrail", 
      "AWS Console Sign In via CloudTrail"
    ],
	"detail": {
		"userIdentity": {
			"type": [
        "Root"
      ]
		}
	}
}
```

## SNS Subscription

Please note that the SNS subscription has to be confirmed by the one owning the email address. After the stack is successfully deployed, the sender address is `no-reply@sns.amazonaws.com`.

Tip: It's a best practice to add the `no-reply@sns.amazonaws.com` address to your mailbox allow list.

## Resources

| Name                                                                                                                                                 | Type     |
| ---------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| [AWS::Events::Rule](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-events-rule.html#aws-resource-events-rule--examples) | resource |
| [AWS::SNS::Topic](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sns-topic.html)                                      | resource |
| [AWS::SNS::Subscription](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-sns-subscription.html)                          | resource |

## Parameters

| Name         | Description                                                           | Type     | Default | Required |
| ------------ | --------------------------------------------------------------------- | -------- | ------- | :------: |
| EmailAddress | Enter an email address you want to subscribe to the Amazon SNS topic. | `string` | n/a     |   yes    |
