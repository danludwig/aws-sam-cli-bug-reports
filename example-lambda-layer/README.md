# Example Lambda Layer

An example layer to test with the aws sam cli

## Deploying

Update the layer version and increment after each new version deploy.

> sam build --use-container

> sam package --region us-east-2 --s3-bucket danludwig-lambda-code-us-east-2 --s3-prefix lambda-layers/danludwig-example-nodejs --output-template-file ./.aws-sam/build/template_packaged.yaml --profile danludwig

> sam deploy --region us-east-2 --template-file ./.aws-sam/build/template_packaged.yaml --stack-name danludwig-example-nodejs-lambda-layer --capabilities CAPABILITY_IAM --profile danludwig

## Updating Permissions

Update the layer version and increment after each new version deploy.

> aws lambda add-layer-version-permission --layer-name danludwig-example-nodejs-lambda-layer --statement-id public-layer-allow --version-number 1 --principal '*' --action lambda:GetLayerVersion --output text --profile danludwig --region us-east-2
