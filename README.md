# Tekton - CI/CD

## Requirements

For this tutorial we need a Kubernetes cluster with an ingress-controller installed that can give us an external IP.

We also need a GitHub repository where we can add the webhook.
Installation

Tekton Triggers requires Tekton Pipelines to be installed. We also need to install the core interceptors (GitHub, GitLab, BitBucket, and CEL) manifests as we’ll use them later on.

By default all resources will be installed in the tekton-pipelines namespace.

### Tekton Namespace
kubectl create -f tekton-pipelines

### Tekton Pipelines
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/previous/v0.23.0/release.yaml

### Tekton Triggers + Interceptors
kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/previous/v0.13.0/release.yaml

kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/previous/v0.13.0/interceptors.yaml

### Configure RBAC for our Tekton Triggers service account:
kubectl apply -f tekton-rbac.yaml

## Creating Resources for Tekton Triggers
For the project we need to create the following resources:

Pipeline: Contains a set of task whose operations are performed when PipelineRun gets triggered or executed.

EventListener: A Kubernetes Service that listens for incoming HTTP requests and executes a Trigger.

Trigger: Decides what to do with the received event. Sets a TriggerBinding, TriggerTemplate and Interceptor to run.

TriggerBinding: Specifies the data to be extracted from the request and saved as parameters. This data will be passed to the TriggerTemplate.

TriggerTemplate: A template of a resource (TaskRun/PipelineRun) to be created when an event is received.

Interceptor: Processes an event to do custom validation or filtering.

Ingress: For GitHub to be able to send a request to our event listener we need to expose it by creating an Ingress resource and pointing it to our event listener service.

## Adding the webhook to Github
In your GitHub repo go to Settings -> Webhooks and click Add Webhook. The fields we need to set are:

Payload URL: Your external IP Address from the Ingress with /hooks path. For an instance: http://10.14.X.X:80/hooks

Content type: application/json

Secret: #######

Under events select Let me select individual events. Uncheck Pushes and check Pull requests.

## Merging the raised PR and testing our trigger

cd linux-tweet-priv/

git status

git add test.txt

git commit -m "Modified test file"

git push

PR raised on master from dev and is merged. PipelineRun should get triggered.
