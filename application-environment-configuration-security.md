## CKAD Practice Questions: Application Environment, Configuration, and Security

### Overview
The "Application Environment, Configuration, and Security" domain in the CKAD (Certified Kubernetes Application Developer) certification focuses on understanding and managing key aspects of Kubernetes that ensure secure, efficient, and well-configured deployments. This involves learning about resource management, security best practices, and configuration mechanisms.

### Topics Covered
- Discover and use resources that extend Kubernetes (CRDs, Operators)
- Understand authentication, authorization, and admission control
- Understand requests, limits, and quotas
- Define resource requirements
- Understand ConfigMaps
- Create and consume Secrets
- Understand ServiceAccounts
- Understand application security (SecurityContexts, Capabilities, etc.)

### Questions

### 1. Create a CustomResourceDefinition (CRD)

**Scenario**:
- Define a CRD named `MyResource` in the group `example.com`.
- The version should be `v1` with a scope of `Namespaced`.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: myresources.example.com
spec:
  group: example.com
  names:
    kind: MyResource
    listKind: MyResourceList
    plural: myresources
    singular: myresource
  scope: Namespaced
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              replicas:
                type: integer
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f crd.yaml
   ```
2. Verify the CRD is created:
   ```bash
   kubectl get crd
   ```

</details>

---

### 2. Use a Custom Resource with a CRD

**Scenario**:
- Create a custom resource named `example-myresource` using the `MyResource` CRD.
- Set the `replicas` field to 3.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: example.com/v1
kind: MyResource
metadata:
  name: example-myresource
spec:
  replicas: 3
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f myresource.yaml
   ```
2. Verify the resource is created:
   ```bash
   kubectl get myresource
   ```
3. Inspect the resource details:
   ```bash
   kubectl describe myresource example-myresource
   ```

</details>

---

### 3. Deploy an Operator Using a Helm Chart

**Scenario**:
- Install a Kubernetes Operator (e.g., `Prometheus Operator`) using a Helm chart.
- Deploy it in the `monitoring` namespace.

<details>
<summary>Details</summary>

#### Steps to Deploy

1. Add the Helm repository:
   ```bash
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   ```
2. Install the Operator:
   ```bash
   helm install prometheus-operator prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
   ```
3. Verify the installation:
   ```bash
   kubectl get pods -n monitoring
   ```

</details>

---

### 4. Update a Custom Resource Managed by an Operator

**Scenario**:
- Modify an existing custom resource named `example-prometheus` created by the Prometheus Operator.
- Update the `replicas` field to 2.

<details>
<summary>Details</summary>

#### Steps to Update

1. Edit the custom resource:
   ```bash
   kubectl edit prometheus example-prometheus -n monitoring
   ```
2. Locate and modify the `replicas` field:
   ```yaml
   replicas: 2
   ```
3. Save the changes and verify the update:
   ```bash
   kubectl get pods -n monitoring
   ```

</details>

---

### 5. Debug an Operator-Managed Resource

**Scenario**:
- A custom resource named `example-alertmanager` managed by the Prometheus Operator is not working as expected.
- Investigate the issue by checking the Operator logs.

<details>
<summary>Details</summary>

#### Steps to Debug

1. Identify the Operator Pod:
   ```bash
   kubectl get pods -n monitoring -l app.kubernetes.io/name=kube-prometheus-stack
   ```
2. Access the logs of the Operator Pod:
   ```bash
   kubectl logs <operator-pod-name> -n monitoring
   ```
3. Investigate events related to the custom resource:
   ```bash
   kubectl describe alertmanager example-alertmanager -n monitoring
   ```

</details>

---

### 6. Explore Available CRDs in the Cluster

**Scenario**:
- List all CRDs in the cluster and identify resources managed by Operators.

<details>
<summary>Details</summary>
#### Steps to Explore

1. List all CRDs:
   ```bash
   kubectl get crds
   ```
2. Describe a specific CRD:
   ```bash
   kubectl describe crd <crd-name>
   ```
3. Identify namespaces and resources associated with the CRD:
   ```bash
   kubectl get <resource> -A
   ```

</details>
---

### 7. Verify Cluster Authentication

**Scenario**:
- Use a kubeconfig file to authenticate to a Kubernetes cluster.
- Verify that you can connect to the cluster and list nodes.

<details>
<summary>Details</summary>

#### Steps to Verify Authentication

1. Check the current context in your kubeconfig:
   ```bash
   kubectl config current-context
   ```
