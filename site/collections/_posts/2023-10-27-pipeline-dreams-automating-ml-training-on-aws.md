---
date: 2023-10-27 00:00:00
title: 'Pipeline Dreams: Automating ML Training on AWS'
description: Train faster!
tags:
  - aws
  - data-science
  - machine-learning
  - python
image: /uploads/1-htzwrgy9e32fit9qwvcw7q.webp
---
<div><div><div><p>In the world of machine learning, automated training pipelines streamline the journey from data to insight. They automate various parts of the machine learning life cycle such as data ingestion, preprocessing, model training, evaluation and deployment. Amazon Web Services (“AWS”) provides various tools to develop an automated training pipeline. In this article, we will walk through setting up a basic automated training pipeline using using the classic iris dataset.</p><h1>Setting the Stage: Requirements and AWS Toolkit</h1><hr /><p>In this section, we will cover some high level requirements as well as a brief overview of the AWS tools we will use.</p></div></div></div>

<div><div><div><h2>Requirements</h2><p>If you choose to follow along by building your own training pipeline, you will need the following.</p><ul><li>An active AWS account (you can sign up&nbsp;<a rel="noopener ugc nofollow" target="_blank" href="https://aws.amazon.com/free/?trk=78b916d7-7c94-4cab-98d9-0ce5e648dd5f&amp;sc_channel=ps&amp;ef_id=CjwKCAjws9ipBhB1EiwAccEi1K7ugfy2KiyndD6pN_A9_pVTNfC-K3Xp41WNOcLEEDoPj1Lyu_7WHRoCtjEQAvD_BwE%3AG%3As&amp;s_kwcid=AL%214422%213%21432339156165%21e%21%21g%21%21aws+account%219572385111%21102212379047">here</a>) with Administrator Access</li><li>Basic knowledge of&nbsp;<strong>AWS CLI (</strong>We will explore alternatives to the AWS CLI in future posts)</li></ul><p>Setting up your AWS account and connecting to AWS via the CLI is beyond the scope of this post, however- feel free to reach out directly if you need help.</p><h2>Toolkit</h2><p>Setting up the automated training pipeline will require the use of the following AWS products.</p><ul><li><strong>S3</strong>: Scalable object storage service designed to store and retrieve any amount of data from anywhere on the web</li><li><strong>Lambda</strong>: Serverless compute service that automatically runs your code in response to events, such as changes to data in an Amazon S3 bucket</li><li><strong>Docker:&nbsp;</strong>Docker is a platform that packages, distributes, and manages applications inside lightweight, portable containers</li><li><strong>Sagemaker</strong>: Fully managed service that provides developers and data scientists with the ability to build, train, and deploy machine learning models quickly and easily</li><li><strong>Step Functions</strong>: Serverless workflow service that lets you coordinate distributed applications and microservices using visual workflows, enabling you to build, run, and visualize complex processes at scale</li></ul></div></div></div>

<div> </div>

<div><div><div><h1>Implementing the Automated Training Pipeline</h1><hr /><p> </p></div></div></div>

<div> </div>

<div><p>Assuming you made it past the requirements, we can switch our focus to building our automated training pipeline. For this simple example, we will focus on the following steps.</p><ol><li>Create AWS S3 bucket to store data and artifacts related to our training pipeline.</li><li>Create AWS Lambda function data ingestion, preprocessing, and training.</li><li>Create an AWS Step Functions state machine to orchestrate the execution of your pipeline stages.</li></ol><p>We take a deep dive into each of these steps below.</p><h2>S3 Bucket</h2><p>The first thing we will do is create a new S3 bucket to store data and artifacts using the AWS CLI.</p><pre data-language="shell"><code class="language-shell">aws s3api create-bucket
&nbsp; --bucket automated-training-pipeline
&nbsp; --region us-east-1</code></pre><p>If this command runs successfully, it should output something like this in your terminal.</p><pre data-language="json"><code class="language-json">{
&nbsp; &nbsp; "Location": "/automated-training-pipeline"
}</code></pre><p>Taking a look at our management console, we can see the S3 bucket was successfully created.</p><div><div> </div></div><h2>Lambda Functions</h2><p> </p><p>For our automated training pipeline, we will rely heavily on the use of lambda functions to execute and trigger certain parts of our process. Specifically, we will use lambdas for:</p><ul><li>Data ingestion</li><li>Data preprocessing</li><li>Model training</li></ul><h3><strong><em>Data Ingestion</em></strong></h3><p>Lambda functions require lambda handlers. The Lambda handler is a user-defined function in a Lambda deployment package that the AWS Lambda service can invoke when the service executes the Lambda function. The handler function receives and processes the event data from the invoker.</p><p>With that in mind, let’s define our&nbsp;<code>lambda_handler</code>&nbsp;for data ingestion. The&nbsp;<code>lambda_handler</code>&nbsp;function serves as the entry point for AWS Lambda to execute the code, which in this case retrieves a CSV file from a specified URL and uploads it to an Amazon S3 bucket. This file has been stored in&nbsp;<code>./src/data</code>.</p><pre data-language="python"><code class="language-python">import boto3
import requests

