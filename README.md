# Tekton Pipeline Example

A simple example repository demonstrating basic Tekton Pipelines concepts.

## Overview

This repository contains a minimal Tekton pipeline setup that demonstrates:
- A simple Task that prints a greeting message
- A Pipeline that uses the Task
- A PipelineRun to execute the Pipeline

## Prerequisites

- Kubernetes cluster (v1.24+)
- Tekton Pipelines installed on your cluster

### Installing Tekton Pipelines

```bash
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

Verify the installation:
```bash
kubectl get pods --namespace tekton-pipelines
```

## Repository Structure

```
.
├── tasks/
│   └── hello-task.yaml          # Simple task definition
├── pipelines/
│   └── hello-pipeline.yaml      # Pipeline using the task
├── runs/
│   └── hello-pipeline-run.yaml  # PipelineRun to execute the pipeline
└── README.md
```

## Usage

### 1. Apply the Task

```bash
kubectl apply -f tasks/hello-task.yaml
```

Verify the task was created:
```bash
kubectl get tasks
```

### 2. Apply the Pipeline

```bash
kubectl apply -f pipelines/hello-pipeline.yaml
```

Verify the pipeline was created:
```bash
kubectl get pipelines
```

### 3. Run the Pipeline

```bash
kubectl apply -f runs/hello-pipeline-run.yaml
```

### 4. Check the PipelineRun Status

```bash
kubectl get pipelineruns
```

View detailed logs:
```bash
kubectl logs -l tekton.dev/pipelineRun=hello-pipeline-run --all-containers
```

Or use the Tekton CLI (tkn):
```bash
tkn pipelinerun logs hello-pipeline-run -f
```

## Customization

You can customize the greeting message by modifying the `message` parameter in [`runs/hello-pipeline-run.yaml`](runs/hello-pipeline-run.yaml:11):

```yaml
params:
  - name: message
    value: "Your custom message here!"
```

## Cleanup

Remove all resources:
```bash
kubectl delete -f runs/hello-pipeline-run.yaml
kubectl delete -f pipelines/hello-pipeline.yaml
kubectl delete -f tasks/hello-task.yaml
```

## Resources

- [Tekton Documentation](https://tekton.dev/docs/)
- [Tekton Pipelines GitHub](https://github.com/tektoncd/pipeline)
- [Tekton Tutorial](https://tekton.dev/docs/getting-started/)