2. List all nodes in the cluster:
   ```bash
   kubectl get nodes
   ```
3. Switch contexts if required:
   ```bash
   kubectl config use-context <context-name>
   ```

</details>

---

### 8. Create and Test Role-Based Access Control (RBAC)

**Scenario**:
- Create an RBAC role named `pod-reader` to allow read access to Pods.
- Bind the role to a user named `developer` in the `development` namespace.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: development
  name: pod-reader-binding
subjects:
- kind: User
  name: developer
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

#### Steps to Apply and Test

1. Apply the RBAC configuration:
   ```bash
   kubectl apply -f rbac-role.yaml
   ```
2. Impersonate the user to test access:
   ```bash
   kubectl auth can-i get pods --namespace=development --as=developer
   ```
3. Verify the user’s limited access by listing Pods:
   ```bash
   kubectl get pods -n development --as=developer
   ```

</details>

---

### 9. Inspect Admission Controller Behavior

**Scenario**:
- Investigate admission control policies in the cluster to determine why a Pod creation request was rejected.

<details>
<summary>Details</summary>

#### Steps to Inspect Admission Controller

1. Enable API server audit logs to capture rejected requests:
   ```bash
   --audit-log-path=/var/log/kubernetes/audit.log
   --audit-policy-file=/etc/kubernetes/audit-policy.yaml
   ```
2. Inspect audit logs for denied requests:
   ```bash
   grep "deny" /var/log/kubernetes/audit.log
   ```
3. Check active admission controllers:
   ```bash
   kubectl get --raw "/configz" | jq .admissionControl
   ```

</details>

---

### 10. Test Service Account Authentication

**Scenario**:
- Create a ServiceAccount named `test-sa` in the `default` namespace and use it to authenticate requests.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: test-sa
---
apiVersion: v1
kind: Pod
metadata:
  name: test-sa-pod
spec:
  serviceAccountName: test-sa
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh", "-c", "while true; do echo hello; sleep 10; done"]
```

#### Steps to Apply and Test

1. Apply the ServiceAccount and Pod configuration:
   ```bash
   kubectl apply -f serviceaccount.yaml
   ```
2. Verify the ServiceAccount token is mounted in the Pod:
   ```bash
   kubectl exec test-sa-pod -- cat /var/run/secrets/kubernetes.io/serviceaccount/token
   ```
3. Use the token for authentication.

</details>

---

### 11. Validate Authorization Policies

**Scenario**:
- Validate if a user named `team-member` can create deployments in the `production` namespace.

<details>
<summary>Details</summary>

#### Steps to Validate Authorization

1. Test the user’s permissions:
   ```bash
   kubectl auth can-i create deployments --namespace=production --as=team-member
   ```
2. Check the Role or ClusterRole associated with the user:
   ```bash
   kubectl get rolebinding,clusterrolebinding -n production | grep team-member
   ```
3. If unauthorized, update the RBAC configuration to grant access.

</details>

---

### 12. Simulate and Audit Access Denials

**Scenario**:
- A user named `audit-user` reports access issues. Simulate and audit their denied requests in the `testing` namespace.

<details>
<summary>Details</summary>

#### Steps to Simulate and Audit

1. Impersonate the user and test access:
   ```bash
   kubectl auth can-i get pods --namespace=testing --as=audit-user
   ```
2. Enable audit logs to capture denied requests.
3. Analyze the audit logs for `audit-user`:
   ```bash
   grep "audit-user" /var/log/kubernetes/audit.log
   ```

</details>

---

### 13. Configure Resource Requests and Limits for a Pod

**Scenario**:
- Create a Pod named `resource-limits-pod` with a `busybox` container.
- Set CPU requests to `100m` and limits to `200m`.
- Set memory requests to `128Mi` and limits to `256Mi`.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-limits-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh", "-c", "while true; do echo hello; sleep 10; done"]
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "200m"
        memory: "256Mi"
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f resource-limits-pod.yaml
   ```
2. Verify the resource requests and limits:
   ```bash
   kubectl describe pod resource-limits-pod
   ```

</details>

---

### 14. Enforce Resource Quotas in a Namespace