def lambda_handler(event, context):
&nbsp; &nbsp; s3 = boto3.client('s3')
&nbsp; &nbsp; data_url = 'https://example.com/dataset.csv'
&nbsp; &nbsp; response = requests.get(data_url)
&nbsp; &nbsp;
&nbsp; &nbsp; # Assuming the data is small enough to fit into memory
&nbsp; &nbsp; s3.put_object(
&nbsp; &nbsp; &nbsp; &nbsp; Bucket='your-s3-bucket',
&nbsp; &nbsp; &nbsp; &nbsp; Key='data/dataset.csv',
&nbsp; &nbsp; &nbsp; &nbsp; Body=response.content
&nbsp; &nbsp; )</code></pre><p>We will package and deploy your AWS Lambda function as a container image using docker. To do so, we first need to create an ECR repository to house our image.</p><pre data-language="shell"><code class="language-shell">aws ecr create-repository --repository-name fetch-iris-data --region us-east-1</code></pre><p>Now that we have our ECR repository stood up, we create our&nbsp;<code>Dockerfile</code>&nbsp;in the root of our project directory.</p><pre data-language="docker"><code class="language-docker"># Use a base image with Python 3.11
FROM public.ecr.aws/lambda/python:3.11

# Install dependencies (if any)
RUN pip install requests boto3

# Copy your function code from the src/data directory to the Lambda task root directory
COPY src/data/fetch_iris_data.py ${LAMBDA_TASK_ROOT}

# Set the CMD to your handler (could also be done as a parameter override outside of the Dockerfile)
CMD ["fetch_iris_data.lambda_handler"]</code></pre><p>We use this Dockerfile to build our image and push it to the ECR registry.</p><pre data-language="shell"><code class="language-shell">docker build -t fetch-iris-data .</code></pre><p>We need to authenticate Docker to the ECR registry.</p><pre data-language="shell"><code class="language-shell">aws ecr get-login-password --region region | docker login --username AWS --password-stdin &lt;your-account-id&gt;.dkr.ecr.&lt;region&gt;.amazonaws.com</code>
</pre><p>Next, we tag the image to match the repository.</p><pre data-language="shell"><code class="language-shell">docker tag fetch-iris-data:latest &lt;your-account-id&gt;.dkr.ecr.&lt;region&gt;.amazonaws.com/fetch-iris-data:latest</code>
</pre><p>Now, we can push our image to the ECR registry.</p><pre data-language="shell"><code class="language-shell">docker push &lt;your-account-id&gt;.dkr.ecr.&lt;region&gt;.amazonaws.com/fetch-iris-data:latest</code></pre><p>Finally, we will create the lambda function and attach the container.</p><pre data-language="shell"><code class="language-shell">aws lambda create-function \
&nbsp; &nbsp; --function-name &lt;function-name&gt; \
&nbsp; &nbsp; --package-type Image \
&nbsp; &nbsp; --code ImageUri=&lt;your-account-id&gt;.dkr.ecr.&lt;region&gt;.amazonaws.com/&lt;repository-name&gt;:&lt;tag&gt; \
&nbsp; &nbsp; --role arn:aws:iam::&lt;your-account-id&gt;:role/&lt;role-name&gt; \
&nbsp; &nbsp; --region &lt;region&gt;</code></pre><p>You will now be able to see the lambda function in your AWS management console. I can invoke the function and confirm the file was downloaded to the S3 bucket correctly.</p><pre data-language="shell"><code class="language-shell">aws lambda invoke --function-name fetch-iris-data-function --payload '{}'

