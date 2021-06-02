# Deploying the solution in your own AWS account
Follow the steps below to deploy the sample templates. 
1. Using command line interface, clone the git repo at https://github.com/santanu-dey/fargate-three-tier-architecture.git
```shell
git clone https://github.com/santanu-dey/fargate-three-tier-architecture.git
``` 
2. Then change directory to the source code folder
```shell
cd fargate-three-tier-architecture
```
3. Upload the template files to an S3 bucket in your account. First, create a target bucket in S3. You must choose a unique bucket name. 
```shell
aws s3 mb s3://<your_own_bucket_name> --profile <your_awscli_profile>
```
The ¬--prrofile parameter is used by AWS CLI to pick up correct configuration for the target AWS account to use, from available list of preconfigured cli profiles. 
Then sync the local templates to the target S3 bucket just created.
You can try a dry run first to make sure unwanted local files are excluded. 
```shell
aws s3 sync --dryrun . s3://<your_own_bucket_name> --exclude ".*" --profile <your_awscli_profile>
```
Then repeat the command without the dry run option.
```shell
aws s3 sync . s3://<your_own_bucket_name> --exclude ".*" --profile <your_awscli_profile>
```
4. Now you can deploy the stack to the AWS account. 
Replace <your_own_bucket_name> with your chosen bucket name, as earlier. Also replace <your_own_db_password> in the command below with a password of your own choice for the database.
``` aws 
cloudformation create-stack \
--stack-name fargate-3-tier-stack \
--template-url https://<your_own_bucket_name>.s3.amazonaws.com/ecs-three-tier-architecture-base.yaml  \
--parameters ParameterKey=ResourceBucket,ParameterValue=<your_own_bucket_name> \
ParameterKey=WordPressDBPassword,ParameterValue=<your_own_db_password> \
--capabilities CAPABILITY_NAMED_IAM --profile <your_awscli_profile>
```
If you wish to check the status the of the stack you can use this command. 
```shell
aws cloudformation describe-stacks --stack-name fargate-3-tier-stack \
--profile <your_awscli_profile>
```
When you see a stack status of CREATE_COMPLETE, you should also see an output section like below from the command above.
```shell
Outputs:
  - Description: ECS Cluster on Fargate
    OutputKey: FargateCluster
    OutputValue: FargateClusterFor3tierApp
  - Description: ALB Endpoint (http URL) to request the application.
    OutputKey: ALBEndPoint
    OutputValue: ALB-for-Fargate-3-Tier-App-1989139838.ap-southeast-1.elb.amazonaws.com
```
Please note down both the output values from the command output like above,
| OutputKey	| OutputValue |
| ALBEndPoint | This is the ALB Endpoint (http URL where the WordPress application will be accessible. |
| FargateCluster | Name of the ECS Cluster on Fargate. |

For a full list of CloudFormation stack events please run
```shell
aws cloudformation describe-stack-events --stack-name fargate-3-tier-stack \
--profile <your_awscli_profile>
```

# Clean-up 
Delete the stack 
```shell
aws cloudformation delete-stack --stack-name fargate-3-tier-stack --profile <your_awscli_profile>
```


# TODO
* naming of target group - tag
* naming of aurora Db
* change the tag name
* encryption of storage and DB 
* storing of secrets 
* use route 53
* tighten the IAM role 
