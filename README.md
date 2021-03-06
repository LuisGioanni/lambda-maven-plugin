# lambda-maven-plugin

[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.github.seanroy/lambda-maven-plugin/badge.svg)](https://maven-badges.herokuapp.com/maven-central/com.github.seanroy/lambda-maven-plugin)
<br/>
### Usage
`group id: com.github.seanroy`<br />
`artifact id: lambda-maven-plugin`<br />
`version: 2.1.5`<br />
<br/><br/>
Please note that the artifact has been renamed from lambduh-maven-plugin to
lambda-maven-plugin.

### Deploy from command line
```
mvn package shade:shade lambda:deploy-lambda 
```

### Delete from command line
```
mvn lambda:delete-lambda
```

### Configuration
All of the AWS Lambda configuration parameters may be set within the lambda plugin configuration or on the Maven command line using the -D directive.

* `accessKey` Your user's AWS access key.
* `secretKey` Your user's AWS secret access key.
* `functionCode` REQUIRED The location of your deliverable. For instance, a jar file for a Java8 lambda function.
* `version` REQUIRED version of the deliverable.
* `s3Bucket` REQUIRED Defaults to lambda-function-code. The AWS S3 bucket to which to upload your code from which it will be deployed to Lambda.
* `region` Defaults to eu-west-1 The AWS region to use for your function.
* `runtime` Defaults to Java8 Specifies whether this is Java8, NodeJs and Python.
* `lambdaRoleArn` REQUIRED The ARN of the AWS role which the lambda user will assume when it executes. Note that the role must be assumable by Lambda and must have Cloudwatch Logs permissions and AWSLambdaDynamoDBExecutionRole policy.
* `lambdaFunctions` Lamda functions that can be configured using tags in pom.xml.
* `lambdaFunctionsJSON` JSON configuration for Lambda Functions. This is preferable configuration.
* `timeout` Defaults to 30 seconds. The amount of time in which the function is allowed to run.
* `memorySize` Defaults to 1024MB NOTE: Please review the AWS Lambda documentation on this setting as it could have an impact on your billing.
* `vpcSubnetIds` The VPC Subnets that Lambda should use to set up your VPC configuration. Format: "subnet-id (cidr-block) | az name-tag".
* `vpcSecurityGroupIds` The VPC Security Groups that Lambda should use to set up your VPC configuration. Format: "sg-id (sg-name) | name-tag". Should be configured.
* `publish` This boolean parameter can be used to request AWS Lambda to update the Lambda function and publish a version as an atomic operation. This is global for all functions and won't overwrite publish paramter in provided Lambda configuration
* `functionNameSuffix` The suffix for the lambda function. Function name is automatically suffixed with it. When left blank no suffix will be applied.
* `forceUpdate` This boolean parameter can be used to force update of existing configuration. Use it when you don't publish a function and want to deploy code in your Lambda function.
* `triggers` A list of one or more triggers that execute Lambda function. Currently `CloudWatch Events - Schedule`, `SNS` and `DynamoDB` is supported. When `functionNameSuffix` is present then suffix will be added automatically.
* `environmentVariables` Map to define environment variables for Lambda functions enable you to dynamically pass settings to your function code and libraries, without making changes to your code. Deployment functionality merges those variables with the one provided in json configuration.

Current configuration of LambdaFunction can be found in LambdaFunction.java.

### Example configuration in pom.xml
```
        <project
        
            ...

            <properties>
                <lambda.functionCode>${project.build.directory}/${project.build.finalName}.jar</lambda.functionCode>
                <lambda.version>${project.version}</lambda.version>
                <lambda.publish>true</lambda.publish>
                <lambda.forceUpdate>true</lambda.forceUpdate>
                <lambda.functionNameSuffix>dev</lambda.functionNameSuffix>
            </properties>
            
           ...
           
            <plugin>
                    <groupId>com.github.seanroy</groupId>
                    <artifactId>lambda-maven-plugin</artifactId>
                    <version>2.1.5</version>
                    <configuration>
                        <functionCode>${lambda.functionCode}</functionCode>
                        <version>${lambda.version}</version>
                        <environmentVpcSecurityGroupsIds>sg-123456</environmentVpcSecurityGroupsIds>
                        <environmentVpcSubnetIds>subnet-123456,subnet-123456,subnet-123456</environmentVpcSubnetIds>
                        <lambdaRoleArn>arn:aws:iam::1234567:role/YourLambdaS3Role</lambdaRoleArn>
                        <s3Bucket>mys3bucket</s3Bucket>
                        <publish>${lambda.publish}</publish>
                        <forceUpdate>${lambda.forceUpdate}</forceUpdate>
                        <functionNameSuffix>${lambda.functionNameSuffix}</functionNameSuffix>
                        <environmentVariables>
                           <key0>value0</key0>
                        </environmentVariables>
                        <lambdaFunctionsJSON>
                            [
                              {
                                "functionName": "my-function-name-0",
                                "description": "I am awesome function",
                                "handler": "no.flowlab.lambda0::test",
                                "timeout": 30,
                                "memorySize": 512,
                                "triggers": [
                                  {
                                    "integration": "CloudWatch Events - Schedule",
                                    "ruleName": "every-minute",
                                    "ruleDescription": "foo bar",
                                    "scheduleExpression": "rate(1 minute)"
                                  },
                                  {
                                    "integration": "DynamoDB",
                                    "dynamoDBTable": "myTable",
                                    "batchSize": 100,
                                    "startingPosition": "TRIM_HORIZON"
                                  },
                                  {
                                    "integration": "SNS",
                                    "SNSTopic": "SNSTopic-1"
                                  },
                                  {
                                    "integration": "SNS",
                                    "SNSTopic": "SNSTopic-2"
                                  },
                                  {
                                    "integration": "Alexa Skills Kit"
                                  }
                                ],
                                "environmentVariables": {
                                  "key1": "value1",
                                  "key2": "value2"
                                }
                              },
                              {
                                "functionName": "my-function-name-1",
                                "description": "I am awesome function too",
                                "handler": "no.flowlab.lambda1"
                              }
                            ]
                        </lambdaFunctionsJSON>
                    </configuration>
            </plugin>
            
            ...
            
        </project>
```

### Credentials
Your AWS credentials may be set on the command line or in the plugin configuration. If `accessKey` and
`secretKey` are not explicitly defined, the plugin will look for them in your environment or in your
~/.aws/credentials file

IAM permissions required by this plugin:
* action `s3:GetObject` and `s3:PutObject` on resource `arn:aws:s3:::<s3Bucket>/*`
* action `s3:ListBucket` on resource `arn:aws:s3:::<s3Bucket>`
* action `s3:CreateBucket` if you want the plugin to create the S3 bucket you specify.
* action `lambda:CreateFunction`
* action `lambda:InvokeFunction`
* action `lambda:GetFunction`
* action `lambda:UpdateFunctionCode`
* action `lambda:UpdateFunctionConfiguration`
* action `lambda:ListAliases`
* action `events:PutRule` on  resource `arn:aws:events:<region>:<acount-number>:rule/*`
* action `events:PutTargets` on  resource `arn:aws:events:<region>:<acount-number>:rule/*`

### Developers
If you are interested in contributing to this project, please note that current development can be found in the SNAPSHOT branch of the coming release.  When making pull requests, please create them against this branch.

A test harness has been provided which can be run with `mvn test` Please use 
this and feel free to add additional tests. Note that the basic-pom.xml file
requires you to add your role arn in order to work.  As such, basic-pom.xml
has been added to .gitignore so that you don't accidentally commit your role
to the file.  If you add more pom's as part of enhancing the test suite,
please remember to add them to .gitignore.

### Releases
2.1.5
* Add support for environment variables [Issue 48](https://github.com/SeanRoy/lambda-maven-plugin/issues/48)
* Thanks [krzysztof@flowlab.no](mailto:krzysztof@flowlab.no)

2.1.4
* Fixed [Issue 46] (https://github.com/SeanRoy/lambda-maven-plugin/issues/46)
* Thanks [krzysztof@flowlab.no](mailto:krzysztof@flowlab.no)

2.1.3
* Fixed [Issue 42] (https://github.com/SeanRoy/lambda-maven-plugin/issues/42)
* Thanks [krzysztof@flowlab.no](mailto:krzysztof@flowlab.no)

2.1.2
* Added trigger to allow Alexa Skills Kit Integration.

2.1.1
* Remove deprecated `scheduledRules` and `topics` functionality
* Thanks [krzysztof@flowlab.no](mailto:krzysztof@flowlab.no)

2.1.0
* Add support for triggers. Deprecated `scheduledRules` and `topics` as thouse have been moved to triggers
* Add support for DynamoDB stream. `lambdaRoleArn` requires AWSLambdaDynamoDBExecutionRole policy
* Update to AWS SDK 1.11.41
* Thanks [krzysztof@flowlab.no](mailto:krzysztof@flowlab.no)

2.0.1
* Fixed [Issue 33] (https://github.com/SeanRoy/lambda-maven-plugin/pull/33) Thank Vũ Mạnh Tú.

2.0.0
* Add support for configuration many lambda functions in one deliverable, supports config in JSON, each lumbda function configuration can be fully customized
* Add support for version aliases when publish is activated
* Change defaults
* Fixed some mainor code smells
* Remove support for annotations
* Refactor code to java8
* Add publish flag, which controls Lambda versioning in AWS
* Force update support
* Add support for SNS topics
* Add support for scheduled rules, cron jobs which trigger lambda function
* Thanks [krzysztof@flowlab.no](mailto:krzysztof@flowlab.no)

1.1.6
* Removed debugging related code.

1.1.5
* Fixed bug where default value of functionNameSuffix evaluating to null instead of a blank string.

1.1.4
* Added functionNameSuffix optional property.

1.1.3
* Fixed [Issue 28] (https://github.com/SeanRoy/lambda-maven-plugin/issues/28) 

1.1.2
* Fixed invalid dependency to lambda-maven-annotations

1.1.1
* Added support for Virtual Private Clouds. Thanks Jem Rayfield.
* Added ability to designate functions for deployment via LambduhFunction annotations. (Details coming soon).

1.1.0
* Added delete goal.

1.0.6
* Issue 19 Added test harness.
* Update function code if code or configuration has changed instead of
deleting and recreating every time.  Thanks Guillermo Menendez

1.0.5
* Accidental deployment of release.  Should be functionally equivalent to 
1.0.4.

1.0.4
* Fixed issue 8
* No longer uploads function code to S3 when no changes have been made to speed up
  development cycle over slow connections.  Thanks Philip M. White.
* Fixed logging.

1.0.3 
* Fixed a bug where getting a bucket fails if existing. Thanks buri17
* Fixed problem with region specification. Thanks buri17
* Adding ability to pull creds from the default provider. Thanks Chris Weiss

1.0.2 
* Fixed PatternSyntaxException on windows https://github.com/SeanRoy/lambda-maven-plugin/issues/1