aws s3 ls s3://automated-training-pipeline/data/

#2023-10-24 13:52:41&nbsp; &nbsp; &nbsp; 4551 iris.data</code></pre><h3><strong><em>Preprocessing</em></strong></h3><p>Now, we have the data we need for our modeling task. For most machine learning tasks, we will want to process that data in some way. In the file below, we define a script that will process our Iris data.</p><pre data-language="python"><code class="language-python">import argparse
import os
import pandas as pd
from sklearn.preprocessing import StandardScaler, LabelEncoder
from sklearn.model_selection import train_test_split


def preprocess(input_data_path, output_data_path):
&nbsp; &nbsp; column_names = [
&nbsp; &nbsp; &nbsp; &nbsp; "sepal_length",
&nbsp; &nbsp; &nbsp; &nbsp; "sepal_width",
&nbsp; &nbsp; &nbsp; &nbsp; "petal_length",
&nbsp; &nbsp; &nbsp; &nbsp; "petal_width",
&nbsp; &nbsp; &nbsp; &nbsp; "species",
&nbsp; &nbsp; ]
&nbsp; &nbsp; df = pd.read_csv(input_data_path, header=None, names=column_names)

&nbsp; &nbsp; df = df.dropna(subset=["species"])

&nbsp; &nbsp; encoder = LabelEncoder()
&nbsp; &nbsp; df["species"] = encoder.fit_transform(df["species"])

&nbsp; &nbsp; train, test = train_test_split(df, test_size=0.2, random_state=42)

&nbsp; &nbsp; mean_train = train.mean()

&nbsp; &nbsp; train.fillna(mean_train, inplace=True)
&nbsp; &nbsp; test.fillna(mean_train, inplace=True)

&nbsp; &nbsp; scaler = StandardScaler()
&nbsp; &nbsp; scaler.fit(train[["sepal_length", "sepal_width", "petal_length", "petal_width"]])

&nbsp; &nbsp; train[
&nbsp; &nbsp; &nbsp; &nbsp; ["sepal_length", "sepal_width", "petal_length", "petal_width"]
&nbsp; &nbsp; ] = scaler.transform(
&nbsp; &nbsp; &nbsp; &nbsp; train[["sepal_length", "sepal_width", "petal_length", "petal_width"]]
&nbsp; &nbsp; )
&nbsp; &nbsp; test[
&nbsp; &nbsp; &nbsp; &nbsp; ["sepal_length", "sepal_width", "petal_length", "petal_width"]
&nbsp; &nbsp; ] = scaler.transform(
&nbsp; &nbsp; &nbsp; &nbsp; test[["sepal_length", "sepal_width", "petal_length", "petal_width"]]
&nbsp; &nbsp; )

&nbsp; &nbsp; train.to_csv(
&nbsp; &nbsp; &nbsp; &nbsp; os.path.join(output_data_path, "train_data.csv"), header=False, index=False
&nbsp; &nbsp; )
&nbsp; &nbsp; test.to_csv(
&nbsp; &nbsp; &nbsp; &nbsp; os.path.join(output_data_path, "test_data.csv"), header=False, index=False
&nbsp; &nbsp; )


if __name__ == "__main__":
&nbsp; &nbsp; parser = argparse.ArgumentParser()
&nbsp; &nbsp; parser.add_argument("--input-data", type=str)
&nbsp; &nbsp; parser.add_argument("--output-data", type=str)
&nbsp; &nbsp; args = parser.parse_args()

