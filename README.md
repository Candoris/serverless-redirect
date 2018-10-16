# serverless-redirect
Serverless url redirect using AWS Lambda

For more information on serverless computing, AWS Lambda, CloudFormation and S3. Go to [the bottom](#additional-research). 

Created an API gateway that triggers a Lambda function when it receives requests. API gateway handles the routing and the Lambda handles the backend code.


Original tutorial I followed:
https://moduscreate.com/blog/redirect-requests-to-a-domain-with-aws-lambda/   
Original tutorial's GitHub:
https://github.com/ModusCreateOrg/lambda-redirector

### Step 1:
Write the app_spec yaml file which is a SAM Template. This specifies the location of the lambda function code. It probably is not necessary to have two functions for this example, but it doesn't hurt so I included both. One for handling Get requests and one for handling any other requests. Those functions assemble the path and query params and append it to our new domain. It then combines that with the HTTP status code and sends it back to the API Gateway.We want to link the lambda function to API Gateway so in the SAM template we add the API Gateway as an event that will trigger the lambda function under "Environment:". The "GetCustom:" tag allows any GET requests to / to trigger lambda. "GetProxy" will allow any GET requests to anything other than / to trigger lambda. We can now package and upload the local artifacts to an S3 bucket.

### Step 2:
In order to package and upload our code to an S3 bucket we will need to create one. To do this you need to go to the AWS S3 console and create a bucket with a globally unique name. Once the bucket is created, our code can be packaged and deployed to the bucket. This can be done on the command line with the aws cloudformation package and deploy commands. When packaging, a packaged.yml file is created which is a processed version of the app_spec.yml file. The deploy command creates a CloudFormation stack that we can then view in the AWS CloudFormation console and an API Gateway that we can view in the AWS API Gateway console.

### Step 3:
The SAM template deployed to an API with a url in the format:  
**https://{restapi_id}.execute-api.{region}.amazonaws.com/{stage_name}/**  
To test, we can run a curl or postman request against that url to make sure it's working. It chould return the url we requested and the http resonse (301) for our target url. The api can also be tested with a curl/postman request. This ugly url should now correctly link to our targeted url.

### Step 4:
Now that we have a working re-router url in our AWS API Gateway, we want to connect it to a custom domain name so that we can target our re-router from a specific source url (candoris.com). To do this we will need to create a custom domain name in our API Gateway as well as an ACM (Amazon Certificate Manager) certificate. If there is already an available certificate that points to our source url (candoris.com) we should be able to use that. If there is no existing certificate, we need to visit the AWS certificate manager. We can then request a certificate through Route 53 for our source url via DNS. Once we accept the certificate it will take roughly an hour to verify. Even though we need to wait for it to officially verify, the certificate should now be available to use in the creation of our custom domain name.

### Step 5:
To create a custom domain name we need to go to the AWS API Gateway console and select custom domain names. We can then create a custom domain name that is what we want our source url to be named (my.candoris.com). The name can be filled out and the certificate we just created should show up in the ACM Certificate dropdown. That will then take roughly 40 minutes to initialize so the base path mapping and  an alias can be created. The base path mapping maps our custom url (my.candoris.com) to our redirector API through the route path. Setting up the alias will require the target domain name from the custom domain box. This target domain will be in the format: **distribution-id.cloudfront.net**  
We can then go to the AWS Route 53 console, select the hosted zone for the correct url (candoris.com) and then create a record set for the source url (my.candoris.com) or edit the existing record set. Then set the record type to "A- IPv4 address" and set the alias target equal to the cloudfront.net url from the custom domain info.

### Step 6: 
Once the certificate is verified and the custom domain name finishes initializing, the new custom source url (my.candoris.com) should accurately point to our target url.

## Additional Research

### S3:
Amazon S3 is object storage built to store and retrieve any amount of data from anywhere – web sites and mobile apps, corporate applications, and data from IoT sensors or devices. Amazon S3 runs on the world’s largest global cloud infrastructure, and is designed from the ground up to deliver 99.999999999% of durability. S3 supports security standards and compliance certifications, including PCI-DSS, HIPAA/HITECH, FedRAMP, EU Data Protection Directive, and FISMA. 

### Serverless Computing: 
Serverless computing allows you to build and run applications and services without thinking about servers. Serverless applications don't require you to provision, scale, and manage any servers. You can build them for nearly any type of application or backend service, and everything required to run and scale your application with high availability is handled for you. Building serverless applications means that your developers can focus on their core product instead of worrying about managing and operating servers or runtimes, either in the cloud or on-premises.

Ex:
![Serverless computing example](https://github.com/Candoris/serverless-redirect/blob/master/images/ServerlessComputingExample.png "Serverless computing example")

 
### AWS Lambda:
Goes hand in hand with serverless computing. Lambda is a serverless computing platform that can be triggered by events. AWS Lambda lets you run code without provisioning or managing servers. You pay only for the compute time you consume - there is no charge when your code is not running. With Lambda, you can run code for virtually any type of application or backend service - all with zero administration. Just upload your code and Lambda takes care of everything required to run and scale your code with high availability. You can set up your code to automatically trigger from other AWS services or call it directly from any web or mobile app. You can build serverless backends using AWS Lambda to handle web, mobile, Internet of Things (IoT), and 3rd party API requests.

Ex:
![Lambda Example](https://github.com/Candoris/serverless-redirect/blob/master/images/AWSLambdaExample.png "Lambda Example")


### CloudFormation:
Allows you to model and configure your AWS resources by declaratively describing your resources in a template. CF manages provisioning and configuring these resources. All the resources exist in a YAML or JSON template (IAC – Infrastructure as Code). 

Ex:
![CloudFormation example](https://github.com/Candoris/serverless-redirect/blob/master/images/CloudFormationExample.png "CloudFormation Example")

### AWS SAM (Serverless Application model):
"AWS SAM is CF on crack!" AWS SAM is a model used to define serverless applications on AWS. AWS SAM is based on AWS CloudFormation. A serverless application is defined in a CloudFormation template and deployed as a CloudFormation stack. AWS SAM defines a set of objects which can be included in a CloudFormation template to describe common components of serverless applications easily.  SAM supports special resource types that makes defining resources a lot easier. AWS SAM allows us to include "AWS::Serverless::Function" in the CF template. Also allows us to use the cloudformation command in the AWS CLI to package and deploy artifacts to S3.
