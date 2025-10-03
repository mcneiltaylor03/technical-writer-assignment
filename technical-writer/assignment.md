# Debugging with `kubectl`
## Overview
`kubectl` is the command-line tool for interacting with Kubernetes clusters. While it is used for many different cluster operations, it is the primary tool for diagnosing and debugging workloads. This reference covers the essential kubectl commands for debugging common pod and container issues.

## Essential Commands

### kubectl get pods

**Description**

Displays all pods in a namespace along with their current status.

**Syntax**

```shell
kubectl get pods [--namespace <name>] [options]
```

**Examples**

- Use `kubectl get pods -o wide` to display pods with node assignment and IP address details.

- Use `kubectl get pods --all-namespaces` to display all pods across every namespace in the cluster. 

- Use `kubectl get pod <pod-name> -o yaml` to display a detailed pod manifest in YAML format.    

**Example Output**

```shell
kubectl get pods
```

```shell
NAME                     READY   STATUS    RESTARTS   AGE
nginx-5d4697d446-f949t   1/1     Running   0          5m
nginx-5d4697d676-wfr12   1/1     Running   0          9m
web-app-7b8c9d-xyz34     1/1     Running   2          1h
mysql-0                  0/1     Pending   0          30s
```

**Use Cases**
- Check the status of pods after deploying an application.
- Identify pods that are failing or stuck in a pending state.
- Export the complete pod configuration for detailed analysis or backup purposes.
- Identify pods that have recently crashed or restarted.

### kubectl describe

**Description**

Displays detailed information about a pod's current state, configuration, and recent events. 

**Syntax**
```shell
kubectl describe pod <pod-name> [--namespace <name>]
```

**Examples**

- Use `kubectl describe pod <pod-name>` to gather information about a pod in the current namespace.
- Use `kubectl describe pod <pod-name> | grep Events -A 10` to focus on recent events that may indicate issues.

**Example Output**

```shell
kubectl describe pod nginx-5d4697d446-f949t
```

```shell
Name:         nginx-5d4697d446-f949t
Namespace:    default
Priority:     0
Node:         minikube/192.168.49.2
Start Time:   Thu, 02 Oct 2025 11:23:00 +0000
Labels:       app=nginx
              pod-template-hash=5d4697d446
Annotations:  <none>
Status:       Running
IP:           172.17.0.4
IPs:
  IP:  172.17.0.4
Controlled By:  ReplicaSet/nginx-5d4697d446
Containers:
  nginx:
    Container ID:   docker://abcdef12345...
    Image:          nginx:1.14.2
    Image ID:       docker-pullable://nginx@sha256:4a07a1b412b1...
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 02 Oct 2025 11:23:02 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-abcde (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-abcde:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m1s   default-scheduler  Successfully assigned default/nginx-5d4697d446-f949t to minikube
  Normal  Pulling    2m59s  kubelet            Pulling image "nginx:1.14.2"
  Normal  Pulled     2m58s  kubelet            Successfully pulled image "nginx:1.14.2" in 1.12140409s
  Normal  Created    2m58s  kubelet            Created container nginx
  Normal  Started    2m58s  kubelet            Started container nginx
```

**Use Cases**

- Diagnose why a pod is stuck in a pending state by reviewing scheduling events.
- Identify resource constraint issues that prevent pod startup.
- Review container restart counts and reasons for crashloop behaviors.

>**Note:**
>
>This command is useful for understanding *why* a pod is in a certain state, as it includes events from the Kubernetes control plane.

### kubectl logs

**Description**

Retrieves the logs of a container within a pod.

**Syntax**
```shell
kubectl logs <pod-name> [-c <container-name>] [options]
```

**Examples**
- Use `kubectl logs <pod-name>` to retrieve logs from the main container in the pod.
- Use `kubectl logs -f <pod-name>` to stream logs in real-time.

**Example Output**
```shell
kubectl logs nginx-5d4697d446-f949t
```
```shell
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2025/10/02 11:23:02 [notice] 1#1: using the "epoll" event method
2025/10/02 11:23:02 [notice] 1#1: nginx/1.14.2
2025/10/02 11:23:02 [notice] 1#1: built by gcc 8.3.0 (Debian 8.3.0-6)
2025/10/02 11:23:02 [notice] 1#1: OS: Linux 5.15.0-56-generic
2025/10/02 11:23:02 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2025/10/02 11:23:02 [notice] 1#1: start worker processes
2025/10/02 11:23:02 [notice] 1#1: start worker process 30
2025/10/02 11:23:02 [notice] 1#1: start worker process 31
172.17.0.1 - - [02/Oct/2025:11:25:15 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
172.17.0.1 - - [02/Oct/2025:11:26:42 +0000] "GET /health HTTP/1.1" 200 612 "-" "kube-probe/1.25" "-"
172.17.0.1 - - [02/Oct/2025:11:27:42 +0000] "GET /health HTTP/1.1" 200 612 "-" "kube-probe/1.25" "-"
```

**Use Cases**
- Review application output and error messages to diagnose runtime issues.
- Troubleshoot containers that are crashing or restarting frequently.
- Stream logs in real-time while reproducing an issue for live debugging.

>**Note:**
>
>For pods with multiple containers, you must specify the container name using the `-c` flag. Use `kubectl describe pod <pod-name>` to list all containers in a pod.


### kubectl exec

**Description**

Executes a command inside a running container in a pod. Use the `-it` flags for interactive sessions or omit them for one-off commands.

**Syntax**
```shell
kubectl exec -it <pod-name> -- <command>
```