&nbsp; &nbsp; preprocess(args.input_data, args.output_data)</code></pre><p>We upload our preprocessing script to S3. Run this command in the same directory as&nbsp;<code>preprocess.py</code></p><pre data-language="shell"><code class="language-shell">aws s3 cp preprocess.py s3://automated-training-pipeline/modeling/preprocess.py</code>
</pre><p>Similar to how we built an image for the data ingestion step, we will build an image for the preprocessing step. Like before, let’s start by creating our repository.</p><pre data-language="shell"><code class="language-shell">aws ecr create-repository --repository-name preprocess-iris-data --region &lt;region&gt;</code>
</pre><p>Now that we have our ECR repository stood up, we create our&nbsp;<code>Dockerfile</code>&nbsp;in the root of our project directory.</p><pre data-language="docker"><code class="language-docker"># Use a standard Python base image
FROM python:3.11-slim-buster

# Install necessary libraries: requests, boto3, scikit-learn, pandas, numpy, etc.
RUN pip install requests boto3 scikit-learn pandas numpy

# No need to copy any Python script here since it will be provided from S3 at runtime

# Set working directory
WORKDIR /app

# Set a default CMD or ENTRYPOINT in case you want to run the container for testing, but this isn't strictly necessary
# since the Processing job will override this with ContainerEntrypoint from your Lambda function
CMD ["python", "-c", "print('Container started successfully')"]</code></pre><p>We build the image.</p><pre data-language="shell"><code class="language-shell">docker build -t preprocess-iris-data .</code></pre><p>Let’s authenticate Docker to the ECR registry. You might not have to do this if your access hasn’t expired.</p><pre data-language="shell"><code class="language-shell">aws ecr get-login-password --region region | docker login --username AWS --password-stdin &lt;your-account-id&gt;.dkr.ecr.&lt;region&gt;.amazonaws.com</code></pre><p>Again, we tag the image to match the repository.</p><pre data-language="shell"><code class="language-shell">docker tag preprocess-iris-data:latest &lt;your-account-id&gt;.dkr.ecr.&lt;region&gt;.amazonaws.com/preprocess-iris-data:latest</code>
</pre><p>Now, we can push our image to the ECR registry.</p><pre data-language="shell"><code class="language-shell">docker push &lt;your-account-id&gt;.dkr.ecr.&lt;region&gt;.amazonaws.com/preprocess-iris-data:latest</code>
</pre><p>Like before, we will create a lambda function that will kick off a SageMaker processing job. We define our lambda handler below.</p><pre data-language="python"><code class="language-python">import boto3
import datetime


def lambda_handler(event, context):
&nbsp; &nbsp; sagemaker_client = boto3.client("sagemaker")

&nbsp; &nbsp; # Parameters for the processing job
&nbsp; &nbsp; job_name = "iris-preprocessing-job-" + datetime.datetime.now().strftime(
&nbsp; &nbsp; &nbsp; &nbsp; "%Y%m%d%H%M%S"
&nbsp; &nbsp; )
&nbsp; &nbsp; role_arn = "arn:aws:iam::&lt;your-account-id&gt;:role/service-role/&lt;role&gt;"
&nbsp; &nbsp; image_uri = (
&nbsp; &nbsp; &nbsp; &nbsp; "&lt;your-account-id&gt;.dkr.&lt;region&gt;.amazonaws.com/preprocess-iris-data:latest"
&nbsp; &nbsp; )
&nbsp; &nbsp; input_s3_uri = "s3://automated-training-pipeline/data/iris.data"
&nbsp; &nbsp; preprocess_s3_uri = "s3://automated-training-pipeline/modeling/preprocess.py"
&nbsp; &nbsp; output_s3_uri = "s3://automated-training-pipeline/data/"

