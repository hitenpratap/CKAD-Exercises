# CKAD Practice Questions: Application Maintenance and Observability

## Overview
Maintaining and observing applications running in Kubernetes is crucial for ensuring reliability, performance, and scalability. This includes understanding the lifecycle of APIs, implementing robust health checks, leveraging Kubernetes-native tools, and debugging efficiently.

## Topics Covered
1. **Understand API Deprecations**
   - Learn about Kubernetes API versioning and how to track deprecations.
   - Understand the process for migrating to newer API versions.

2. **Implement Probes and Health Checks**
   - Configure liveness, readiness, and startup probes to ensure application stability.
   - Best practices for probe configurations.

3. **Use Built-in CLI Tools to Monitor Kubernetes Applications**
   - Explore tools like `kubectl top`, `kubectl describe`, and `kubectl get` for application monitoring.
   - Analyze resource usage and identify bottlenecks.

4. **Utilize Container Logs**
   - Learn to access and analyze container logs using `kubectl logs`.
   - Understand log levels and how to debug issues effectively.

5. **Debugging in Kubernetes**
   - Techniques for debugging workloads using `kubectl exec`, ephemeral containers, and debugging nodes.
   - Common debugging workflows and tools.

## Questions

### 1. Identify Deprecated APIs in a Kubernetes Cluster

**Scenario**:
- Use `kubectl` commands to identify deprecated APIs in a running cluster.
- Verify if any workloads are using deprecated API versions.

<details>
<summary>Details</summary>

#### Steps to Identify Deprecated APIs

1. Check deprecated APIs:
   ```bash
   kubectl get apiresources
   ```
2. List resources using specific API versions:
   ```bash
   kubectl get <resource> --api-version=<api-version>
   ```
3. Use tools like `kubectl deprecations` or third-party utilities to analyze cluster-wide API usage.

</details>

---

### 2. Migrate a Deployment from a Deprecated API Version

**Scenario**:
- Migrate a Deployment using `apps/v1beta1` to `apps/v1`.
- Ensure compatibility with the newer API version.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration (Updated Version)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: updated-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: updated-app
  template:
    metadata:
      labels:
        app: updated-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

#### Steps to Migrate

1. Export the existing deployment:
   ```bash
   kubectl get deployment old-deployment -o yaml > old-deployment.yaml
   ```
2. Update the `apiVersion` and resource specifications in the YAML file.
3. Apply the updated YAML:
   ```bash
   kubectl apply -f updated-deployment.yaml
   ```
4. Verify the new deployment:
   ```bash
   kubectl get deployment updated-deployment
   ```

</details>

---

### 3. Use `kubectl convert` to Update API Versions

**Scenario**:
- Convert a resource manifest from a deprecated API version to a supported version using `kubectl convert`.

<details>
<summary>Details</summary>

#### Steps to Use `kubectl convert`

1. Install the `kubectl-convert` plugin if not already installed:
   ```bash
   kubectl krew install convert
   ```
2. Convert a resource manifest:
   ```bash
   kubectl convert -f deprecated-resource.yaml --output-version apps/v1 > updated-resource.yaml
   ```
3. Apply the updated manifest:
   ```bash
   kubectl apply -f updated-resource.yaml
   ```
4. Verify the resource:
   ```bash
   kubectl get <resource-type> <resource-name>
   ```

</details>

---

### 4. Analyze API Deprecations Using Audit Logs

**Scenario**:
- Enable audit logging in the cluster to track usage of deprecated APIs.
- Extract and analyze deprecated API calls from the logs.

<details>
<summary>Details</summary>

#### Steps to Enable and Analyze Audit Logs

1. Configure the API server to enable audit logging:
   ```bash
   --audit-log-path=/var/log/kubernetes/audit.log
   --audit-policy-file=/etc/kubernetes/audit-policy.yaml
   ```
2. Use an audit policy file with the following content:
   ```yaml
   apiVersion: audit.k8s.io/v1
   kind: Policy
   rules:
   - level: Metadata
     resources:
     - group: ""
       resources: ["deprecatedapis"]
   ```
3. Extract deprecated API usage from the logs:
   ```bash
   grep "deprecated" /var/log/kubernetes/audit.log
   ```
4. Migrate resources based on the findings.

</details>

---