**Scenario**:
- Create a namespace named `quota-namespace`.
- Apply a ResourceQuota to limit CPU usage to `2` cores and memory to `4Gi`.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: quota-namespace
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: quota-namespace
spec:
  hard:
    requests.cpu: "2"
    requests.memory: "4Gi"
    limits.cpu: "2"
    limits.memory: "4Gi"
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f resource-quota.yaml
   ```
2. Verify the quota:
   ```bash
   kubectl get resourcequota -n quota-namespace
   ```
3. Test resource creation to ensure it complies with the quota.

</details>

---

### 15. Create a LimitRange to Restrict Resource Usage

**Scenario**:
- Create a LimitRange in the `dev-namespace` to enforce default CPU and memory limits for Pods.
- Set default CPU requests to `200m` and limits to `500m`.
- Set default memory requests to `256Mi` and limits to `1Gi`.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: compute-limit-range
  namespace: dev-namespace
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "1Gi"
    defaultRequest:
      cpu: "200m"
      memory: "256Mi"
    type: Container
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f limit-range.yaml
   ```
2. Verify the LimitRange:
   ```bash
   kubectl describe limitrange compute-limit-range -n dev-namespace
   ```
3. Test Pod creation to observe default values applied.

</details>

---

### 16. Debug a Pod Failing Due to Quota Constraints

**Scenario**:
- A Pod named `quota-fail-pod` is failing to schedule in the `test-namespace` due to exceeded quota.
- Debug the issue and identify the quota violation.

<details>
<summary>Details</summary>

#### Steps to Debug

1. Describe the Pod to inspect the error:
   ```bash
   kubectl describe pod quota-fail-pod -n test-namespace
   ```
2. Check the ResourceQuota in the namespace:
   ```bash
   kubectl get resourcequota -n test-namespace
   ```
3. Adjust the Pod’s resource requests or update the quota as needed.

</details>

---

### 17. Monitor Namespace Resource Usage

**Scenario**:
- Monitor the resource usage in a namespace named `monitor-namespace` to ensure it adheres to quota limits.

<details>
<summary>Details</summary>

#### Steps to Monitor

1. View resource usage:
   ```bash
   kubectl describe resourcequota -n monitor-namespace
   ```
2. List resource consumption for all Pods in the namespace:
   ```bash
   kubectl top pods -n monitor-namespace
   ```
3. Take corrective actions if usage exceeds limits or requests.

</details>

---

### 18. Enforce CPU and Memory Limits for All Pods in a Cluster

**Scenario**:
- Apply a Cluster-wide LimitRange to enforce default resource limits for all namespaces.
- Set default CPU limits to `1` core and memory limits to `2Gi`.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cluster-compute-limits
  namespace: default
spec:
  limits:
  - default:
      cpu: "1"
      memory: "2Gi"
    type: Container
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f cluster-limit-range.yaml
   ```
2. Verify the LimitRange:
   ```bash
   kubectl describe limitrange cluster-compute-limits
   ```
3. Observe the effect on newly created Pods.

</details>

---

### 19. Set Resource Requests for a Deployment

**Scenario**:
- Create a Deployment named `web-app` with 3 replicas.
- Configure each container to request `250m` CPU and `512Mi` memory.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        resources:
          requests:
            cpu: "250m"
            memory: "512Mi"
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f web-app.yaml
   ```
2. Verify resource requests:
   ```bash
   kubectl describe deployment web-app
   ```

</details>

---

### 20. Define Resource Limits for Pods in a Namespace

**Scenario**:
- Create a LimitRange in the `test-namespace` to restrict maximum CPU usage to `1` core and memory usage to `2Gi` for Pods.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: pod-limits
  namespace: test-namespace
spec:
  limits:
  - max:
      cpu: "1"
      memory: "2Gi"
    type: Pod
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f limit-range-pod.yaml
   ```
2. Verify the LimitRange:
   ```bash
   kubectl describe limitrange pod-limits -n test-namespace
   ```
3. Test Pod creation to observe enforced limits.

</details>

---

### 21. Enforce Default Requests for Containers

**Scenario**:
- Apply a LimitRange in the `default-namespace` to set default CPU requests to `100m` and memory requests to `256Mi` for containers.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: container-defaults
  namespace: default-namespace
spec:
  limits:
  - defaultRequest:
      cpu: "100m"
      memory: "256Mi"
    type: Container
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f container-defaults.yaml
   ```
2. Verify the LimitRange:
   ```bash
   kubectl describe limitrange container-defaults -n default-namespace
   ```
3. Observe applied defaults for newly created containers.

</details>

---

### 22. Debug Resource Requests for a Pod

**Scenario**:
- A Pod named `high-cpu-pod` is failing to schedule due to insufficient CPU resources.
- Investigate and resolve the issue.