&nbsp; &nbsp; response = sagemaker_client.create_processing_job(
&nbsp; &nbsp; &nbsp; &nbsp; ProcessingJobName=job_name,
&nbsp; &nbsp; &nbsp; &nbsp; RoleArn=role_arn,
&nbsp; &nbsp; &nbsp; &nbsp; ProcessingInputs=[
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "InputName": "input-1",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "S3Input": {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "S3Uri": input_s3_uri,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "LocalPath": "/opt/ml/processing/input",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "S3DataType": "S3Prefix",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "S3InputMode": "File",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "S3DataDistributionType": "FullyReplicated",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "S3CompressionType": "None",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "InputName": "code",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "S3Input": {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "S3Uri": preprocess_s3_uri,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "LocalPath": "/opt/ml/processing/code",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "S3DataType": "S3Prefix",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "S3InputMode": "File",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "S3DataDistributionType": "FullyReplicated",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "S3CompressionType": "None",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; &nbsp; ],
&nbsp; &nbsp; &nbsp; &nbsp; ProcessingOutputConfig={
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "Outputs": [
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "OutputName": "output-1",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "S3Output": {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "S3Uri": output_s3_uri,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "LocalPath": "/opt/ml/processing/output",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "S3UploadMode": "EndOfJob",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ]
&nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; &nbsp; ProcessingResources={
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "ClusterConfig": {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "InstanceCount": 1,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "InstanceType": "ml.t3.medium",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "VolumeSizeInGB": 5,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; &nbsp; AppSpecification={
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "ImageUri": image_uri,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "ContainerArguments": [
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "--input-data",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "/opt/ml/processing/input/iris.data",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "--output-data",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "/opt/ml/processing/output",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ],
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "ContainerEntrypoint": ["python3", "/opt/ml/processing/code/preprocess.py"],
&nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; )

&nbsp; &nbsp; return {"statusCode": 200, "body": response}</code></pre><p>We zip up our Lambda handler and store in S3.</p><pre data-language="shell"><code class="language-shell">zip lambda_function_preprocess.zip lambda_handler_preprocess.py

aws s3 cp lambda_function_preprocess.zip s3://automated-training-pipeline/modeling/lambda_function_preprocess.zip</code></pre><p>Finally, we create our lambda function.</p><pre data-language="shell"><code class="language-shell">aws lambda create-function \
&nbsp; &nbsp; --function-name iris-preprocessing-trigger \
&nbsp; &nbsp; --runtime python3.11 \
&nbsp; &nbsp; --role arn:aws:iam::&lt;your-account-id&gt;:role/&lt;role&gt;\
&nbsp; &nbsp; --handler lambda_handler_preprocess.lambda_handler \
&nbsp; &nbsp; --code S3Bucket=automated-training-pipeline,S3Key=modeling/lambda_function_preprocess.zip \
&nbsp; &nbsp; --memory-size 256 \
&nbsp; &nbsp; --timeout 900 </code></pre><p>Since we haven’t strung the Lambda functions together yet, we can manually invoke the lambda and check the contents of S3 to ensure our training and testing datasets are there.</p><pre data-language="shell"><code class="language-shell">aws lambda invoke --function-name iris-preprocessing-trigger --payload '{}'

aws s3 ls s3://automated-training-pipeline/data/

#2023-10-24 14:50:46&nbsp; &nbsp; &nbsp; 4551 iris.data
#2023-10-24 17:08:42&nbsp; &nbsp; &nbsp; 2404 test_data.csv
#2023-10-24 17:08:42&nbsp; &nbsp; &nbsp; 9635 train_data.csv</code></pre><h3><strong><em>Model training</em></strong></h3><p>Instead of creating our own image for train, we will use a built SageMaker image for logistic regression. You can create your own image if you want a customized training process. Consequently, we write our lambda handler right away.</p><pre data-language="python"><code class="language-python">import boto3
import datetime


def lambda_handler(event, context):
&nbsp; &nbsp; sagemaker_client = boto3.client("sagemaker")

&nbsp; &nbsp; job_name = "iris-training-job-" + datetime.datetime.now().strftime("%Y%m%d%H%M%S")
&nbsp; &nbsp; role_arn = "arn:aws:iam::&lt;your-account-id&gt;:role/service-role/&lt;role&gt;"&nbsp; # Update the ARN as required

&nbsp; &nbsp; image_uri = "382416733822.dkr.ecr.us-east-1.amazonaws.com/linear-learner:1" # Public learner

&nbsp; &nbsp; training_data_s3_uri = "s3://automated-training-pipeline/data/train_data.csv"
&nbsp; &nbsp; output_s3_uri = "s3://automated-training-pipeline/models/"

&nbsp; &nbsp; response = sagemaker_client.create_training_job(
&nbsp; &nbsp; &nbsp; &nbsp; TrainingJobName=job_name,
&nbsp; &nbsp; &nbsp; &nbsp; RoleArn=role_arn,
&nbsp; &nbsp; &nbsp; &nbsp; AlgorithmSpecification={
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "TrainingImage": image_uri,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "TrainingInputMode": "File",
&nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; &nbsp; InputDataConfig=[
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "ChannelName": "train",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "DataSource": {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "S3DataSource": {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "S3DataType": "S3Prefix",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "S3Uri": training_data_s3_uri,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "S3DataDistributionType": "FullyReplicated",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "ContentType": "text/csv",&nbsp; # Specify that we're using CSV format
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; &nbsp; ],
&nbsp; &nbsp; &nbsp; &nbsp; OutputDataConfig={
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "S3OutputPath": output_s3_uri,
&nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; &nbsp; ResourceConfig={
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "InstanceCount": 1,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "InstanceType": "ml.m5.large",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "VolumeSizeInGB": 5,
&nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; &nbsp; HyperParameters={
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "predictor_type": "multiclass_classifier",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "num_classes": "3",
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "mini_batch_size": "30",
&nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; &nbsp; StoppingCondition={
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "MaxRuntimeInSeconds": 3600,
&nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; )

&nbsp; &nbsp; model_artifact_uri = f"{output_s3_uri}/{job_name}/output/model.tar.gz"

&nbsp; &nbsp; return {
&nbsp; &nbsp; &nbsp; &nbsp; "statusCode": 200,
&nbsp; &nbsp; &nbsp; &nbsp; "body": {"trainingJobName": job_name, "modelArtifactUri": model_artifact_uri},
&nbsp; &nbsp; }</code></pre><p>Before we can create our lambda function, we have to zip up our handler and send it to S3.</p><pre data-language="shell"><code class="language-shell">zip lambda_function_train.zip lambda_handler_train.py

aws s3 cp lambda_function_train.zip s3://automated-training-pipeline/modeling/lambda_function_train.zip</code></pre><p>Now we can create our lambda function.</p><pre data-language="shell"><code class="language-shell">aws lambda create-function \&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;
&nbsp; &nbsp; --function-name iris-training-trigger \
&nbsp; &nbsp; --runtime python3.11 \
&nbsp; &nbsp; --role arn:aws:iam::&lt;your-account-id&gt;:role/&lt;role&gt; \
&nbsp; &nbsp; --handler lambda_handler_train.lambda_handler \
&nbsp; &nbsp; --code S3Bucket=automated-training-pipeline,S3Key=modeling/lambda_function_train.zip \</code></pre><p>Again, we can manually invoke our training lambda to ensure it works correctly.</p><pre data-language="shell"><code class="language-shell">aws lambda invoke --function-name iris-training-trigger --payload '{}' output.txt

aws s3 ls s3://automated-training-pipeline/models/

#PRE iris-training-job-20231025015943/</code>
</pre><h2>Step Functions</h2><p>Now, we have a very basic pipeline consisting of data ingestion, preprocessing, and model training. However, each of these components exist on their own — how do we string them together to create an automated pipeline. Step functions!</p><p>To string our lambda functions together, the first thing we will do is define a state machine using Amazon States Language and save it in a&nbsp;<code>.json</code>&nbsp;file so we can push it to AWS using the CLI. For our purposes, we will define our state machine in the following way.</p><pre data-language="json"><code class="language-json">{
&nbsp; "Comment": "A simple AWS Step Functions state machine that orchestrates 3 Lambda functions.",
&nbsp; "StartAt": "DataIngestion",
&nbsp; "States": {
&nbsp; &nbsp; "DataIngestion": {
&nbsp; &nbsp; &nbsp; "Type": "Task",
&nbsp; &nbsp; &nbsp; "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:function:data-ingestion-lambda",
&nbsp; &nbsp; &nbsp; "Next": "Preprocessing"
&nbsp; &nbsp; },
&nbsp; &nbsp; "Preprocessing": {
&nbsp; &nbsp; &nbsp; "Type": "Task",
&nbsp; &nbsp; &nbsp; "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:function:preprocessing-lambda",
&nbsp; &nbsp; &nbsp; "Next": "Training"
&nbsp; &nbsp; },
&nbsp; &nbsp; "Training": {
&nbsp; &nbsp; &nbsp; "Type": "Task",
&nbsp; &nbsp; &nbsp; "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:function:training-lambda",
&nbsp; &nbsp; &nbsp; "End": true
&nbsp; &nbsp; }
&nbsp; }
}</code></pre><p>We can deploy our step function to AWS.</p><pre data-language="shell"><code class="language-shell">aws stepfunctions create-state-machine \
&nbsp; &nbsp; --name “automated-training-pipeline“ \
&nbsp; &nbsp; --definition file://pipeline.json \
&nbsp; &nbsp; --role-arn "arn:aws:iam::&lt;your-account-id&gt;:role/&lt;execution role&gt;”</code></pre><p>Once the state machine has been created, we can visualize it in the management console.</p><div><div><img src="/uploads/screenshot-2023-10-27-at-7-36-13-am.png" width="718" height="1150" /></div></div><p>We can manually invoke the pipeline as follows.</p><pre data-language="shell"><code class="language-shell">aws stepfunctions start-execution \
&nbsp; &nbsp; --state-machine-arn "arn:aws:states:us-east-1:&lt;your-account-id&gt;:stateMachine:&lt;YourStateMachineName&gt;"</code></pre><h1>Additional Considerations &amp; Conclusion</h1><hr /><p>The pipeline we built in this example is extremely basic — we don’t even consider model evaluation! In a real world setting, this approach can be expanded to cover additional parts of the model building lifecycle including model evaluation &amp; deployment.</p></div>

<div><div><div><p>Additionally, we built our pipeline but we have to trigger it manually. In the real world, you can trigger the execution of a Step Functions state machine based on various events. Using triggers provides a more event-driven architecture. Here are some common ways to set up triggers for your state machine:</p><ol><li><strong>Amazon S3 Events</strong>: If you want to run your state machine when a new file is uploaded to an S3 bucket, you can set up an event within the bucket to trigger a Lambda function, which in turn starts the execution of your state machine.</li><li><strong>AWS CodeCommit</strong>: If you’re using AWS CodeCommit as your repository, you can use AWS Lambda and Amazon CloudWatch Events to trigger the state machine whenever there’s a new commit.</li><li><strong>GitHub or Other Repositories</strong>: If you’re using a service like GitHub, you can use webhooks to notify an AWS service of a new commit or a pull request merge. Typically, you’d set up an API Gateway endpoint to receive the webhook, trigger a Lambda function from the API Gateway call, and then have the Lambda function start your state machine.</li><li><strong>Amazon DynamoDB Streams</strong>: If you want to trigger your state machine based on changes in a DynamoDB table, you can use DynamoDB Streams. Whenever there’s a change in the table, the stream can trigger a Lambda function, which then starts the state machine.</li><li><strong>AWS EventBridge (previously CloudWatch Events)</strong>: AWS EventBridge allows you to create rules based on a wide range of AWS service events. You can target a Lambda function with these rules. Then, like in other scenarios, the Lambda function would start your state machine.</li></ol><p>In conclusion, an automated training pipeline in the context of machine learning and data science offers multiple advantages. Here’s a concise list of pros for your conclusion:</p><ol><li><strong>Consistency and Reproducibility</strong>: Automation ensures that the training process remains consistent across different runs. This aids in reproducing results and eliminates variations due to manual interventions.</li><li><strong>Efficiency</strong>: Automated pipelines can streamline processes, reducing the time required for training and retraining models. This can be especially beneficial when iterating over different model architectures or parameters.</li><li><strong>Scalability</strong>: As your data or model complexity grows, an automated pipeline can scale up resources and processes without the need for manual oversight.</li><li><strong>Resource Optimization</strong>: Automation can manage resources more effectively, potentially leading to cost savings. For instance, cloud resources can be automatically scaled down when not in use.</li></ol><p>Feel free to reach out with any questions!</p></div></div></div>