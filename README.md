## Overview

* Creates an AWS lambda function written in Go
  * uses [https://github.com/eawsy/aws-lambda-go-shim]
  * fast AWS launch using the Python runtime
  * loads the Go code as a 1.8 plugin
* Creates an AWS API Gateway endpoint to call the Go lambda function
* Deploys to AWS using two different methods...
  1. AWS CloudFormation SAM
    * [https://aws.amazon.com/about-aws/whats-new/2016/11/introducing-the-aws-serverless-application-model/]
  2. Serverless
    * [https://serverless.com/]
    * simpler alternative; uses CloudFormation

## Dependencies

* Go packages
  * $ go get -u -d github.com/eawsy/aws-lambda-go-net/...
* AWS CLI (for CloudFormation SAM)
  * [http://docs.aws.amazon.com/cli/latest/userguide/installing.html]
* Serverless (alternative which uses CloudFormation)
  * $ npm install -g serverless

## Build

* Compiles the 'handler.so' file (Golang 1.8 plugin shared-library)
* Packages the compiled handler for deployment as an AWS lambda function (handler.zip)

## Deploy using CloudFormation

```bash
# create the s3 bucket for deployment data
$ aws s3 mb s3://BUCKET-NAME

# package the deployment using the AWS SAM template
$ aws cloudformation package \
      --template-file cf_template.sam.yaml \
      --output-template-file deploy.out.yaml \
      --s3-bucket fithub-deploy-packages

# deploy the stack
$ aws cloudformation deploy \
      --template-file deploy.out.yaml \
      --capabilities CAPABILITY_IAM \
      --stack-name <STACK-NAME>
```

## Test CloudFormation Deployment

```bash
# get the end-point url
$ URL=`aws cloudformation describe-stacks \
      --stack-name <STACK-NAME> \
      --query Stacks[0].Outputs[0] | jq .OutputValue`

# call the end-point url
$ curl -v <URL>
```

## Deploy using Serverless

```bash
$ sls deploy

Serverless: Uploading CloudFormation file to S3...
Serverless: Uploading service .zip file to S3 (9.95 MB)...
Serverless: Updating Stack...
Serverless: Checking Stack update progress...
..............................
Serverless: Stack update finished...
Service Information
service: serverless-golang
stage: dev
region: us-east-1
api keys:
  None
endpoints:
  GET - https://2tu6gun39a.execute-api.us-east-1.amazonaws.com/dev/hello
functions:
  hello: serverless-golang-dev-hello
```

# Test Serverless Deployment

```bash
# get the end-point url
$ URL=`sls info | grep GET | cut -f5 -d" "`

# call the end-point url
$ curl -v $URL
```