<details>
<summary>Details</summary>

#### Steps to Debug

1. Describe the Pod to inspect the error:
   ```bash
   kubectl describe pod high-cpu-pod
   ```
2. Check node resource availability:
   ```bash
   kubectl describe node <node-name>
   ```
3. Reduce the Pod's CPU requests or update cluster capacity as needed.

</details>

---

### 23. Monitor Resource Usage Against Requests

**Scenario**:
- Monitor resource usage for a Deployment named `resource-monitor`.
- Compare actual usage against defined requests and limits.

<details>
<summary>Details</summary>

#### Steps to Monitor

1. View resource usage for Pods in the Deployment:
   ```bash
   kubectl top pods -l app=resource-monitor
   ```
2. Check resource requests and limits:
   ```bash
   kubectl describe deployment resource-monitor
   ```
3. Analyze discrepancies and adjust resource definitions as needed.

</details>

---

### 24. Apply Resource Requests and Limits Across Multiple Containers

**Scenario**:
- Create a Pod named `multi-container-pod` with two containers.
- Configure requests and limits for each container as follows:
  - Container `app1`: `200m` CPU and `512Mi` memory.
  - Container `app2`: `300m` CPU and `1Gi` memory.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: app1
    image: busybox
    resources:
      requests:
        cpu: "200m"
        memory: "512Mi"
      limits:
        cpu: "400m"
        memory: "1Gi"
  - name: app2
    image: busybox
    resources:
      requests:
        cpu: "300m"
        memory: "1Gi"
      limits:
        cpu: "600m"
        memory: "2Gi"
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f multi-container-pod.yaml
   ```
2. Verify resource definitions for each container:
   ```bash
   kubectl describe pod multi-container-pod
   ```

</details>

---

### 25. Create a ConfigMap and Inject It into a Pod as Environment Variables

**Scenario**:
- You have an application that requires environment variables for `APP_ENV` and `LOG_LEVEL`.
- Create a ConfigMap named `app-config` to store these variables and inject them into a Pod named `env-pod`.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  APP_ENV: production
  LOG_LEVEL: INFO
---
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "env"]
    env:
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_ENV
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: LOG_LEVEL
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f env-pod.yaml
   ```
2. Verify the environment variables in the Pod:
   ```bash
   kubectl logs env-pod
   ```

</details>

---

### 26. Mount a ConfigMap as Files in a Volume

**Scenario**:
- Create a ConfigMap named `file-config` containing configuration data.
- Mount the ConfigMap into a Pod so that its keys become file names in the `/etc/config` directory.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: file-config
  namespace: default
data:
  app.properties: |
    app.name=example
    app.version=1.0
  database.properties: |
    db.name=testdb
    db.host=localhost
---
apiVersion: v1
kind: Pod
metadata:
  name: volume-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "cat /etc/config/app.properties"]
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: file-config
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f volume-pod.yaml
   ```
2. Verify the ConfigMap is mounted as files:
   ```bash
   kubectl logs volume-pod
   ```

</details>

---

### 27. Update a ConfigMap and Verify Changes in a Pod

**Scenario**:
- Create a ConfigMap named `live-config` and mount it as a volume in a Pod.
- Update the ConfigMap dynamically and verify the changes are reflected without restarting the Pod.


<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: live-config
  namespace: default
data:
  message: "Hello, Kubernetes!"
---
apiVersion: v1
kind: Pod
metadata:
  name: live-config-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "while true; do cat /etc/live-config/message; sleep 5; done"]
    volumeMounts:
    - name: config-volume
      mountPath: /etc/live-config
  volumes:
  - name: config-volume
    configMap:
      name: live-config
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f live-config-pod.yaml
   ```
2. Update the ConfigMap:
   ```bash
   kubectl edit configmap live-config
   ```
3. Verify the changes dynamically in the Pod:
   ```bash
   kubectl logs live-config-pod
   ```

</details>

---

### 28. Use a ConfigMap to Store JSON Configuration Data

**Scenario**:
- Create a ConfigMap named `json-config` to store JSON configuration.
- Mount it as a file in a Pod to provide configuration data for an application.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: json-config
  namespace: default
data:
  config.json: |
    {
      "name": "example",
      "debug": true
    }
