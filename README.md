# Kubernetes Logging Template

> **Disclaimer**: This repository was written and tested to be used in the future, as a possible reference, if needed. I can't ensure that the code will continue to work as intended with future updates to the tools used. Enjoy 🤓

## Introduction

This repository provides the values to deploy the Helm Charts for Grafana Loki and Vector to create a solution that gathers the logs from the resources running in the Kubernetes cluster, and it delivers them to an S3 bucket on AWS.
Loki will be available as a data source in Grafana to enable engineers to view and filter logs from a dashboard.

## Requirements

The following are the versions of the various tools used:

- Helm - 4.1.0
  - loki - 6.50.0
  - vector - 0.49.0
- Kubectl - 1.34.2

This configuration has been tested on an EKS cluster on AWS, so it is assumed that you have a cluster already set up and working, with a Grafana instance running inside the cluster.

## Structure

The project contains a `loki` and `vector` folders, each with its own `Chart.yaml` and `values.yaml` files.
The reason I've wrapped the upstream Chart with a local one, is because this way it is possible to have a CICD pipeline that builds and packages the image of the Chart in a repository (e.g. ECR on AWS or Artifact Registry on GCP).
This way whatever happens to the upstream Chart we'll have our own version saved in our repository, accessible for our projects.

## Usage

### 1. Deploy Loki

```sh
helm dependency update ./loki
helm install loki ./loki -n loki --create-namespace
kubectl get pods -n loki
```

### 2. Deploy Vector

```sh
helm dependency update ./vector
helm install vector ./vector -n vector --create-namespace
kubectl get pods -n vector
```

### 3. Verify Grafana integration

1. Connect to your Grafana instance.
   1. If you're not sure what's your endpoint, check if you have a service running in the `monitoring` namespace and run a port-forward to a local port in your PC.
   2. You should also be able to recover the login credentials from the secrets created in the same namespace.
2. Add Loki as a data source on Grafana:
   1. On the sidebar, click the "*Connections*" tab, and then “*Add new connection*“.
   2. Search for Loki from the available sources.
   3. Select Loki and then “*Add new data source*“.
   4. In the "*Connection*" field, paste the Loki service endpoint: `http://loki-gateway.loki.svc.cluster.local`.
3. Go to the "Explore" tab and select Loki as data source.
4. Specify a label filter and execute the query.

### 4. Verify S3 integration

On S3 the control is pretty simple.
You should see the bucket with compressed files in paths that depends on the index Loki is configured to create.
If you open those compressed archives, you'll find files with a `.tsdb` extension, so to read them you'll need a compatible software.

#### 4.1 S3 Retention

A consideration on S3 could be to create a Lifecycle Policy to move the files to another storage type in the bucket to keep them for a longer duration than what is configured in Loki so that your business has time to go back and recover interesting logs in case of need.
