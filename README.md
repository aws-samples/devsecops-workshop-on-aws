## DevSecOps on AWS

### [Building a Pipeline for Test and Production Stacks]()
This walkthrough builds a pipeline for a sample WordPress site in a stack. The pipeline is separated into four stages. Each stage must contain at least one action, which is a task the pipeline performs on your artifacts (your input). A stage organizes actions in a pipeline. CodePipeline must complete all actions in a stage before the stage processes new artifacts, for example, if you submitted new input to rerun the pipeline.

By the end of this walkthrough, you'll have a pipeline that performs the following workflow:

1. The first stage of the pipeline retrieves a source artifact (an AWS CloudFormation template) from a repository. You'll prepare an artifact that includes a sample WordPress template and upload it to an S3 bucket.

3. In the second stage, the pipeline performs a series of validation tests to the AWS CloudFormation template. These include cfn-validate-template, cfn_nag and taskcat, and then the pipeline continues to the next stage.

3. In the third stage, the pipeline creates a test stack and then waits for your approval.
After you review the test stack, you can choose to continue with the original pipeline or create and submit another artifact to make changes. If you approve, this stage deletes the test stack, and then the pipeline continues to the next stage.

4. In the fourth stage, the pipeline creates a change set against a production stack, and then waits for your approval.
In your initial run, you won't have a production stack. The change set shows you all of the resources that AWS CloudFormation will create. If you approve, this stage executes the change set and builds your production stack.

### How to run 