### 5. Monitor API Deprecations Across Kubernetes Releases

**Scenario**:
- Use Kubernetes release notes and documentation to monitor API deprecations and removals.
<details>
<summary>Details</summary>

#### Steps to Monitor and Prepare for Deprecations

1. Review the Kubernetes release notes for API deprecations:
   ```
   https://kubernetes.io/docs/setup/release/notes/
   ```
2. Use a third-party tool like `pluto` to detect deprecated APIs:
   ```bash
   pluto detect-files -d manifests/
   ```
3. Proactively update manifests to supported API versions before upgrading the cluster.

</details>

---

### 6. Configure a Liveness Probe to Restart a Pod

**Scenario**:
- Deploy a Pod named `liveness-app` running the `nginx:latest` image.
- Configure a liveness probe to check the `/healthz` endpoint on port 80.
- The probe should check every 5 seconds and restart the container if the endpoint fails.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-app
spec:
  containers:
  - name: nginx
    image: nginx:latest
    livenessProbe:
      httpGet:
        path: /healthz
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 5
```

#### Steps to Apply

Save the YAML file and apply it:

```bash
kubectl apply -f liveness-app.yaml
```

Verify the probe functionality:

1. Simulate a failure by removing the `/healthz` endpoint from the container.
2. Observe the container restart:
   ```bash
   kubectl describe pod liveness-app
   ```

</details>

---

### 7. Configure a Readiness Probe for Traffic Control

**Scenario**:
- Deploy a Pod named `readiness-app` running the `nginx:latest` image.
- Configure a readiness probe to check the `/ready` endpoint on port 80.
- The probe should check every 10 seconds to determine if the Pod is ready to serve traffic.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-app
spec:
  containers:
  - name: nginx
    image: nginx:latest
    readinessProbe:
      httpGet:
        path: /ready
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
```

#### Steps to Apply

Save the YAML file and apply it:

```bash
kubectl apply -f readiness-app.yaml
```

Verify the probe functionality:

1. Observe the Pod's `READY` status:
   ```bash
   kubectl get pods
   ```
2. Simulate readiness issues by removing the `/ready` endpoint and observe traffic behavior.

</details>

---

### 8. Configure a Startup Probe for Slow-Starting Applications

**Scenario**:
- Deploy a Pod named `startup-app` running a custom application.
- Configure a startup probe to check the `/startup` endpoint on port 8080.
- Allow up to 60 seconds for the application to initialize before failing the Pod.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: startup-app
spec:
  containers:
  - name: custom-app
    image: custom-app:latest
    startupProbe:
      httpGet:
        path: /startup
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5
      failureThreshold: 12
```

#### Steps to Apply

Save the YAML file and apply it:

```bash
kubectl apply -f startup-app.yaml
```

Verify the probe functionality:

1. Observe the Pod's `STATUS` during initialization:
   ```bash
   kubectl get pods
   ```
2. Simulate startup delays and monitor the impact on the Pod.

</details>

---

### 9. Configure Combined Probes for a StatefulSet

**Scenario**:
- Deploy a StatefulSet named `stateful-app` running `mysql:5.7`.
- Configure:
  - A liveness probe to check the `/health` endpoint on port 3306.
  - A readiness probe to check the `/ready` endpoint on port 3306.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: stateful-app
spec:
  serviceName: "mysql-service"
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        livenessProbe:
          tcpSocket:
            port: 3306
          initialDelaySeconds: 10
          periodSeconds: 5
        readinessProbe:
          tcpSocket:
            port: 3306
          initialDelaySeconds: 10
          periodSeconds: 5
```

#### Steps to Apply

Save the YAML file and apply it:

```bash
kubectl apply -f stateful-app.yaml
```

Verify the probe functionality:

1. Observe the readiness and liveness behavior:
   ```bash
   kubectl describe pod -l app=mysql
   ```
2. Simulate failures for each probe and monitor the StatefulSet behavior.

</details>

---

### 10. Implement Best Practices for Probe Configurations