---
apiVersion: v1
kind: Pod
metadata:
  name: json-config-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "cat /etc/config/config.json"]
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: json-config
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f json-config-pod.yaml
   ```
2. Verify the JSON configuration is accessible:
   ```bash
   kubectl logs json-config-pod
   ```

</details>

---

### 29. Use ConfigMaps with Deployment Environment Variables

**Scenario**:
- Create a ConfigMap named `deployment-env-config` and use it to populate environment variables for a Deployment named `env-deployment`.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: deployment-env-config
  namespace: default
data:
  LOG_LEVEL: DEBUG
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: env-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: env-app
  template:
    metadata:
      labels:
        app: env-app
    spec:
      containers:
      - name: app
        image: busybox
        command: ["/bin/sh", "-c", "env"]
        envFrom:
        - configMapRef:
            name: deployment-env-config
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f env-deployment.yaml
   ```
2. Verify the environment variables are loaded:
   ```bash
   kubectl logs -l app=env-app
   ```

</details>

---

### 30. Debug a Pod Failing Due to Missing ConfigMap

**Scenario**:
- A Pod named `missing-configmap-pod` fails to start due to a missing ConfigMap.
- Debug and resolve the issue.

<details>
<summary>Details</summary>

#### Steps to Debug

1. Inspect the Pod events:
   ```bash
   kubectl describe pod missing-configmap-pod
   ```
2. Check if the required ConfigMap exists:
   ```bash
   kubectl get configmap
   ```
3. Create or update the ConfigMap and reapply the Pod.

</details>

---

### 31. Validate ConfigMap Usage Across Namespaces

**Scenario**:
- Use a ConfigMap named `shared-config` in multiple namespaces to provide common settings.

<details>
<summary>Details</summary>

#### Steps to Validate

1. Export the ConfigMap from the `default` namespace:
   ```bash
   kubectl get configmap shared-config -n default -o yaml > shared-config.yaml
   ```
2. Apply the ConfigMap to another namespace:
   ```bash
   kubectl apply -f shared-config.yaml -n test-namespace
   ```
3. Verify the ConfigMap is used in the target namespace:
   ```bash
   kubectl describe configmap shared-config -n test-namespace
   ```

</details>

---

### 32. Enforce Rolling Updates for ConfigMap Changes

**Scenario**:
- Use a ConfigMap named `rolling-update-config` in a Deployment.
- Trigger a rolling update for the Deployment when the ConfigMap is updated.

<details>
<summary>Details</summary>

#### Steps to Enforce Rolling Updates

1. Add an annotation to the Deployment template to track ConfigMap updates:
   ```yaml
   spec:
     template:
       metadata:
         annotations:
           config-update: "{{now | date:'2006-01-02T15:04:05Z07:00'}}"
   ```
2. Update the ConfigMap and redeploy:
   ```bash
   kubectl apply -f rolling-update-config.yaml
   ```
3. Verify that Pods in the Deployment are updated:
   ```bash
   kubectl rollout status deployment rolling-update-deployment
   ```

</details>

---

### 33. Create a Secret and Inject It as Environment Variables

**Scenario**:
- Create a Secret named `db-credentials` with `username` and `password` keys.
- Inject the Secret into a Pod named `db-pod` as environment variables.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
  namespace: default
data:
  username: dXNlcm5hbWU=  # Base64 encoded value of 'username'
  password: cGFzc3dvcmQ=  # Base64 encoded value of 'password'
---
apiVersion: v1
kind: Pod
metadata:
  name: db-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "env"]
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f db-pod.yaml
   ```
2. Verify the environment variables:
   ```bash
   kubectl logs db-pod
   ```

</details>

---

### 34. Mount a Secret as Files in a Volume

**Scenario**:
- Create a Secret named `tls-cert` with `cert.pem` and `key.pem` keys.
- Mount the Secret as a volume in a Pod named `tls-pod`.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-cert
  namespace: default
data:
  cert.pem: BASE64_CERT_CONTENT  # Replace with actual base64 encoded content
  key.pem: BASE64_KEY_CONTENT    # Replace with actual base64 encoded content
---
apiVersion: v1
kind: Pod
metadata:
  name: tls-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "ls /etc/tls"]
    volumeMounts:
    - name: tls-volume
      mountPath: /etc/tls
      readOnly: true
  volumes:
  - name: tls-volume
    secret:
      secretName: tls-cert
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f tls-pod.yaml
   ```
2. Verify the Secret is mounted as files:
   ```bash
   kubectl logs tls-pod
   ```

</details>

---

### 35. Create an Opaque Secret and Use It in a Deployment

