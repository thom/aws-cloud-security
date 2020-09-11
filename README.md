# Udacity AWS Cloud Architect Program - Project 3: Cloud Security - Protecting Resources and Data in the Cloud

## Exercise 1 - Deploy Project Environment

### Task 1: Review Architecture Diagram

![Starting architecture diagram](schematics/AWS-WebServiceDiagram-v1-insecure.png) _Starting architecture diagram_

#### Expected user flow:
- Clients will invoke a public-facing web service to pull free recipes.
- The web service is hosted by an HTTP load balancer listening on port 80.
- The web service is forwarding requests to the web application instance which
  listens on port 5000.
- The web application instance will, in turn, use the public-facing AWS API to
  pull recipe files from the S3 bucket hosting free recipes. An IAM role and
  policy will provide the web app instance permissions required to access
  objects in the S3 bucket.
- Another S3 bucket is used as a vault to store secret recipes; there are
  privileged users who would need access to this bucket. The web application
  server does not need access to this bucket.

#### Attack flow:
- Scripts simulating an attack will be run from a separate instance which is in an un-trusted subnet.
- The scripts will attempt to break into the web application instance using the public IP and attempt to access data in the secret recipe S3 bucket.

### Task 2: Review CloudFormation Template

#### VPC Stack for the underlying network:
- A VPC with 2 public subnets, one private subnet, and internet gateways etc for internet access
- See [c3-vpc.yml](cfn/c3-vpc.yml)

#### S3 bucket stack:
- 2 S3 buckets that will contain data objects for the application
- See [c3-s3.yml](cfn/c3-s3.yml)

#### Application stack:
- An EC2 instance that will act as an external attacker from which we will test the ability of our environment to handle threats
- An EC2 instance that will be running a simple web service
- Application LoadBalancer
- Security groups
- IAM role
- See [c3-app.yml](cfn/c3-app.yml)

### Task 3: Deployment of Initial Infrastructure

The objective is to deploy the CloudFormation stacks that will create the above environment.
 
The AMIs specified in the CloudFormation template exist in the us-east-1 (N. Virginia) region. It is necessary to set this as default region when deploying resources for this project.

#### 1. Deploy the infrastructure

##### Deploy the S3 buckets
```
aws cloudformation create-stack --region us-east-1 --stack-name c3-s3 --template-body file://starter/c3-s3.yml
```
 
Expected example output:
```
{
    "StackId": "arn:aws:cloudformation:us-east-1:4363053XXXXXX:stack/c3-s3/70dfd370-2118-11ea-aea4-12d607a4fd1c"
}
```

##### Deploy the VPC and Subnets
```
aws cloudformation create-stack --region us-east-1 --stack-name c3-vpc --template-body file://starter/c3-vpc.yml
```
 
Expected example output:
```
{
    "StackId": "arn:aws:cloudformation:us-east-1:4363053XXXXXX:stack/c3-vpc/70dfd370-2118-11ea-aea4-12d607a4fd1c"
}
```
 
##### Deploy the Application Stack 
Specify a pre-existing key-pair name:

```
aws cloudformation create-stack --region us-east-1 --stack-name c3-app --template-body file://starter/c3-app.yml --parameters ParameterKey=KeyPair,ParameterValue=<add key pair name> --capabilities CAPABILITY_IAM
```
 
Expected example output:
```
{
    "StackId": "arn:aws:cloudformation:us-east-1:4363053XXXXXX:stack/c3-app/70dfd370-2118-11ea-aea4-12d607a4fd1c"
}
```
 
#### 2. Upload data to S3 buckets
Upload the free recipes to the free recipe S3 bucket:  
```
aws s3 cp free_recipe.txt s3://<BucketNameRecipesFree>/ --region us-east-1
```
 
Upload the secret recipes to the secret recipe S3 bucket:  
```
aws s3 cp secret_recipe.txt s3://<BucketNameRecipesSecret>/ --region us-east-1
```

#### 3. Test the application
Invoke the web service using the application load balancer URL: `http://<ApplicationURL>/free_recipe`

### Task 4:  Identify Bad Practices

The architecture has several obvious poor practices as it relates to security:

1. WebAppSG: Even though the Recipe Web Service is behind an Application Load Balancer, its security group permits ingress traffic from the internet to ports 22, 5000 and 80 exposing the server to attacks. It also allows all egress traffic to any IP address.
2. AppLoadBalancerSG: The security group used by the Application Load Balancer is using port 80 (HTTP) instead of 443 (HTTPS).
3. RecipeWebServiceInstance: The Recipe Web Service Instance is placed behind an Application Load Balancer but still located in a public subnet.
4. S3BucketRecipesSecret: There is no encryption configured for the S3 bucket, allowing anyone with access to the account to read it.
5. InstanceRole, InstanceProfileRole and InstanceRolePolicy-C3: The IAM instance profile role used by the web service does not restrict the granted S3 actions and thus violates the least privilege principle.

See [E1T4.txt](answers/E1T4.txt).

## Requirements

Graded according to the [Project Rubric](https://review.udacity.com/#!/rubrics/2800/view).

## License

- **[MIT license](http://opensource.org/licenses/mit-license.php)**
- Copyright 2020 © [Thomas Weibel](https://github.com/thom).