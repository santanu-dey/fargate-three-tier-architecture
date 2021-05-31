# How to deploy this stack

Run the following commands order to 


1. validate each individual templates stack locally

```
aws cloudformation validate-template \
--template-body file://ecs-three-tier-architecture-base.yaml \
--profile bootcamp
```

the `--profile` parameter is used by aws cli to pick up correct configuration for the target AWS account I am using, from available preconfigured cli profiles. 


2. upload the template files to S3

Create a target bucket in S3. You must chose an unique bucket name. 

```
aws s3 mb s3://cloud-formation-template --profile bootcamp
```

Sync the local templates to the target S3 bucket just created.

You can try a dryrun first to make sure unwanted local files are excluded. 

```
aws s3 sync --dryrun . s3://cloud-formation-template --exclude ".*" --profile bootcamp
```

Then repeat the command without dry run

```
aws s3 sync . s3://cloud-formation-template --exclude ".*" --profile bootcamp
```

Make the dependent templates publicly readable 
```
aws s3api put-object-acl --bucket cloud-formation-template --key dependencies/networking-template.yaml --acl public-read --profile bootcamp
```

3. Now you can deploy the stack

Either of the commands below should work

```
santadey@a483e7807e92 cloud-formation % aws cloudformation create-stack \
--stack-name three-tier-app \
--template-url https://cloud-formation-template.s3.amazonaws.com/fargate-three-tier-architecture-base.yaml  \
--parameters ParameterKey=ResourceBucket,ParameterValue=cloud-formation-template \
ParameterKey=WordPressDBPassword,ParameterValue=yourpassword \
--capabilities CAPABILITY_NAMED_IAM --profile bootcamp
```


4. check the status of the stack

```
aws cloudformation describe-stacks --stack-name three-tier-app \
--profile bootcamp
```

Full list of events 
```
aws cloudformation describe-stack-events --stack-name three-tier-app \
--profile bootcamp
```

4. delete the stack 

```
aws cloudformation delete-stack --stack-name three-tier-app --profile bootcamp
```



#TODO
* make sure that the task name has a different naming for each cluster. 
* fix quotes, comments and output section
* naming of target group
* encryption of storage and DB 
* storing of secrets 
* use route 53
* tighten the IAM role 
