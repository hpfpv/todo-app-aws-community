![cover.png](https://github.com/hpfpv/todo-app-aws/blob/main/blog-post/cover.png)

## Étapes de déploiement

### Backend

**Mettre a jour la region AWS dans les functions**

```
REGION="REPLACE_ME_AWS_REGION"
sed -i '.old' "s/REPLACE_ME_AWS_REGION/${REGION}/g" backend/main-service/functions/*.py
sed -i '.old' "s/REPLACE_ME_AWS_REGION/$REGION/g" backend/attachments-service/functions/*.py

```

**Mettre a jour l'URL de l'application dans les functions (CORS)**

```
URL="REPLACE_ME_APP_URL"
sed -i '.old' "s/REPLACE_ME_APP_URL/$URL/g" backend/main-service/functions/*.py
sed -i '.old' "s/REPLACE_ME_APP_URL/$URL/g" backend/attachments-service/functions/*.py

```

**Mettre a jour l'URL de l'application dans le template CloudFormation (CORS)**

```
sed -i '.old' "s/REPLACE_ME_APP_URL/$URL/g" backend/main-service/template.yaml
sed -i '.old' "s/REPLACE_ME_APP_URL/$URL/g" backend/attachments-service/template.yaml

```

**Mentionner le nom du stack CloudFormation**

```
MAIN_STACK_NAME="todo-app-main-stack"
sed -i '.old' "s/REPLACE_ME_MAIN_STACK_NAME/$MAIN_STACK_NAME/g" backend/main-service/template.yaml
sed -i '.old' "s/REPLACE_ME_MAIN_STACK_NAME/$MAIN_STACK_NAME/g" backend/attachments-service/template.yaml

FILES_STACK_NAME="todo-app-attachments-stack"
sed -i '.old' "s/REPLACE_ME_FILES_STACK_NAME/$FILES_STACK_NAME/g" backend/main-service/template.yaml
sed -i '.old' "s/REPLACE_ME_FILES_STACK_NAME/$FILES_STACK_NAME/g" backend/attachments-service/template.yaml

```

**Déployer le main service stack**

```
cd backend/main-service
sam build -t template.yaml 
sam deploy --guided

```

**Déployer le attachement service stack**

```
cd ../attachments-service
sam build -t template.yaml 
sam deploy --guided --capabilities CAPABILITY_NAMED_IAM

```
> Capabilities **CAPABILITY_NAMED_IAM** pour la création de roles et polices IAM avec des noms definis dans le stack CloudFormation.

**Retirer les commentaires des lignes 88-89 et 111-114 dans le main service template et déployer le stack à nouveau**


### Frontend

**Créer le bucket S3 pour servir de site web static**

```
aws s3 mb s3://todo-app-web-aug-2708 --region=$REGION

```

**Remplacer tous les champs necessaires dans le fichier script.js (voir CloudFormation Outputs)**

Pour obtenir les Outputs des stacks sans aller dans la console AWS:

```
aws cloudformation describe-stacks --stack-name $MAIN_STACK_NAME --region $REGION > backend/main-service/main-output.json
aws cloudformation describe-stacks --stack-name $FILES_STACK_NAME --region $REGION > backend/attachments-service/attachements-output.json

```

**Copier le contenu du dossier frontend dans le bucket s3**

```
aws s3 cp frontend s3://todo-app-web-aug-2708 --recursive --exclude "*.DS_Store" --region=$REGION

```

**AWS Console - Créer une distribution CloudFront, OAI, SSL certificate**


## Pipeline CI/CD - GitHub Actions

**Créer un utilisateur IAM avec les bonnes permissions sur les resources à déployer/mettre à jour**

```
AWS_ACCOUNT_ID=REPLACE_ME_ACCOUNT_ID

# Créer l'utilisateur
aws iam create-user --user-name github-serverless-aug

# Associer la permission Administrator (non recommandé - toujours utiliser le moins de permissions requises)
aws iam attach-user-policy --user-name github-serverless-aug --policy-arn "arn:aws:iam::aws:policy/AdministratorAccess"

# Créer un Access Key pour l'utilisateur
aws iam create-access-key --user-name github-serverless-aug > access-key.json

# !!! Supprimer le fichier contenant le Access Key après utilisation

```