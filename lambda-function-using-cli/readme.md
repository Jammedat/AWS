The Nautilus DevOps team continues to explore serverless architecture by setting up another Lambda function. This time, the task must be completed using the AWS Console to familiarize the team with the web interface. The function will return a custom greeting and demonstrate the capabilities of AWS Lambda effectively.

Create Python Script: Create a Python script named lambda_function.py with a function that returns the body Welcome to KKE AWS Labs! and status code 200.

Zip the Python Script: Zip the script into a file named function.zip.

Create Lambda Function: Create a Lambda function named devops-lambda-cli using the zipped file and specify Python as the runtime.

IAM Role: Use the IAM role named lambda_execution_role.

### Solution  
1. Create a script  
```bash
import json

def lambda_handler(event, context):
    return {
       'codeStatus': 200
       'body': json.dumps('Welcome to KKE AWS Labs!')
    }
```

2. Zip the script file  
` zip function.zip lambda_functtion.py `


3. Get the account id as  
`aws sts get-caller-identity`
```bash
{
    "UserId": "AIDARN7EM34JJBTV42GDW",
    "Account": "098726371090",
    "Arn": "arn:aws:iam::098726371090:user/kk_labs_user_167551"
}
```

4. Create the function  
```bash
aws lambda create-function \
> --function-name devops-lambda-cli \
> --role arn:aws:iam::098726371090:role/lambda_execution_role \
> --runtime python3.9 \
> --handler lambda_function.lambda_handler \
> --zip-file fileb://function.zip
```