**Scenario**:
- Configure a Deployment named `best-practices-app` with best practices for probes:
  - Avoid tight probe intervals that can overload the application.
  - Use appropriate initial delay settings based on application behavior.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: best-practices-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: best-practices
  template:
    metadata:
      labels:
        app: best-practices
    spec:
      containers:
      - name: app-container
        image: app:latest
        livenessProbe:
          httpGet:
            path: /healthz
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
```

#### Steps to Apply

Save the YAML file and apply it:

```bash
kubectl apply -f best-practices-app.yaml
```

Verify the configuration:

1. Observe the Pod's behavior under normal and failure conditions:
   ```bash
   kubectl describe pod -l app=best-practices
   ```
2. Adjust probe configurations based on application performance.

</details>

---

### 11. Monitor Resource Usage with `kubectl top`

**Scenario**:
- Use the `kubectl top` command to monitor resource usage for nodes and pods.
- Identify which pods are consuming the most CPU and memory in the `default` namespace.

<details>
<summary>Details</summary>

#### Steps to Monitor Resource Usage

1. Check node resource usage:
   ```bash
   kubectl top nodes
   ```
2. Check pod resource usage in the default namespace:
   ```bash
   kubectl top pods -n default
   ```
3. Sort and analyze the output to identify resource bottlenecks.

</details>

---

### 12. Describe a Pod to Debug Issues

**Scenario**:
- Use `kubectl describe` to inspect a Pod named `debug-pod`.
- Identify potential issues such as events, status, and probe failures.

<details>
<summary>Details</summary>

#### Steps to Debug Using `kubectl describe`

1. Describe the Pod:
   ```bash
   kubectl describe pod debug-pod
   ```
2. Analyze the output for the following:
   - Events: Check for warnings or errors.
   - Container Status: Look for `Waiting` or `Terminated` states.
   - Probe Failures: Inspect liveness or readiness probe issues.

</details>

---

### 13. Analyze Application Logs with `kubectl logs`

**Scenario**:
- Use `kubectl logs` to view the logs of a Pod named `app-logs`.
- Filter logs for a specific container in a multi-container Pod.

<details>
<summary>Details</summary>

#### Steps to View Logs

1. View logs for a single-container Pod:
   ```bash
   kubectl logs app-logs
   ```
2. View logs for a specific container in a multi-container Pod:
   ```bash
   kubectl logs app-logs -c container-name
   ```
3. Stream logs in real time:
   ```bash
   kubectl logs -f app-logs
   ```

</details>

---

### 14. List and Filter Resources Using `kubectl get`

**Scenario**:
- Use `kubectl get` to list resources such as pods, services, and deployments in the `production` namespace.
- Filter resources with specific labels.

<details>
<summary>Details</summary>

#### Steps to List and Filter Resources

1. List all Pods in the `production` namespace:
   ```bash
   kubectl get pods -n production
   ```
2. Filter Pods with a specific label:
   ```bash
   kubectl get pods -n production -l app=my-app
   ```
3. Display additional details such as wide output:
   ```bash
   kubectl get pods -n production -o wide
   ```

</details>

---

### 15. Monitor Events in a Namespace

**Scenario**:
- Use `kubectl get events` to monitor events in the `staging` namespace.
- Analyze the events to identify issues such as failed scheduling or probe failures.

<details>
<summary>Details</summary>

#### Steps to Monitor Events

1. List all events in the `staging` namespace:
   ```bash
   kubectl get events -n staging
   ```
2. Sort events by timestamp:
   ```bash
   kubectl get events -n staging --sort-by='.metadata.creationTimestamp'
   ```
3. Focus on specific event types, such as warnings:
   ```bash
   kubectl get events -n staging | grep Warning
   ```

</details>

---

### 16. Analyze Resource Usage with `kubectl describe` and `kubectl top`

**Scenario**:
- Use `kubectl describe` and `kubectl top` together to correlate resource usage with Pod configurations.
- Investigate a Pod named `resource-intensive-app` for potential bottlenecks.

<details>
<summary>Details</summary>

#### Steps to Analyze Resource Usage

1. Describe the Pod to understand its configuration:
   ```bash
   kubectl describe pod resource-intensive-app
   ```
2. Check the Pod's resource usage:
   ```bash
   kubectl top pod resource-intensive-app
   ```
3. Compare the resource requests and limits with actual usage.

</details>

---

### 17. View Logs for a Single-Container Pod

**Scenario**:
- Access logs for a Pod named `single-container-app` running a single container.
- Analyze the logs for potential errors.

<details>
<summary>Details</summary>

#### Steps to Access Logs

1. View logs for the Pod:
   ```bash
   kubectl logs single-container-app
   ```
2. Search for specific keywords in the logs:
   ```bash
   kubectl logs single-container-app | grep ERROR
   ```
3. Save the logs to a file for further analysis:
   ```bash
   kubectl logs single-container-app > app-logs.txt
   ```

</details>

---

### 18. Access Logs for a Multi-Container Pod

**Scenario**:
- Access logs for a Pod named `multi-container-app` that has multiple containers.
- Focus on a specific container named `backend` to debug an issue.

<details>
<summary>Details</summary>

#### Steps to Access Logs

1. List containers in the Pod:
   ```bash
   kubectl get pods multi-container-app -o jsonpath='{.spec.containers[*].name}'
   ```
2. Access logs for the `backend` container:
   ```bash
   kubectl logs multi-container-app -c backend
   ```
3. Stream logs in real time for the `backend` container:
   ```bash
   kubectl logs -f multi-container-app -c backend
   ```

</details>

---

### 19. Stream Logs for a Deployment

**Scenario**:
- Monitor real-time logs for all Pods in a Deployment named `real-time-logs`.
- Identify any errors occurring during traffic handling.

<details>
<summary>Details</summary>

#### Steps to Stream Logs

1. List Pods in the Deployment:
   ```bash
   kubectl get pods -l app=real-time-logs
   ```
2. Stream logs for all Pods:
   ```bash
   kubectl logs -f -l app=real-time-logs
   ```
3. Filter logs for specific errors or warnings:
   ```bash
   kubectl logs -f -l app=real-time-logs | grep WARN
   ```

</details>

---

### 20. Understand Log Levels and Filter Logs

**Scenario**:
- Access logs for a Pod named `log-level-app`.
- Filter logs by severity levels such as `INFO`, `WARN`, or `ERROR`.

<details>
<summary>Details</summary>

#### Steps to Filter Logs by Levels

1. View all logs for the Pod:
   ```bash
   kubectl logs log-level-app
   ```
2. Filter logs for `INFO` messages:
   ```bash
   kubectl logs log-level-app | grep INFO
   ```
3. Identify critical issues by filtering `ERROR` messages:
   ```bash
   kubectl logs log-level-app | grep ERROR
   ```

</details>

---

### 21. Debug a CrashLoopBackOff Pod

**Scenario**:
- Debug a Pod named `crash-loop-app` that is in a `CrashLoopBackOff` state.
- Access logs for the last failed attempt to identify the issue.

<details>
<summary>Details</summary>

#### Steps to Debug

1. Check the status of the Pod:
   ```bash
   kubectl get pod crash-loop-app
   ```
2. View logs for the previous container instance:
   ```bash
   kubectl logs crash-loop-app --previous
   ```
3. Analyze the logs to determine the root cause of the crash.

</details>

---

### 22. Aggregate Logs Across Pods

**Scenario**:
- Aggregate logs for all Pods in the `staging` namespace with the label `app=my-app`.
- Save the aggregated logs to a file for analysis.

<details>
<summary>Details</summary>

#### Steps to Aggregate Logs

1. View logs for all Pods in the namespace:
   ```bash
   kubectl logs -l app=my-app -n staging
   ```
2. Save the logs to a file:
   ```bash
   kubectl logs -l app=my-app -n staging > aggregated-logs.txt
   ```
3. Use `grep` or other tools to analyze the aggregated logs:
   ```bash
   grep ERROR aggregated-logs.txt
   ```

</details>

---

### 23. Debug a Pod Using `kubectl exec`

**Scenario**:
- Access a running Pod named `debug-pod`.
- Execute a command inside the container to check the file system.

<details>
<summary>Details</summary>

#### Steps to Debug

1. Access the Pod interactively:
   ```bash
   kubectl exec -it debug-pod -- /bin/sh
   ```
2. List files in the root directory:
   ```bash
   ls /
   ```
3. Exit the shell:
   ```bash
   exit
   ```

</details>

---

### 24. Attach to a Running Pod to View Real-Time Output

**Scenario**:
- Attach to a Pod named `real-time-pod` and view real-time application output.

<details>
<summary>Details</summary>

#### Steps to Attach

1. Attach to the Pod:
   ```bash
   kubectl attach real-time-pod
   ```
2. Observe real-time logs or output from the Pod.
3. Detach without stopping the container:
   ```bash
   Ctrl+C
   ```

</details>

---

### 25. Debug a Pod Using Ephemeral Containers

**Scenario**:
- Inject an ephemeral container into a running Pod named `troubleshoot-pod` to debug issues without restarting the Pod.

<details>
<summary>Details</summary>

#### Steps to Add an Ephemeral Container

1. Edit the Pod to add an ephemeral container:
   ```bash
   kubectl debug troubleshoot-pod --image=busybox --target=main-container
   ```
2. Access the ephemeral container interactively:
   ```bash
   kubectl exec -it troubleshoot-pod -c debug-container -- /bin/sh
   ```
3. Perform necessary debugging tasks, such as checking network connectivity:
   ```bash
   ping google.com
   ```

</details>

---

### 26. Debug a CrashLoopBackOff Pod

**Scenario**:
- Investigate a Pod named `crash-loop-pod` in a `CrashLoopBackOff` state to identify the root cause of the failure.

<details>
<summary>Details</summary>

#### Steps to Debug

1. View the Pod's events:
   ```bash
   kubectl describe pod crash-loop-pod
   ```
2. Access logs from the previous failed attempt:
   ```bash
   kubectl logs crash-loop-pod --previous
   ```
3. Use `kubectl debug` to inject a container and inspect the environment:
   ```bash
   kubectl debug crash-loop-pod --image=busybox
   ```

</details>

---

### 27. Analyze Network Issues Using `kubectl exec`

**Scenario**:
- Debug a Pod named `network-test-pod` to check DNS resolution and network connectivity.

<details>
<summary>Details</summary>

#### Steps to Analyze Network Issues

1. Access the Pod interactively:
   ```bash
   kubectl exec -it network-test-pod -- /bin/sh
   ```
2. Test DNS resolution:
   ```bash
   nslookup kubernetes.default
   ```
3. Check network connectivity:
   ```bash
   ping 8.8.8.8
   ```

</details>

---

### 28. Debug a Node Using `kubectl describe` and SSH

**Scenario**:
- Investigate a node named `node-1` that is experiencing resource issues.

<details>
<summary>Details</summary>

#### Steps to Debug

1. Describe the node:
   ```bash
   kubectl describe node node-1
   ```
2. Check Pod distribution and resource usage on the node.
3. SSH into the node to inspect system-level metrics:
   ```bash
   ssh user@node-1
   top
   ```

</details>

---

### 29. Monitor Pod Lifecycle Events

**Scenario**:
- Monitor events for a Pod named `event-monitor-pod` to understand its lifecycle and identify potential issues.

<details>
<summary>Details</summary>

#### Steps to Monitor Events

1. View events for the Pod:
   ```bash
   kubectl describe pod event-monitor-pod
   ```
2. Continuously monitor events for the namespace:
   ```bash
   kubectl get events -w
   ```

</details>

---

### 30. Use `kubectl port-forward` to Debug Applications

**Scenario**:
- Forward a local port to access a service running in a Pod named `app-pod` on port 8080.

<details>
<summary>Details</summary>

#### Steps to Port-Forward

1. Forward the local port 9090 to the application in the Pod:
   ```bash
   kubectl port-forward app-pod 9090:8080
   ```
2. Access the application in a browser or using `curl`:
   ```bash
   curl http://localhost:9090
   ```

</details>
---

## Notes and Tips
- **API Deprecations**: Regularly review Kubernetes release notes to stay updated on deprecations and changes.
- **Probes**: Ensure liveness probes don’t inadvertently restart healthy containers. Test probe configurations in a staging environment before applying to production.
- **CLI Tools**: Familiarize yourself with Kubernetes command-line tools to quickly identify and resolve issues.
- **Logs**: Use structured logging for better traceability and leverage tools like `stern` or `kubetail` for aggregated log viewing.
- **Debugging**: Use ephemeral containers for debugging production pods without modifying existing workloads.

## Resources
- [Kubernetes Documentation on API Deprecations](https://kubernetes.io/docs/reference/using-api/deprecation-policy/)
- [Kubernetes Probes and Health Checks](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
- [Monitoring Kubernetes with CLI Tools](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/)
- [Kubernetes Debugging Guide](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

