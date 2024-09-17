# PagerDuty Amazon EventBridge Partner Integration to AWS Lambda

This pattern demonstrates how to use the PagerDuty Amazon EventBridge SaaS integration and AWS Lambda to process events from PagerDuty. This pattern is leveraging the PagerDuty Amazon EventBridge SaaS integration to send incident acknowledgement events from the customer's PagerDuty account to their AWS account, via an Amazon EventBridge Partner event bus. Once the PagerDuty events are in the customer's account, an Amazon EventBridge rule routes incident acknowledgement events to a downstream Lambda function. In production cases, the Lambda function could transform the event, send it to a downstream application, archive it, or send a notification email to a customer using SES. Amazon CloudWatch Log Groups are provisioned for debugging and auditing. This pattern deploys two EventBridge rules, one Lambda function, and two CloudWatch Log Groups.
    
Learn more about this pattern at Serverless Land Patterns: https://serverlessland.com/patterns/pagerduty-eventbridge-lambda

Important: this application uses various AWS services and there are costs associated with these services after the Free Tier usage - please see the [AWS Pricing page](https://aws.amazon.com/pricing/) for details. You are responsible for any AWS costs incurred. No warranty is implied in this example.

## Requirements

* [Create an AWS account](https://portal.aws.amazon.com/gp/aws/developer/registration/index.html) if you do not already have one and log in. The IAM user that you use must have sufficient permissions to make necessary AWS service calls and manage AWS resources.
* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) installed and configured
* [Git Installed](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* [AWS CDK CLI](https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html) (AWS CDK) installed
* [Create a PagerDuty Account](https://www.pagerduty.com/sign-up/) if you do not already have one and log in. 
* [Set up the PagerDuty Amazon EventBridge SaaS integration](https://support.pagerduty.com/main/docs/amazon-eventbridge-integration-guide) if you have not already configured the integration.


## Deployment Instructions

1. Create a new directory, navigate to that directory in a terminal and clone the GitHub repository:
    ``` 
    git clone https://github.com/aws-samples/serverless-patterns
    ```
2. Change directory to the pattern directory:
    ```
    cd pagerduty-eventbridge-lambda
    ```
3. From the command line, use AWS CDK to deploy the AWS resources for the pattern as specified in the app.py file. A command-line argument is needed to deploy the CDK stack, "pagerdutyEventBusName". This argument should be the name of the **SaaS event bus** associated with your PagerDuty [partner event source](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-saas.html).
    ```
    cdk deploy --parameters pagerdutyEventBusName=SAAS_EVENT_BUS_NAME_HERE
    ```

4. Note the outputs from the CDK deployment process. These contain the resource names and/or ARNs which are used for testing. This stack will output the name of the Lambda function deployed for testing to the CLI. See the example below. 

```
Outputs:
PagerdutyIntegrationStack.PagerdutyAcknowledgementLambdaOutput = PagerdutyIntegrationStack-PagerdutyAcknowledgementLambd....
```

## How it works

This service interaction uses an existing PagerDuty SaaS integration in the customer's AWS account. If you do not have the PagerDuty SaaS integration set up in your AWS account, please set it up before deploying this pattern. View the [integration on PagerDuty's marketplace](https://support.pagerduty.com/main/docs/amazon-eventbridge-integration-guide). See [PagerDuty's sample events](https://support.pagerduty.com/main/docs/amazon-eventbridge-integration-guide#when-configuring-the-rule-pattern-in-aws-what-does-the-typical-event-look-like-coming-out-of-pagerduty) for available events. 

This pattern demonstrates how to:
1. Write EventBridge rules that match PagerDuty's event pattern
2. Send events from PagerDuty's EventBridge SaaS integration to Amazon CloudWatch for logging and debugging
3. Transform PagerDuty events using AWS Lambda

See the below architecture diagram from the data flow of this pattern. 

![Architecture Diagram](./img/readme-arch-diagram.png)

## Testing

### Test the AWS Lambda Function

The event.json file included in this pattern is a sample EventBridge event from PagerDuty. This event can be used to test the Lambda function and EventBridge rules deployed by this pattern.

To test the Lambda function via the CLI, copy and paste the following command, replacing the variables in <> with your own values:
```
aws lambda invoke --function-name <PagerdutyIntegrationStack-PagerdutyAcknowledgementLambd-....> --payload file://event.json --cli-binary-format raw-in-base64-out response.json
```

You should receive a response that looks like: 
```
{
    "StatusCode": 200,
    "ExecutedVersion": "$LATEST"
}
```

The command should have create a response.json file in your directory. If you open this file, you see the output of the Lambda function.  

### Test the EventBridge Rule

You cannot put events on the PagerDuty SaaS event bus, only PagerDuty can publish events to this event bus. To test that the EventBridge rules deployed by this pattern were successfully deployed, follow these instructions: 

1. Navigate to the Amazon EventBridge console. Select "Rules". 

2. From the Event bus list, choose the SaaS event bus associated with your PagerDuty partner event source. 

3. From the Rules list, select the "PagerdutyIntegrationStack-PagerdutyAcknowledgementEventsRule54D..." 

![EventBridge Console](./img/EBconsole-rules.png)

4. Choose "Edit" to enter the rule editor. Click through to "Step 2. Build Event Pattern." 

![EventBridge Console](./img/BuildEvent.png)

5. Scroll down to "Sample event - optional." Select "Enter my own," and delete the pre-populated event. Copy the contents of event.json into the event editor. 

![EventBridge Console](./img/SampleEvent.png)

6. Scroll down to "Event pattern." Choose "Test Pattern." 

![EventBridge Console](./img/TestEvent.png)

You should see a green box appear that says "Sample event matched the event pattern." This means that the rule will successfully route incoming events to the AWS Lambda function. 

![EventBridge Console](./img/TestEventSuccessful.png)


## Cleanup
 
1. Delete the stack
    ```bash
    cdk destroy
    ```

----
Copyright 2024 Amazon.com, Inc. or its affiliates. All Rights Reserved.

SPDX-License-Identifier: MIT-0