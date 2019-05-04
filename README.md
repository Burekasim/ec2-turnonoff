# EC2 Turn OnOff script

AWS Lambda script to start/stop EC2 instances based on EC2 tags. 


## Features

- Turn On and Turn Off instances based on EC2 tag.
- If workweek_tag environment variable set in Lamdba, the script will not turn on servers over the weekend.



## Configuration

Just add a Tags named TurnOn and TurnOff to your relevant EC2 instances with the following format:

`[Hour]:[Minutes]`

Examples:
- `16:30` to perform an action at 4:30PM
- `08:00` to perform an action at 8AM.


## Installation

### Via Terraform
This project is a Terraform module. Reference module by adding the following to your terraform configuration:
```sh
module "ec2-turnonoff" {
  source  = "${path-to-modules}/ec2-turnonoff"
}
```

Initialize module by (re-)initializing your terraform project:
```sh
terraform init
```

Apply changes to terraform configuration:

```sh
terraform plan -out terraform.tfplan
```

Check if everything is ok and then call:

```sh
terraform apply terraform.tfplan
```


### Via CloudFormation

TBD

### Manually

1) Create an IAM role for AWS Lambda and add the following inline policy:

```json
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Effect":"Allow",
         "Action":[
            "logs:CreateLogGroup",
            "logs:CreateLogStream",
            "logs:PutLogEvents"
         ],
         "Resource":"arn:aws:logs:*:*:*"
      },
      {
         "Effect":"Allow",
         "Action":[
            "ec2:DescribeRegions",
            "ec2:DescribeInstances"
         ],
         "Resource":"*"
      },
      {
         "Sid":"VisualEditor0",
         "Effect":"Allow",
         "Action":[
            "ec2:StartInstances"
         ],
         "Resource":"*",
         "Condition":{
            "StringLike":{
               "ec2:ResourceTag/TurnOn":"*"
            }
         }
      },
      {
         "Sid":"VisualEditor1",
         "Effect":"Allow",
         "Action":[
            "ec2:StopInstances"
         ],
         "Resource":"*",
         "Condition":{
            "StringLike":{
               "ec2:ResourceTag/TurnOff":"*"
            }
         }
      }
   ]
}
```

2) Create a Lambda function

* Select blueprint
    * Select "Blank Function"
* Configure triggers
    * Select "CloudWatch Events - Schedule"
    * Choose "cron(0/30 * * * ? *)"
    * Enable trigger
* Configure function
    * Enter name
    * Select "Python 3.7" from the Runtime dropdown
    * Copy the source code from ebs-backup.py to code window
    * Check that Handler is "lambda_function.lambda_handler"
    * Choose the previously created IAM role
     
3) Check the logs

Check the logs in the CloudWatch Logs area.