**Scenario**:
- Create an opaque Secret named `api-keys`.
- Use it in a Deployment named `api-deployment` to configure the application.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: api-keys
  namespace: default
data:
  API_KEY: QVBJS0VZMTIz  # Base64 encoded value of 'APIKEY123'
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
      - name: app
        image: busybox
        command: ["/bin/sh", "-c", "env"]
        env:
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: api-keys
              key: API_KEY
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f api-deployment.yaml
   ```
2. Verify the Secret is injected as environment variables:
   ```bash
   kubectl logs -l app=api
   ```

</details>

---

### 36. Rotate a Secret Without Restarting Pods

**Scenario**:
- Update the Secret named `db-credentials` and verify that changes are reflected in the Pod without a restart.

<details>
<summary>Details</summary>

#### Steps to Rotate Secrets

1. Update the Secret data:
   ```bash
   kubectl edit secret db-credentials
   ```
2. Verify that the updated Secret is mounted in the Pod:
   ```bash
   kubectl exec db-pod -- cat /path/to/mounted/secret
   ```
3. If required, restart Pods to reload environment variables.

</details>

---

### 37. Use Secrets with Kubernetes Service Accounts

**Scenario**:
- Create a Secret named `sa-token-secret` and associate it with a ServiceAccount named `custom-sa`.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: sa-token-secret
  annotations:
    kubernetes.io/service-account.name: custom-sa
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: custom-sa
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f sa-secret.yaml
   ```
2. Verify the ServiceAccount and associated Secret:
   ```bash
   kubectl describe sa custom-sa
   ```

</details>

---

### 38. Debug a Pod Failing Due to Missing Secret

**Scenario**:
- A Pod named `secret-fail-pod` fails to start because a referenced Secret is missing.
- Debug and resolve the issue.

<details>
<summary>Details</summary>

#### Steps to Debug

1. Describe the Pod to check events:
   ```bash
   kubectl describe pod secret-fail-pod
   ```
2. Verify the Secret exists:
   ```bash
   kubectl get secret
   ```
3. Create or update the Secret and reapply the Pod.

</details>

---

### 39. Use Secrets in Multi-Container Pods

**Scenario**:
- Create a Secret named `multi-container-secret`.
- Use it in a Pod with two containers, each accessing the Secret differently.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: multi-container-secret
  namespace: default
data:
  shared-key: U0VDUkVUS0VZ  # Base64 encoded value of 'SECRETKEY'
---
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: app1
    image: busybox
    command: ["/bin/sh", "-c", "env"]
    env:
    - name: SHARED_KEY
      valueFrom:
        secretKeyRef:
          name: multi-container-secret
          key: shared-key
  - name: app2
    image: busybox
    command: ["/bin/sh", "-c", "cat /etc/secret-key"]
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secret-key
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: multi-container-secret
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f multi-container-pod.yaml
   ```
2. Verify both containers access the Secret:
   ```bash
   kubectl logs multi-container-pod -c app1
   kubectl logs multi-container-pod -c app2
   ```

</details>

---

### 40. Validate Encrypted Secret Storage

**Scenario**:
- Verify that Secrets are encrypted at rest in an etcd database.

<details>
<summary>Details</summary>

#### Steps to Validate

1. Ensure encryption configuration is enabled in the cluster.
2. View encrypted data in etcd:
   ```bash
   ETCDCTL_API=3 etcdctl get /registry/secrets/default/db-credentials --prefix --keys-only
   ```
3. Confirm that Secret values are encrypted.

</details>

---

### 41. Create and Use a Custom ServiceAccount for a Pod

**Scenario**:
- Create a ServiceAccount named `custom-sa` in the `default` namespace.
- Use the ServiceAccount in a Pod named `sa-pod` to access cluster resources.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: custom-sa
  namespace: default
---
apiVersion: v1
kind: Pod
metadata:
  name: sa-pod
spec:
  serviceAccountName: custom-sa
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "echo Hello from custom ServiceAccount"]
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f sa-pod.yaml
   ```
2. Verify the Pod is using the custom ServiceAccount:
   ```bash
   kubectl describe pod sa-pod
   ```

</details>

---

### 42. Restrict a ServiceAccount to a Namespace

