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
├── .tekton/
│   └── hello-pipeline.yaml      # Pipeline with embedded task and PipelineRun
└── README.md
```

## Usage

### Apply the Pipeline and Run

The `.tekton/hello-pipeline.yaml` file contains all resources in a single file using YAML document separators (`---`):
- Pipeline definition with an embedded TaskSpec
- PipelineRun to execute the pipeline

```bash
kubectl apply -f .tekton/hello-pipeline.yaml
```

This will create both the Pipeline and PipelineRun resources.

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

You can customize the greeting message by modifying the `message` parameter in [`.tekton/hello-pipeline.yaml`](.tekton/hello-pipeline.yaml:40):

```yaml
params:
  - name: message
    value: "Your custom message here!"
```

Or create a new PipelineRun with different parameters:

```bash
kubectl create -f - <<EOF
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: hello-pipeline-run-custom
spec:
  pipelineRef:
    name: hello-pipeline
  params:
    - name: message
      value: "Your custom message!"
EOF
```

## Cleanup

Remove all resources:
```bash
kubectl delete -f .tekton/hello-pipeline.yaml
```

Or delete specific resources:
```bash
kubectl delete pipelinerun hello-pipeline-run
kubectl delete pipeline hello-pipeline
```

## Resources

- [Tekton Documentation](https://tekton.dev/docs/)
- [Tekton Pipelines GitHub](https://github.com/tektoncd/pipeline)
- [Tekton Tutorial](https://tekton.dev/docs/getting-started/)