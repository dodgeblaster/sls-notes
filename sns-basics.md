# SNS Basics

These docs assume a few things:
- You are using the serverless cli
- You have env variables set in serverless.yml

Example of a serverless.yml using a function and a dynamo table:
```yml
service: project-name

plugins:
    - serverless-pseudo-parameters

provider:
    name: aws
    runtime: nodejs10.x
    environment:
        STAGE: ${opt:stage, 'dev'}
        REGION: ${opt:region, 'us-east-1'}
        ACCOUNT_ID: #{AWS::AccountId}
        TABLE: ${self:service}-${opt:stage, 'dev'}
    iamRoleStatements:
        - Effect: Allow
          Action:
              - SNS:Publish
          Resource: 'arn:aws:sns:#{AWS::Region}:#{AWS::AccountId}:name-of-topic'

custom:
    stage: ${opt:stage, self:provider.stage}
    region: ${opt:region, self:provider.region}
    
functions:
    hello:
        handler: handler.hello
        events:
            - http:
                  path: /
                  method: get
                  
resources:
    Resources:
        mySNSTopic:
          Type: AWS::SNS::Topic
          Properties:
            TopicName: name-of-topic-${self:custom.stage}
```

## How to setup SNS
In any file, you can setup a sns emitter by doing the following:
```js
const AWS = require('aws-sdk')
const sns = new AWS.SNS({
    region: process.env.AWS_REGION
})
```

## Publish an event
```js
const emit = {
  userCreated: async (data) => {
    return SNS.publish({
            Subject: 'user-created',
            Message: JSON.stringify(data),
            TopicArn: `arn:aws:sns:${process.env.REGION}:${process.env.ACCOUNT_ID}:user-created`
        }).promise()
  }
}
```
