![cover.png](https://github.com/hpfpv/todo-app-aws/blob/main/blog-post/cover.png)

## Deployment steps

### Backend

**Update AWS region in functions**

```
REGION="REPLACE_ME_AWS_REGION"
sed -i '.old' "s/REPLACE_ME_AWS_REGION/${REGION}/g" backend/main-service/functions/*.py
sed -i '.old' "s/REPLACE_ME_AWS_REGION/$REGION/g" backend/attachments-service/functions/*.py

```

**Update application URL in functions (CORS)**

```
URL="REPLACE_ME_APP_URL"
sed -i '.old' "s/REPLACE_ME_APP_URL/$URL/g" backend/main-service/functions/*.py
sed -i '.old' "s/REPLACE_ME_APP_URL/$URL/g" backend/attachments-service/functions/*.py

```

**Update application URL in CloudFormation templates (CORS)**

```
sed -i '.old' "s/REPLACE_ME_APP_URL/$URL/g" backend/main-service/template.yaml
sed -i '.old' "s/REPLACE_ME_APP_URL/$URL/g" backend/attachments-service/template.yaml

```

**Mention the name of the CloudFormation stack**

```
MAIN_STACK_NAME="todo-app-main-stack"
sed -i '.old' "s/REPLACE_ME_MAIN_STACK_NAME/$MAIN_STACK_NAME/g" backend/main-service/template.yaml
sed -i '.old' "s/REPLACE_ME_MAIN_STACK_NAME/$MAIN_STACK_NAME/g" backend/attachments-service/template.yaml

FILES_STACK_NAME="todo-app-attachments-stack"
sed -i '.old' "s/REPLACE_ME_FILES_STACK_NAME/$FILES_STACK_NAME/g" backend/main-service/template.yaml
sed -i '.old' "s/REPLACE_ME_FILES_STACK_NAME/$FILES_STACK_NAME/g" backend/attachments-service/template.yaml

```

**Deploy the main service stack**

```
cd backend/main-service
sam build -t template.yaml 
sam deploy --guided

```

**Deploy the attachement service stack**

```
cd ../attachments-service
sam build -t template.yaml 
sam deploy --guided --capabilities CAPABILITY_NAMED_IAM

```
> Capabilities **CAPABILITY_NAMED_IAM** for creating IAM roles and policies with names defined in the CloudFormation stack.

**Uncomment lines 88-89 and 111-114 in the main service template and deploy the stack again**


### Frontend

**Create the S3 bucket to serve as a static website**

```
aws s3 mb s3://todo-app-web-aug-2708 --region=$REGION

```

**Replace all necessary fields in the script.js file (see CloudFormation Outputs)**

To get stack Outputs without going to the AWS console:

```
aws cloudformation describe-stacks --stack-name $MAIN_STACK_NAME --region $REGION > backend/main-service/main-output.json
aws cloudformation describe-stacks --stack-name $FILES_STACK_NAME --region $REGION > backend/attachments-service/attachements-output.json

```

**Copy the contents of the frontend folder to the S3 bucket**

```
aws s3 cp frontend s3://todo-app-web-aug-2708 --recursive --exclude "*.DS_Store" --region=$REGION

```

**AWS Console - Create a CloudFront distribution, OAI, SSL certificate**


## CI/CD Pipeline - GitHub Actions

**Create an IAM user with the right permissions set on the resources to deploy/update**

```
AWS_ACCOUNT_ID=REPLACE_ME_ACCOUNT_ID

# Create the user
aws iam create-user --user-name github-serverless-aug

# Associate the Administrator permission (not recommended - always use the least privileges)
aws iam attach-user-policy --user-name github-serverless-aug --policy-arn "arn:aws:iam::aws:policy/AdministratorAccess"

# Create an access key for the user
aws iam create-access-key --user-name github-serverless-aug > access-key.json

# !!! Delete the file containing the Access Key after use

```