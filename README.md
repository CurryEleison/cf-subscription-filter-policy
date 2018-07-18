# Managing SNS Subscription Filter Policies with Cloudformation

Currently, AWS CloudFormation doesn't have a resource to manage filter policies on SNS subscriptions. This is a workaround with a lambda-backed custom resource which should work in most cases. If AWS ever introduces an officially supported CloudFormation resource to do this job you should definitely switch to that and stop using this workaround.

## Usage
You want to copy the `FilterPolicyLambdaRole`, `FilterPolicyLambdaPermissions` and `FilterPolicyLambda` resources from the sample into your own CloudFormation template and update your stack.

Once you have added the lambda you can use it like so:
```
  MySubscriptionFilterPolicy:
    Type: Custom::SnsSubscriptionFilterPolicy
    Properties:
      # ServiceToken should point to the lambda created
      ServiceToken: !GetAtt FilterPolicyLambda.Arn
      # TopicArn should be the ARN of the SNS topic
      TopicArn: !Ref MyFilteringTopic
      # Endpoint is the ARN of the subscription endpoint
      Endpoint: !GetAtt MyEndpointResouce.Arn
      # Filterpolicy is the policy you want to attach to the subscription
      FilterPolicy: '{"store": ["example_corp"], "customer_interests": ["rugby", "football", "baseball"]'
```

## Implementation Choices
Write to me if you are concerned about some of the points below. If the code ever gets any real use I promíse to improve it.
### Only one subscription per topic and endpoint
Unfortunately, CloudFormation doesn't expose the ARN of an SNS subscription, so to find the correct physical resource to update we have to use ARNs of the source topic and the subscription endpoint. This will not work if a topic has multiple subscriptions to the same endpoint. 
### Fails on monster topics
If you have more than 100 subscriptions to a topic the code would get flaky
### Error messages not super helpful
I had some trouble surfacing the right error messages. If anybody ever uses this I promise to improve it
### Inline lambda code
The code is written as an inline lambda so it's easy to add to an existing stack, and so cfnresponse is included automatically. 
### Yaml defined filter policies can fail
If a filter policy gets sufficiently complicated it seems that the translation into a json policy can fail. You cna just put the filter policy as a quoted json string instead as a workaround. This is fixable, but I don't need it myself and haven't spent the time.
