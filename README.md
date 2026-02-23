# Tekton Pipeline Example

A simple example repository demonstrating basic Tekton Pipelines concepts.

## Overview

This repository contains a minimal Tekton pipeline setup that demonstrates:
- Two sequential tasks (say-hello and say-goodbye)
- A Pipeline that orchestrates the tasks
- A PipelineRun to execute the Pipeline
- Tekton Triggers with EventListener for webhook-based execution

## Prerequisites

- Kubernetes cluster (v1.24+)
- Tekton Pipelines installed on your cluster
- Tekton Triggers installed on your cluster (for EventListener support)

### Installing Tekton Pipelines

```bash
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

Verify the installation:
```bash
kubectl get pods --namespace tekton-pipelines
```

### Installing Tekton Triggers

```bash
kubectl apply --filename https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml
kubectl apply --filename https://storage.googleapis.com/tekton-releases/triggers/latest/interceptors.yaml
```

Verify the installation:
```bash
kubectl get pods --namespace tekton-pipelines
```

## Repository Structure

```
.
├── .tekton/
│   ├── hello-pipeline.yaml      # Pipeline with embedded task and PipelineRun
│   └── triggers.yaml            # EventListener, TriggerBinding, TriggerTemplate, and RBAC
└── README.md
```

## Usage

### Option 1: Manual Pipeline Execution

#### 1. Apply the Pipeline

The [`.tekton/hello-pipeline.yaml`](.tekton/hello-pipeline.yaml:1) file contains all resources in a single file using YAML document separators (`---`):
- Pipeline definition with an embedded TaskSpec
- PipelineRun to execute the pipeline

```bash
kubectl apply -f .tekton/hello-pipeline.yaml
```

This will create both the Pipeline and PipelineRun resources.

#### 2. Check the PipelineRun Status

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

### Option 2: Webhook-Based Execution with EventListener

#### 1. Apply the Triggers

```bash
kubectl apply -f .tekton/triggers.yaml
```

This creates:
- **TriggerBinding**: Extracts data from webhook payload
- **TriggerTemplate**: Creates PipelineRun from extracted data
- **EventListener**: Exposes webhook endpoint
- **ServiceAccount & RBAC**: Permissions for trigger execution

#### 2. Expose the EventListener

Get the EventListener service:
```bash
kubectl get svc el-hello-event-listener
```

For local testing, port-forward the service:
```bash
kubectl port-forward svc/el-hello-event-listener 8080:8080
```

#### 3. Trigger the Pipeline via Webhook

Send a POST request to trigger the pipeline:
```bash
curl -X POST http://localhost:8080 \
  -H 'Content-Type: application/json' \
  -d '{"message": "Hello from webhook trigger!"}'
```

#### 4. Watch the Pipeline Execute

```bash
kubectl get pipelineruns -w
```

View logs of the latest run:
```bash
tkn pipelinerun logs -f --last
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
kubectl delete -f .tekton/triggers.yaml
```

Or delete specific resources:
```bash
# Delete PipelineRuns
kubectl delete pipelineruns --all

# Delete Pipeline
kubectl delete pipeline hello-pipeline

# Delete Triggers
kubectl delete eventlistener hello-event-listener
kubectl delete triggertemplate hello-trigger-template
kubectl delete triggerbinding hello-trigger-binding
```

## Resources

- [Tekton Documentation](https://tekton.dev/docs/)
- [Tekton Pipelines GitHub](https://github.com/tektoncd/pipeline)
- [Tekton Triggers Documentation](https://tekton.dev/docs/triggers/)
- [Tekton Tutorial](https://tekton.dev/docs/getting-started/)