## Building a custom image to train models on SM

### 1 - Build image locally

Run following command to build image locally:

```
docker build -t ngboost-custom-container-test .
```

### 2 - Test Locally [local-notebook](local-notebook.ipynb)

Open Notebook and run all cells to check if it's working locally.

It's necessary to have sagemaker SDK installed locally, if you do not have, please install dependencies from [requirements.txt](requirements.txt) file.

**Hint: You can work with Virtualenv on this step**

### 3 - Push Image to ECR

Run following code to push image to ECR:

```
%%bash

# Specify an algorithm name
algorithm_name=ngboost-custom-container-test

account=$(aws sts get-caller-identity --query Account --output text)

# Get the region defined in the current configuration (default to us-west-2 if none defined)
region=$(aws configure get region)
region=${region:-us-east-1}

fullname="${account}.dkr.ecr.${region}.amazonaws.com/${algorithm_name}:latest"

# If the repository doesn't exist in ECR, create it.

aws ecr describe-repositories --repository-names "${algorithm_name}" > /dev/null 2>&1
if [ $? -ne 0 ]
then
aws ecr create-repository --repository-name "${algorithm_name}" > /dev/null
fi

# Get the login command from ECR and execute it directly

$(aws ecr get-login --region ${region} --no-include-email)

# Build the docker image locally with the image name and then push it to ECR
# with the full name.

docker build -t ${algorithm_name} .
docker tag ${algorithm_name} ${fullname}

docker push ${fullname}
```

### 4 - Train on SageMaker (remotely)

Run [remote-notebook](remote-notebook.ipynb) notebook to train remotely on SageMaker.