**Examples**
- Use `kubectl exec -it <pod-name> -- /bin/sh` to open an interactive shell in the default container.
- Use `kubectl exec <pod-name> -- env` to print environment variables.
- Use `kubectl exec <pod-name> -- cat /etc/config/config.yaml` to view contents of a configuration file.

**Example Output**
```shell
kubectl exec nginx-5d4697d446-f949t -- env
```

```shell
PATH=/usr/local/sbin:/usr/local/bin:/usr/
HOSTNAME=default-pod
TERM=xterm
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
APP_VERSION=1.2.3
DATABASE_URL=postgresql://db-service:5432/myapp
REDIS_HOST=redis-service
REDIS_PORT=6379
LOG_LEVEL=info
HOME=/root
```

**Use Cases**
- Verify environment variables and test network connectivity to other services.
- Execute diagnostic commands like `curl`, `ping`, or `netstat` inside the container.
- Troubleshoot application behavior by inspecting running processes or log files.

>**Note:**
>
>For pods with multiple containers, specify the container name using the `-c` flag.

### kubectl debug

**Description**
Creates a copy of a pod or attaches an ephemeral container to a pod for troubleshooting.

**Syntax**
```shell
kubectl debug <pod-name> [options]
```

**Examples**
- Use `kubectl debug node/<node-name> -it --image=ubuntu` to create a debug pod on a specific node.
- Use `kubectl debug <pod-name> --copy-to=my-pod-debug` to clone a pod for debugging.
- Use `kubectl debug <pod-name> -it --image=busybox` to attach an ephemeral debug container with BusyBox tools.

**Example Output**
```shell
kubectl debug nginx-5d4697d446-f949t --copy-to=nginx-debug
```

```shell
Creating debugging pod nginx-debug with container nginx.
If you don't see a command prompt, try pressing enter.
root@nginx-debug:/# 
root@nginx-debug:/# ls -la
total 80
drwxr-xr-x   1 root root 4096 Oct  2 11:45 .
drwxr-xr-x   1 root root 4096 Oct  2 11:45 ..
-rwxr-xr-x   1 root root    0 Oct  2 11:45 .dockerenv
drwxr-xr-x   2 root root 4096 Sep 15  2021 bin
drwxr-xr-x   2 root root 4096 Aug  9  2021 boot
drwxr-xr-x   5 root root  360 Oct  2 11:45 dev
drwxr-xr-x   1 root root 4096 Oct  2 11:45 etc
drwxr-xr-x   2 root root 4096 Aug  9  2021 home
drwxr-xr-x   1 root root 4096 Sep 15  2021 lib
drwxr-xr-x   2 root root 4096 Sep 15  2021 lib64
drwxr-xr-x   2 root root 4096 Sep 15  2021 media
drwxr-xr-x   2 root root 4096 Sep 15  2021 mnt
drwxr-xr-x   2 root root 4096 Sep 15  2021 opt
dr-xr-xr-x 267 root root    0 Oct  2 11:45 proc
drwx------   2 root root 4096 Sep 15  2021 root
drwxr-xr-x   3 root root 4096 Sep 15  2021 run
drwxr-xr-x   2 root root 4096 Sep 15  2021 sbin
drwxr-xr-x   2 root root 4096 Sep 15  2021 srv
dr-xr-xr-x  13 root root    0 Oct  2 11:45 sys
drwxrwxrwt   1 root root 4096 Oct  2 11:23 tmp
drwxr-xr-x   1 root root 4096 Sep 15  2021 usr
drwxr-xr-x   1 root root 4096 Sep 15  2021 var
root@nginx-debug:/# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  32648  5240 pts/0    Ss+  11:45   0:00 nginx: master process nginx -g daemon off;
nginx       30  0.0  0.0  33112  2408 pts/0    S+   11:45   0:00 nginx: worker process
nginx       31  0.0  0.0  33112  2408 pts/0    S+   11:45   0:00 nginx: worker process
root        42  0.2  0.0   3868  3180 pts/1    Ss   11:45   0:00 /bin/bash
root        56  0.0  0.0   7640  2708 pts/1    R+   11:45   0:00 ps aux
root@nginx-debug:/# exit
exit
```

**Use Cases**
- Troubleshoot pods without modifying the original container or disrupting the workload.
- Clone a failing pod to test configuration changes without affecting the running instance.

>**Note:**
>
>The `kubectl debug` command is useful for debugging minimal container images that don't include standard debugging tools. Ephemeral containers are temporary and will not persist after the pod is deleted.

## Reference Table

|Command|Purpose|Example|
|---|---|---|
|`kubectl get pods`| List pods and their status| `kubectl get pods --all-namespaces`|
|`kubectl describe pod <pod-name>`|Display detailed information about a pod| `kubectl describe pod my-pod`|
|`kubectl logs <pod-name>`|View logs from a container|`kubectl logs my-pod`| 
|`kubectl exec -it <pod-name> -- /bin/sh`|Execute commands inside a container|`kubectl exec -it my-pod -- /bin/sh`| 
|`kubectl debug <pod-name>`|Create an ephemeral debug container|`kubectl debug my-pod -it --image=busybox`|

## External References

- [Kubernetes Overview](https://kubernetes.io/docs/concepts/overview/)
- [`kubectl` Reference](https://kubernetes.io/docs/reference/kubectl/)
- [`kubectl` Quick Reference](https://kubernetes.io/docs/reference/kubectl/quick-reference/)
- [Ephemeral Containers](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers/)


