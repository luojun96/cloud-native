# Checking Cluster Problems

## Node Issues

[node-problem-detector](https://github.com/kubernetes/node-problem-detector)

## Pod Issues

- Pods are stuck in 'Pending' state
- Pods not starting or 'CrashLoopBackOff' or 'ImagePullBackOff' or 'ErrImagePull' or 'CreateContainerConfigError'
- Pods not ready
- Service can't reach the pods
- Liveness probe failures
- Unexpected termination of pods
- Pods are stuck in 'Terminating' state

### Steps to check pod issues

- login the host of the problem pod using ssh
  - create a pod which supports ssh
  - use a load balance proxy to forwad ssh request
- check logs
  - systemd log

  ```bash
  journalctl -afu kubelet -S "2023-04-12 15:39:00"
  ```

  - container log

  ```bash
  kubectl logs -f -c <containername> <podname>
  kubectl logs -f --all-containers <podname> 
  kubectl logs -f -c <containername> <podname> --previous
  kubectl exec <podname> -- tail -f /path/to/log
  ```