**Scenario**:
- Create a ServiceAccount named `restricted-sa` in the `dev` namespace.
- Ensure the ServiceAccount can only list Pods in the `dev` namespace.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: restricted-sa
  namespace: dev
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: dev
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: dev
subjects:
- kind: ServiceAccount
  name: restricted-sa
  namespace: dev
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f restricted-sa.yaml
   ```
2. Verify the ServiceAccount permissions:
   ```bash
   kubectl auth can-i list pods --as=system:serviceaccount:dev:restricted-sa -n dev
   ```

</details>

---

### 43. Use a ServiceAccount with an External Secret

**Scenario**:
- Create a ServiceAccount named `external-sa`.
- Associate it with a Secret named `sa-token`.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: sa-token
  annotations:
    kubernetes.io/service-account.name: external-sa
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: external-sa
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f external-sa.yaml
   ```
2. Verify the ServiceAccount is associated with the Secret:
   ```bash
   kubectl describe sa external-sa
   ```

</details>

---

### 44. Debug a Pod Using a Specific ServiceAccount

**Scenario**:
- A Pod named `debug-sa-pod` is failing to access resources due to insufficient permissions.
- Debug the issue and resolve it by assigning an appropriate Role to the ServiceAccount.

<details>
<summary>Details</summary>

#### Steps to Debug

1. Check the ServiceAccount used by the Pod:
   ```bash
   kubectl describe pod debug-sa-pod
   ```
2. Verify the ServiceAccount permissions:
   ```bash
   kubectl auth can-i <action> <resource> --as=system:serviceaccount:<namespace>:<serviceaccount>
   ```
3. Update the Role or RoleBinding to grant required permissions:
   ```bash
   kubectl edit rolebinding <rolebinding-name>
   ```

</details>

---

### 45. Automate ServiceAccount Creation with Annotations

**Scenario**:
- Create a ServiceAccount named `auto-sa` with annotations to automatically associate a Secret for authentication.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: auto-sa
  annotations:
    example.com/auto-secret: "true"
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f auto-sa.yaml
   ```
2. Verify the Secret is associated with the ServiceAccount:
   ```bash
   kubectl describe sa auto-sa
   ```

</details>

---

### 46. Assign a ServiceAccount to a StatefulSet

**Scenario**:
- Assign a ServiceAccount named `stateful-sa` to a StatefulSet named `stateful-app`.
- Ensure the StatefulSet uses the ServiceAccount for cluster resource access.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: stateful-sa
  namespace: default
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: stateful-app
spec:
  serviceName: "stateful-service"
  replicas: 3
  serviceAccountName: stateful-sa
  selector:
    matchLabels:
      app: stateful
  template:
    metadata:
      labels:
        app: stateful
    spec:
      containers:
      - name: app
        image: busybox
        command: ["/bin/sh", "-c", "while true; do echo Hello from StatefulSet; sleep 10; done"]
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f stateful-app.yaml
   ```
2. Verify the StatefulSet is using the ServiceAccount:
   ```bash
   kubectl describe statefulset stateful-app
   ```

</details>

---

### 47. Test Cluster-Wide Permissions of a ServiceAccount

**Scenario**:
- Create a ClusterRoleBinding to grant a ServiceAccount named `admin-sa` cluster-wide administrative permissions.
- Verify the permissions.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-sa
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-role-binding
subjects:
- kind: ServiceAccount
  name: admin-sa
  namespace: default
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f admin-sa.yaml
   ```
2. Verify the permissions:
   ```bash
   kubectl auth can-i '*' '*' --as=system:serviceaccount:default:admin-sa
   ```

</details>

---

### 48. Use a ServiceAccount in a Job

**Scenario**:
- Create a ServiceAccount named `job-sa` and use it in a Kubernetes Job to access specific resources.

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: job-sa
---
apiVersion: batch/v1
kind: Job
metadata:
  name: job-with-sa
spec:
  template:
    spec:
      serviceAccountName: job-sa
      containers:
      - name: app
        image: busybox
        command: ["/bin/sh", "-c", "echo Accessing resources with Job"]
      restartPolicy: Never
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f job-with-sa.yaml
   ```
2. Verify the Job is using the ServiceAccount:
   ```bash
   kubectl describe job job-with-sa
   ```

</details>

---

### 49. Configure a Pod with a SecurityContext

**Scenario**:
- Create a Pod named `secure-pod` with a SecurityContext.
- Ensure the container runs as a non-root user and disables privilege escalation.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "echo Hello Secure World"]
    securityContext:
      allowPrivilegeEscalation: false
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f secure-pod.yaml
   ```
2. Verify the SecurityContext:
   ```bash
   kubectl describe pod secure-pod
   ```

</details>

---

### 50. Add Capabilities to a Container

**Scenario**:
- Create a Pod named `capability-pod`.
- Add the `NET_ADMIN` and `SYS_TIME` capabilities to the container.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: capability-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "echo Adding capabilities"]
    securityContext:
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f capability-pod.yaml
   ```
