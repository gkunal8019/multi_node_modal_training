# Multi_node_modal_training

Reference to the official PyTorch ImageNet example documentation.

# Distributed Training with ResNet50 on Kubernetes

This documentation provides instructions for setting up and running distributed training using ResNet50 with the NCCL backend on Kubernetes.

## Prerequisites

- Kubernetes cluster with GPU support.
- Docker image `dockerimage/multinodes:0.2`.
- Access to a shared storage path on the host (`/raid`).
- Password-less SSH access between the servers.

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
python main.py -a resnet50 --dist-url 'tcp://10.10.116.244:4000' --dist-backend 'nccl' --multiprocessing-distributed --world-size 2 --batch-size 128 --rank 0 image2012
```

### Node 1

```bash
python main.py -a resnet50 --dist-url 'tcp://10.10.343.244:4000' --dist-backend 'nccl' --multiprocessing-distributed --world-size 2 --batch-size 128 --rank 1 image2012
```

## Notes

- Ensure that the IP address (`10.10.424.244`) is accessible from both nodes.
- The `image2012` argument is the path to the dataset you are using for training.
- Ensure that password-less SSH access is set up between the servers to allow seamless communication.

## Additional Resources

For more details on distributed training with PyTorch, refer to the official [PyTorch ImageNet Example Documentation](https://github.com/pytorch/examples/blob/main/imagenet/README.md).

## Troubleshooting

- If you encounter any issues with pod scheduling or GPU allocation, verify your Kubernetes node labels and GPU resource availability.
- For any networking issues, ensure that the specified `dist-url` is reachable from both nodes.
- Check that password-less SSH is properly configured to prevent connectivity issues between the nodes.

---

## Saving Logs

To save logs with the current date and time:

#### Node 0

```bash
LOG_FILE="training_$(date +'%Y%m%d_%H%M%S').log"
python main.py -a resnet18 --dist-url 'tcp://10.10.246.202:9090' --dist-backend 'nccl' --multiprocessing-distributed --world-size 2 --rank 0 --epochs 2 image2012 > "$LOG_FILE" 2>&1
```

#### Node 1

```bash
LOG_FILE="training_$(date +'%Y%m%d_%H%M%S').log"
python main.py -a resnet18 --dist-url 'tcp://10.10.246.202:9090' --dist-backend 'nccl' --multiprocessing-distributed --world-size 2 --rank 1 --epochs 2 image2012 > "$LOG_FILE" 2>&1
```

Alternatively, you can use `tee` to append logs to a file:

#### Node 0

```bash
python main.py -a resnet18 --dist-url 'tcp://10.10.246.202:9090' --dist-backend 'nccl' --multiprocessing-distributed --world-size 2 --rank 0 --epochs 2 image2012 2>&1 | tee -a training.log
```

#### Node 1

```bash
python main.py -a resnet18 --dist-url 'tcp://10.10.246.202:9090' --dist-backend 'nccl' --multiprocessing-distributed --world-size 2 --rank 1 --epochs 2 image2012 2>&1 | tee -a training.log
```

This addition includes the commands for saving logs with the current date and time, as well as the alternative method using `tee` to append logs to a file.





