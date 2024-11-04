# Images

Each microservice version is mapped to exactly one image. By providing different configurations we can define different behaviors based on:

- Customer (acarpia, fidital...)
- Environment (uat, prod...)

**`TODO:`** Define a convention to pass and name configurations.

## Repository

AWS ECR repositories can be configured to have all-immutable tags (latest included) or al mutable. To ease the configuration we'll use a single, mutable repo for each service, instead of diving production and developing environments in 2 repositories. semantic versioning and immutability will be managed at the devops level instead. (Note: docker hub for instance does not have this behavior, everything is mutable).

**`TODO:`** Each image should be in a namespace like `fididoc`, refactor old images.

## How to push an image to the ecr registry manually

Make sure that you have the latest version of the AWS CLI and Docker installed. For more information, see [Getting Started with Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html).

Use the following steps to authenticate and push an image to your repository. For additional registry authentication methods, including the Amazon ECR credential helper, see [Registry Authentication](https://docs.aws.amazon.com/AmazonECR/latest/userguide/Registries.html#registry_auth).

Replace `$REPOSITORY_NAME` with the name of the repository, e.g. `fididoc-import-be`.

0. **Make sure you have configured authentication in aws cli. A quick and dirty way is generating an access token from the iam panel.**

    ```sh
    $ aws configure
    AWS Access Key ID [None]: accesskey
    AWS Secret Access Key [None]: secretkey
    Default region name [None]: us-west-2
    Default output format [None]:
    ```

1. **Retrieve an authentication token and authenticate your Docker client to your registry. Use the AWS CLI:**

    ```sh
    aws ecr get-login-password --region eu-south-1 | docker login --username AWS --password-stdin 305653446094.dkr.ecr.eu-south-1.amazonaws.com
    ```

    *Note: If you receive an error using the AWS CLI, make sure that you have the latest version of the AWS CLI and Docker installed.*

2. **Build your Docker image using the following command. For information on building a Docker file from scratch see the instructions [here](https://docs.docker.com/engine/reference/builder/). You can skip this step if your image is already built:**

    ```sh
    docker build -t $REPOSITORY_NAME .
    ```

3. **After the build completes, tag your image so you can push the image to this repository:**

    ```sh
    docker tag $REPOSITORY_NAME:latest 305653446094.dkr.ecr.eu-south-1.amazonaws.com/$REPOSITORY_NAME:latest
    ```

4. **Run the following command to push this image to your newly created AWS repository:**

    ```sh
    docker push 305653446094.dkr.ecr.eu-south-1.amazonaws.com/$REPOSITORY_NAME:latest
    ```

### Copy-Past script

`$APP_NAME` and `$APP_VERSION` must match the one present in the pom.xml.

```sh
# Build the Docker image and assign multiple tags
docker build --build-arg APP_NAME=$APP_NAME \
             --build-arg APP_VERSION=$APP_VERSION \
             -t $REPOSITORY_NAME:$VERSION \
             -t $REPOSITORY_NAME:latest \
             -t 305653446094.dkr.ecr.eu-south-1.amazonaws.com/$REPOSITORY_NAME:$VERSION \
             -t 305653446094.dkr.ecr.eu-south-1.amazonaws.com/$REPOSITORY_NAME:latest .

# Push each tag to ECR
docker push 305653446094.dkr.ecr.eu-south-1.amazonaws.com/$REPOSITORY_NAME:$VERSION
docker push 305653446094.dkr.ecr.eu-south-1.amazonaws.com/$REPOSITORY_NAME:latest


docker build --build-arg APP_NAME=fididocimport-be \
             --build-arg APP_VERSION=1.0.0-SNAPSHOT \
             -t fididoc/fididoc-import-be:1.0.0-SNAPSHOT \
             -t fididoc/fididoc-import-be:latest \
             -t 305653446094.dkr.ecr.eu-south-1.amazonaws.com/fididoc/fididoc-import-be:1.0.0-SNAPSHOT \
             -t 305653446094.dkr.ecr.eu-south-1.amazonaws.com/fididoc/fididoc-import-be:latest .

# Push each tag to ECR
docker push 305653446094.dkr.ecr.eu-south-1.amazonaws.com/fididoc/fididoc-import-be:1.0.0-SNAPSHOT
docker push 305653446094.dkr.ecr.eu-south-1.amazonaws.com/fididoc/fididoc-import-be:latest
```