2. Verify the added capabilities:
   ```bash
   kubectl describe pod capability-pod
   ```

</details>

---

### 51. Restrict a Pod to a Read-Only Filesystem

**Scenario**:
- Create a Pod named `read-only-pod`.
- Ensure the container has a read-only root filesystem.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: read-only-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "touch /tmp/test"]
    securityContext:
      readOnlyRootFilesystem: true
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f read-only-pod.yaml
   ```
2. Verify the Pod's behavior:
   ```bash
   kubectl logs read-only-pod
   ```

</details>

---

### 52. Apply SELinux Labels to a Pod

**Scenario**:
- Create a Pod named `selinux-pod`.
- Configure SELinux labels for enhanced security.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: selinux-pod
spec:
  securityContext:
    seLinuxOptions:
      level: "s0:c123,c456"
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "echo SELinux Configured"]
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f selinux-pod.yaml
   ```
2. Verify SELinux labels:
   ```bash
   kubectl describe pod selinux-pod
   ```

</details>

---

### 53. Configure Network Policies for Pod Isolation

**Scenario**:
- Create a NetworkPolicy named `deny-all` to deny all traffic to a Pod named `isolated-pod`.
- Allow traffic only from Pods with the label `role=frontend`.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: isolated
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f deny-all.yaml
   ```
2. Test connectivity to the Pod:
   ```bash
   kubectl exec <frontend-pod> -- curl <isolated-pod>
   ```

</details>

---

### 54. Disable Privileged Mode for a Container

**Scenario**:
- Create a Pod named `no-privilege-pod`.
- Ensure the container cannot run in privileged mode.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-privilege-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "echo Privileged mode disabled"]
    securityContext:
      privileged: false
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f no-privilege-pod.yaml
   ```
2. Verify the Pod's SecurityContext:
   ```bash
   kubectl describe pod no-privilege-pod
   ```

</details>

---

### 55. Enforce CPU and Memory Limits Using SecurityContext

**Scenario**:
- Create a Pod named `resource-limited-pod`.
- Configure resource requests and limits for CPU and memory.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-limited-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "echo Resource Limits Enforced"]
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "200m"
        memory: "256Mi"
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f resource-limited-pod.yaml
   ```
2. Verify the resource limits:
   ```bash
   kubectl describe pod resource-limited-pod
   ```

</details>

---

### 56. Use AppArmor Profiles for a Pod

**Scenario**:
- Create a Pod named `apparmor-pod`.
- Assign an AppArmor profile to enhance security.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: apparmor-pod
  annotations:
    container.apparmor.security.beta.kubernetes.io/app: localhost/k8s-default
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "echo AppArmor Profile Applied"]
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f apparmor-pod.yaml
   ```
2. Verify the AppArmor profile is applied:
   ```bash
   kubectl describe pod apparmor-pod
   ```

</details>

---

### Notes and Tips
1. **CRDs and Operators**: Understand how Custom Resource Definitions (CRDs) extend Kubernetes’ functionality and how Operators automate application management.
2. **Authentication and Authorization**: Review how Kubernetes handles authentication (e.g., via tokens) and implements RBAC for authorization.
3. **Resource Management**:
   - Practice setting `requests` and `limits` for CPU and memory in pod specifications.
   - Understand how resource quotas are applied at the namespace level.
4. **ConfigMaps and Secrets**:
   - Use ConfigMaps to decouple configuration artifacts from application logic.
   - Secure sensitive data using Secrets and ensure they are properly consumed.
5. **ServiceAccounts**: Assign specific ServiceAccounts to pods for fine-grained access control.
6. **SecurityContexts**: Explore the use of SecurityContexts to set permissions and capabilities for pods and containers.

### Resources
- [Kubernetes Official Documentation](https://kubernetes.io/docs/)
- [CKAD Exam Curriculum](https://github.com/cncf/curriculum/blob/master/CKAD_Curriculum_v1.31.pdf)
- [Kubernetes Security Best Practices](https://kubernetes.io/docs/concepts/security/overview/)
- Practice Labs: [Play with Kubernetes](https://labs.play-with-k8s.com/)

