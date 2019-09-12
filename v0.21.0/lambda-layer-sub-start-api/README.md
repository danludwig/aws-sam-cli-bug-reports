# AWS SAM CLI Bug Report for v0.20.0

## Function using !Sub layer with !GetAtt to custom resources in environment variables.

Seems to be an issue when a function resource references a layer using `!Sub` against a parameter default and also contains environment variables whose values come from `!GetAtt` against custom cloudformation resources.

## Steps to reproduce
1. Install aws sam cli version 0.21.0
1. `git clone https://github.com/danludwig/aws-sam-cli-bug-reports.git`
1. `cd ./v0.21.0/lambda-layer-sub-start-api`
1. `sam local start-api --region us-east-2`
1. Invoke [http://127.0.0.1:3000/hello](http://127.0.0.1:3000/hello)
1. Examine runtime error stack trace

## Severity

Moderate. Results in developers having to comment out and hard-code a layer arn during local development. 

## Details

This example was built from scratch with minor modifications to a new `sam init` project.

The bug does not seem to surface when environemnt variable values only use `!Ref` against template parameters.

Commenting out environment variables which use `!GetAtt` against a custom resource cause the builds to succeed. For example, the following will result in a successful build:

```
      Environment:
        Variables:
          SAFE_ENVIRONMENT_VARIABLE_1:
            !Ref TemplateParameterOne
          # OFFENDING_ENVIRONMENT_VARIABLE:
          #   !GetAtt MyCustomResource.Parameter.Value
          SAFE_ENVIRONMENT_VARIABLE_2:
            !Ref MyCustomResource
```

Runtime error stack trace:

```
$ sam local start-api --region us-east-2
Unable to process properties of HelloWorldFunction.AWS::Serverless::Function
Unable to process properties of HelloWorldFunction.AWS::Serverless::Function
Mounting HelloWorldFunction at http://127.0.0.1:3000/hello [GET]
You can now browse to the above endpoints to invoke your functions. You do not need to restart/reload SAM CLI while working on your functions, changes will be reflected instantly/automatically. You only need to restart SAM CLI if you update your AWS SAM template
2019-09-12 17:44:48  * Running on http://127.0.0.1:3000/ (Press CTRL+C to quit)
Invoking app.lambdaHandler (nodejs10.x)
2019-09-12 17:44:55 Found credentials in shared credentials file: ~/.aws/credentials

Fetching lambci/lambda:nodejs10.x Docker container image......
Mounting C:\Users\Dan\Code\sam-inits\aws-sam-cli-bug-reports\v0.21.0\lambda-layer-sub-start-api\hello-world as /var/task:ro,delegated inside runtime container
2019-09-12T21:44:57.218Z        undefined       ERROR   Uncaught Exception      {"errorType":"Runtime.ImportModuleError","errorMessage":"Error: Cannot find module 'danludwig-example-nodejs-lambda-layer'","stack":["Runtime.ImportModuleError: Error: Cannot find module 'danludwig-example-nodejs-lambda-layer'","    at _loadUserApp (/var/runtime/UserFunction.js:100:13)","    at Object.module.exports.load (/var/runtime/UserFunction.js:140:17)","    at Object.<anonymous> (/var/runtime/index.js:36:30)","    at Module._compile (internal/modules/cjs/loader.js:776:30)","    at Object.Module._extensions..js (internal/modules/cjs/loader.js:787:10)","    at Module.load (internal/modules/cjs/loader.js:653:32)","    at tryModuleLoad (internal/modules/cjs/loader.js:593:12)","    at Function.Module._load (internal/modules/cjs/loader.js:585:3)","    at Function.Module.runMain (internal/modules/cjs/loader.js:829:12)","    at startup (internal/bootstrap/node.js:283:19)"]}
START RequestId: 52fdfc07-2182-154f-163f-5f0f9a621d72 Version: $LATEST
END RequestId: 52fdfc07-2182-154f-163f-5f0f9a621d72
REPORT RequestId: 52fdfc07-2182-154f-163f-5f0f9a621d72     Duration: 0.00 ms       Billed Duration: 100 ms Memory Size: 128 MB     Max Memory Used: 42 MB  
{
  "errorType": "Runtime.ImportModuleError",
  "errorMessage": "Error: Cannot find module 'danludwig-example-nodejs-lambda-layer'"
}
Function returned an invalid response (must include one of: body, headers, multiValueHeaders or statusCode in the response object). Response received:
2019-09-12 17:44:57 127.0.0.1 - - [12/Sep/2019 17:44:57] "GET /hello HTTP/1.1" 502 -
2019-09-12 17:44:57 127.0.0.1 - - [12/Sep/2019 17:44:57] "GET /favicon.ico HTTP/1.1" 403 -
```
