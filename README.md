# Multi_node_modal_training

Reference to the official PyTorch ImageNet example documentation.


# Distributed Training with ResNet50 on Kubernetes

This documentation provides instructions for setting up and running distributed training using ResNet50 with the NCCL backend on Kubernetes.

## Prerequisites

- Kubernetes cluster with GPU support.
- Docker image `paawanpurdhani/multinodes:0.2`.
- Access to a shared storage path on the host (`/raid`).

## Setup

### 1. Create the Kubernetes Pod

Create a YAML file (`multinode.yaml`) with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multinode
spec:
  containers:
  - name: mycontainer
    image: paawanpurdhani/multinodes:0.2
    volumeMounts:
    - mountPath: /workspace
      name: myhostpath
    - mountPath: /dev/shm
      name: dshm
    workingDir: /workspace
    resources:
      limits:
        nvidia.com/gpu: 1
    command: ["sleep"]
    args: ["infinity"]
  volumes:
  - name: myhostpath
    hostPath:
      path: /raid
      type: Directory
  - name: dshm
    emptyDir:
      medium: Memory
      sizeLimit: 2Gi
  nodeSelector:
    kubernetes.io/hostname: gpuworker1
```

Create the pod using the following command:

```bash
kubectl create -f multinode.yaml
```

### 2. Check Pod Status

Check the status of the pod to ensure it is running:

```bash
kubectl get po
```

To check which pod is scheduled on which node and see the IP of the pod:

```bash
kubectl get po -o wide
```

### 3. Access the Pod

Access the pod using the following command:

```bash
kubectl exec -it multinode bash
```

## Running the Training

Run the following commands on each node to start the distributed training:

### Node 0

```bash
python main.py -a resnet50 --dist-url 'tcp://10.10.246.244:4000' --dist-backend 'nccl' --multiprocessing-distributed --world-size 2 --batch-size 128 --rank 0 image2012
```

### Node 1

```bash
python main.py -a resnet50 --dist-url 'tcp://10.10.246.244:4000' --dist-backend 'nccl' --multiprocessing-distributed --world-size 2 --batch-size 128 --rank 1 image2012
```

## Notes

- Ensure that the IP address (`10.10.246.244`) is accessible from both nodes.
- The `image2012` argument is the path to the dataset you are using for training.

## Additional Resources

For more details on distributed training with PyTorch, refer to the official [PyTorch ImageNet Example Documentation](https://github.com/pytorch/examples/blob/main/imagenet/README.md).

## Troubleshooting

- If you encounter any issues with pod scheduling or GPU allocation, verify your Kubernetes node labels and GPU resource availability.
- For any networking issues, ensure that the specified `dist-url` is reachable from both nodes.

## Contributing

Contributions are welcome! Please submit a pull request or open an issue on the GitHub repository.
