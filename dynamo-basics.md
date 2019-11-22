# Serverless Dynamo Basics

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
        TABLE: ${self:service}-${opt:stage, 'dev'}
    iamRoleStatements:
        - Effect: Allow
          Action:
              - dynamodb:Query
              - dynamodb:Scan
              - dynamodb:GetItem
              - dynamodb:PutItem
              - dynamodb:UpdateItem
              - dynamodb:DeleteItem
          Resource: 'arn:aws:dynamodb:#{AWS::Region}:#{AWS::AccountId}:table/${self:service}-${self:custom.stage}'

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
        feedbackContactTable:
            Type: AWS::DynamoDB::Table
            Properties:
                TableName: ${self:service}-${self:custom.stage}
                AttributeDefinitions:
                    - AttributeName: PK
                      AttributeType: S
                    - AttributeName: SK
                      AttributeType: S
                KeySchema:
                    - AttributeName: PK
                      KeyType: HASH
                    - AttributeName: SK
                      KeyType: RANGE
                BillingMode: PAY_PER_REQUEST
```


## How to setup Dynamo in a function
In any file, you can setup a Dynamo operations by doing the following:
```js
const AWS = require('aws-sdk')
const TABLE = process.env.TABLE
const dynamoDb = new AWS.DynamoDB.DocumentClient({
    region: process.env.REGION || 'us-east-1'
})
```

The `aws-sdk` is part of the environment of aws lambda's, so we can assume that is available for us to use.
We get the table name from our environment variables set in our servlerless.yml file.
We also get the region from our env variables.

# Basic Operations

### Put Item
```js
const addItem: async (data) => {
    const params = {
        TableName: TABLE,
        Item: {
            PK: data.aPartitionKey,
            SK: data.aSortKey,
            anything: data.anything
        }
    }

    return await dynamoDb.put(params).promise()
}

```
### Remove Item
```js
const removeItem: async (data) => {
    const params = {
        TableName: TABLE,
        Key: {
            PK: data.aPartitionKey,
            SK: data.aSortKey,
        }
    }

    return await dynamoDb.delete(params).promise()
}
```

### Get 1 Item
```js
const getItem: async (data) => {
    const params = {
        TableName: TABLE,
        Key: {
            PK: data.aPartitionKey,
            SK: data.aSortKey,
        }
    }

    const result = await dynamoDb.get(params).promise()
    return result.Item
}
```

### Get Many Items
```js
const getAllItems: async (data) => {
    const params = {
        TableName: TABLE,
        KeyConditionExpression: 'PK = :pk AND begins_with(SK, :sk)',
        ExpressionAttributeValues: {
            ':pk': data.aPartitionKey,
            ':sk': data.beginningOfSortKey
        }
    }

    const result = await dynamoDb.query(params).promise()
    return result.Items
}
```