1. [Clone](https://help.github.com/articles/cloning-a-repository/) this repository. From your terminal application, execute the following command:

`git clone https://github.com/aws-samples/aws-devsecops-workshop`

This creates a directory named aws-devsecops-workshop in your current directory.

2. [Create AWS CodeCommit repository](http://docs.aws.amazon.com/codecommit/latest/userguide/getting-started.html#getting-started-create-repo) in your AWS account and [set up AWS CLI](http://docs.aws.amazon.com/codecommit/latest/userguide/how-to-create-repository.html#how-to-create-repository-cli). Name your repository wordpress-cfn. Alternatively, from your terminal application, execute the following command:

`aws codecommit create-repository --repository-name wordpress-cfn --repository-description "This template installs WordPress with a local MySQL database for storage"`

Note the `cloneUrlHttp URL` in the response from the CLI.

3. Add a new remote. From your terminal application, within the wordpress-dfn directory, execute the following command:

`git init && git remote add AWSCodeCommit HTTP_CLONE_URL_FROM_STEP_2`

[Create the local git setup](http://docs.aws.amazon.com/codecommit/latest/userguide/setting-up.html) required to push code to CodeCommit repository.


4. Initialise the parameters Wordpress fetches from SSM. Example:
```
aws ssm put-parameter --name dbName --type String --value "WordPressDB"
aws secretsmanager create-secret --name dbPassword --secret-string DBPassword
aws secretsmanager create-secret --name dbRootPassword --secret-string DBRootPassword
aws ssm put-parameter --name dbUser --type String --value "DBuser"
```

5. Launch the CFN templates that create the [pipeline](https://console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks/new?stackName=devsecops-wordpress-pipeline&templateURL=https://artifact-store-ejanicas.s3-eu-west-1.amazonaws.com/basic-pipeline.yaml) and the [config rules](https://console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks/new?stackName=config&templateURL=https://artifact-store-ejanicas.s3-eu-west-1.amazonaws.com/config.yaml).


## Tools used in this Workshop

#### Pre. [git-secrets](https://github.com/awslabs/git-secrets)
`git-secrets` scans commits, commit messages, and `--no-ff` merges to prevent adding secrets into your git repositories. If a commit, commit message, or any commit in a `--no-ff` merge history matches one of your configured prohibited regular expression patterns, then the commit is rejected.

#### Pre. [Secrets Manager](https://aws.amazon.com/secrets-manager/)
AWS Secrets Manager helps you protect secrets needed to access your applications, services, and IT resources. The service enables you to easily rotate, manage, and retrieve database credentials, API keys, and other secrets throughout their lifecycle. Users and applications retrieve secrets with a call to Secrets Manager APIs, eliminating the need to hardcode sensitive information in plain text. Secrets Manager offers secret rotation with built-in integration for Amazon RDS, Amazon Redshift, and Amazon DocumentDB. Also, the service is extensible to other types of secrets, including API keys and OAuth tokens. In addition, Secrets Manager enables you to control access to secrets using fine-grained permissions and audit secret rotation centrally for resources in the AWS Cloud, third-party services, and on-premises.

#### Pre. [CodeGuru Reviewer](https://aws.amazon.com/codeguru/)
Amazon CodeGuru is a machine learning service for automated code reviews and application performance recommendations. It helps you find the most expensive lines of code that hurt application performance and keep you up all night troubleshooting, then gives you specific recommendations to fix or improve your code. CodeGuru is powered by machine learning, best practices, and hard-learned lessons across millions of code reviews and thousands of applications profiled on open source projects and internally at Amazon. With CodeGuru, you can find and fix code issues such as resource leaks, potential concurrency race conditions, and wasted CPU cycles. And with low, on-demand pricing, it is inexpensive enough to use for every code review and application you run. CodeGuru supports Java applications today, with support for more languages coming soon. CodeGuru helps you catch problems faster and earlier, so you can build and run better software.


#### 0. [cfn-validate-template](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-validate-template.html)
The aws cloudformation validate-template command is designed to check only the syntax of your template. It does not ensure that the property values that you have specified for a resource are valid for that resource. Nor does it determine the number of resources that will exist when the stack is created.


#### 1. [cfn_nag](https://github.com/stelligent/cfn_nag)
The cfn-nag tool looks for patterns in CloudFormation templates that may indicate insecure infrastructure. Roughly speaking it will look for:

- IAM rules that are too permissive (wildcards)
- Security group rules that are too permissive (wildcards)
- Access logs that aren't enabled
- Encryption that isn't enabled
- Password literals


#### 2. [taskcat](https://github.com/aws-quickstart/taskcat)
**taskcat** is a tool that tests AWS CloudFormation templates. It deploys your AWS CloudFormation template in multiple AWS Regions and generates a report with a pass/fail grade for each region. You can specify the regions and number of Availability Zones you want to include in the test, and pass in parameter values from your AWS CloudFormation template. taskcat is implemented as a Python class that you import, instantiate, and run.

https://aws.amazon.com/blogs/infrastructure-and-automation/up-your-aws-cloudformation-testing-game-using-taskcat/


### 3. [Continuous integration and evaluation of security controls](https://aws.amazon.com/blogs/security/how-to-manage-security-governance-using-devops-methodologies/)
After you write multiple action statements and scenarios supporting the user story (both positive and negative), you can write them up as a runbook, an AWS Config rule, or a combination of both as required.

The second example acceptance criteria above would need to be written as a runbook, as it’s a responsive control. You wouldn’t want to generate a stream of emails to your security operations manager to validate that it’s working.

The other two examples could be written as AWS Config rules using a call to the iam:simulate-custom-policy API, since they are related to preventative controls. An AWS Config rule allows your entire account to be continuously evaluated for compliance, essentially evaluating your control adherence on a <15min basis, rather than from a yearly audit.

Committing those runbook and AWS Config rules to a central code repository fosters the agility of the controls. In the case of runbooks, you may want to adopt a light-weight markup format, such as markdown, that you are able to check in like code. The defined controls can then sit in a CI/CD pipeline, allowing the security controls to be as agile as your pace of innovation.

##### 3.1. [AWS Config](https://aws.amazon.com/config/)
You can use AWS Config as your framework for creating and deploying governance and compliance rules across your AWS accounts and regions. You can codify your compliance requirements as AWS Config rules and author remediation actions using AWS Systems Manager Automation documents and package them together within a conformance pack that can be easily deployed across an organization. Therefore, using AWS Config, you can automate assessment of your resource configurations and resource changes to help you ensure continuous compliance and self-governance across your AWS infrastructure.

##### 3.2. [Amazon Inspector](https://aws.amazon.com/inspector/?nc2=h_ql_prod_se_in)
Amazon Inspector is an API-driven service that analyzes network configurations in your AWS account and uses an optional agent for visibility into your Amazon EC2 instances. This makes it easy for you to build Inspector assessments right into your existing DevOps process, decentralizing and automating vulnerability assessments, and empowering your development and operations teams to make security assessments an integral part of the deployment process.

## License
This library is licensed under the MIT-0 License. See the LICENSE file.