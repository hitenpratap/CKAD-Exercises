# CKAD Practice Questions: Application Deployment

## Overview
This section focuses on the Application Deployment domain of the Certified Kubernetes Application Developer (CKAD) exam. It covers strategies, tools, and techniques for deploying and managing applications in Kubernetes clusters. Understanding deployment strategies, rolling updates, and tools like Helm and Kustomize is crucial for success.

---

## Topics Covered
- Kubernetes deployment strategies (e.g., blue/green, canary)
- Rolling updates and Deployment configurations
- Using Helm for application deployment

---

## Questions

### 1. Implement a blue/green deployment strategy

**Scenario**:
- Deploy two versions of the same application named `app-blue` (version 1.0) and `app-green` (version 2.0).
- Traffic should initially flow to the `app-blue` deployment.
- Switch traffic to `app-green` by updating the Service selector.

#### Solution:

<details>
<summary>Declarative YAML Configuration</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: blue
  template:
    metadata:
      labels:
        app: my-app
        version: blue
    spec:
      containers:
      - name: app-container
        image: my-app:1.0
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: green
  template:
    metadata:
      labels:
        app: my-app
        version: green
    spec:
      containers:
      - name: app-container
        image: my-app:2.0
---
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
    version: blue
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

#### Steps to Apply:
1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f blue-green-deployment.yaml
   ```
2. Switch traffic to the `app-green` deployment by updating the Service selector:
   ```bash
   kubectl patch service my-app-service -p '{"spec":{"selector":{"app":"my-app","version":"green"}}}'
   ```
</details>

---

### 2. Implement a canary deployment strategy

**Scenario**:
- Deploy a new version of an application named `app-canary`.
- Initially, only 10% of traffic should flow to the new version.
- Gradually increase traffic to 100% if the new version is stable.

#### Solution:

<details>
<summary>Declarative YAML Configuration</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-stable
spec:
  replicas: 9
  selector:
    matchLabels:
      app: my-app
      version: stable
  template:
    metadata:
      labels:
        app: my-app
        version: stable
    spec:
      containers:
      - name: app-container
        image: my-app:1.0
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
      version: canary
  template:
    metadata:
      labels:
        app: my-app
        version: canary
    spec:
      containers:
      - name: app-container
        image: my-app:2.0
---
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

#### Steps to Apply:
1. Deploy the stable and canary versions:
   ```bash
   kubectl apply -f canary-deployment.yaml
   ```
2. Gradually scale up the canary deployment and scale down the stable deployment:
   ```bash
   kubectl scale deployment app-canary --replicas=5
   kubectl scale deployment app-stable --replicas=5
   ```
</details>
---

### 3. Configure a rolling update for an application

**Scenario**:
- Update the image of an existing deployment named `rolling-update-app` from `nginx:1.19` to `nginx:1.21`.
- Ensure there is no downtime during the update.

#### Solution:

<details>
<summary>Declarative YAML Configuration</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rolling-update-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: rolling-app
  template:
    metadata:
      labels:
        app: rolling-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
      strategy:
        type: RollingUpdate
        rollingUpdate:
          maxUnavailable: 1
          maxSurge: 1
```

#### Steps to Apply:
1. Save and apply the YAML file:
   ```bash
   kubectl apply -f rolling-update.yaml
   ```
2. Verify the rollout:
   ```bash
   kubectl rollout status deployment/rolling-update-app
   ```
</details>

---

### 4. Deploy an application using Helm

**Scenario**:
- Use Helm to deploy an application named `helm-app`.
- Configure the Helm chart to use `nginx:latest` and 3 replicas.

#### Solution:

<details>
<summary>Helm Command</summary>

```bash
helm create helm-app
```
Edit the `values.yaml` file:
```yaml
replicaCount: 3
image:
  repository: nginx
  tag: latest
  pullPolicy: IfNotPresent
```
Deploy the Helm chart:
```bash
helm install helm-app ./helm-app
```

#### Steps to Verify:
1. Verify the Helm release:
   ```bash
   helm list
   ```
2. Check the pods:
   ```bash
   kubectl get pods -l app.kubernetes.io/name=helm-app
   ```
</details>

---

### 5. Rollback a failed deployment

**Scenario**:
- Rollback a deployment named `test-deploy` to its previous stable version after a failed update.

#### Solution:

<details>
<summary>Command</summary>

Rollback the deployment:
```bash
kubectl rollout undo deployment/test-deploy
```

#### Steps to Verify:
1. Check the rollout history:
   ```bash
   kubectl rollout history deployment/test-deploy
   ```
2. Ensure the pods are running the previous stable version:
   ```bash
   kubectl get pods
   ```
</details>

### 6. Perform a staged rollout with incremental updates

**Scenario**:
- Deploy an application named `staged-rollout-app` with an initial 25% traffic allocation to the new version.
- Incrementally roll out the new version while ensuring no downtime.

<details>
<summary>Declarative YAML Configuration</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: staged-rollout-app
spec:
  replicas: 4
  selector:
    matchLabels:
      app: staged-rollout
  template:
    metadata:
      labels:
        app: staged-rollout
    spec:
      containers:
      - name: app-container
        image: staged-rollout:1.0
      strategy:
        type: RollingUpdate
        rollingUpdate:
          maxUnavailable: 1
          maxSurge: 1
```

#### Steps to Apply

Save the YAML file and deploy the initial version:

```bash
kubectl apply -f staged-rollout.yaml
```

Incrementally update the deployment to the new version:

```bash
kubectl set image deployment/staged-rollout-app app-container=staged-rollout:1.1
```

Monitor the rollout status:

```bash
kubectl rollout status deployment/staged-rollout-app
```

</details>

---

### 7. Deploy an application using Helm with custom values

**Scenario**:
- Use Helm to deploy an application named `custom-helm-app`.
- Override the default values for replica count (5) and image version (`nginx:1.20`).

<details>
<summary>Command</summary>


#### Create and customize the Helm chart

```bash
helm create custom-helm-app
```

Edit the `values.yaml` file:

```yaml
replicaCount: 5
image:
  repository: nginx
  tag: 1.20
  pullPolicy: IfNotPresent
```

Deploy the Helm chart with the custom values:

```bash
helm install custom-helm-app ./custom-helm-app
```

#### Steps to Verify

1. List Helm releases:
   ```bash
   helm list
   ```
2. Check the pods created by the Helm deployment:
   ```bash
   kubectl get pods -l app.kubernetes.io/name=custom-helm-app
   ```

</details>

---

### 8. Rollback a Helm release to a previous revision

**Scenario**:
- A Helm release named `example-app` was updated to a faulty version.
- Rollback the release to the previous stable revision.

<details>
<summary>Command</summary>

#### Rollback the Helm release

```bash
helm rollback example-app 1
```

#### Steps to Verify

1. Check the Helm release history:
   ```bash
   helm history example-app
   ```
2. Confirm the pods reflect the rollback:
   ```bash
   kubectl get pods -l app.kubernetes.io/name=example-app
   ```

</details>

---

### 9. Perform a rolling update for a StatefulSet

**Scenario**:
- Update the image of an existing StatefulSet named `stateful-app` from `redis:6.2` to `redis:7.0`.
- Ensure each pod is updated sequentially.

<details>
<summary>Declarative YAML Configuration</summary>

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: stateful-app
spec:
  serviceName: "stateful-service"
  replicas: 3
  selector:
    matchLabels:
      app: stateful
  template:
    metadata:
      labels:
        app: stateful
    spec:
      containers:
      - name: redis
        image: redis:7.0
```

#### Steps to Apply

Save the YAML file and update the StatefulSet:

```bash
kubectl apply -f stateful-update.yaml
```

Monitor the pod updates sequentially:

```bash
kubectl rollout status statefulset/stateful-app
```

</details>

---

### 10. Deploy a workload using Helm with external secrets

**Scenario**:
- Deploy an application named `secret-helm-app` using Helm.
- Use external Secrets for database credentials.

<details>
<summary>External Secret YAML Configuration</summary>

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
  namespace: default
stringData:
  username: admin
  password: securepassword
```

Deploy the Secret:

```bash
kubectl apply -f external-secret.yaml
```

#### Helm Chart Configuration

Customize `values.yaml` in the Helm chart:

```yaml
environment:
  DB_USER: {{ .Values.secrets.username }}
  DB_PASSWORD: {{ .Values.secrets.password }}
```

Deploy the Helm chart:

```bash
helm install secret-helm-app ./helm-chart
```

#### Steps to Verify

1. Ensure the Secret is mounted correctly:
   ```bash
   kubectl describe pod -l app.kubernetes.io/name=secret-helm-app
   ```
2. Verify the application is running:
   ```bash
   kubectl get pods
   ```

</details>




## Notes and Tips
1. **Deployment Strategies:**
   - Blue/green deployments require two separate environments. Use `Service` objects to switch traffic between versions.
   - Canary deployments can be implemented using `nginx-ingress` or `service mesh` solutions like Istio.

2. **Rolling Updates:**
   - Use `kubectl rollout status` to monitor updates.
   - Always test updates in a non-production environment before applying them.

3. **Helm Tips:**
   - Use `helm lint` to validate your Helm charts before deployment.
   - Always version-control your `values.yaml` file.
---

## Resources
1. [Kubernetes Official Documentation](https://kubernetes.io/docs/)
2. [Helm Official Documentation](https://helm.sh/docs/)
3. [Kustomize Official Documentation](https://kustomize.io/)
4. [CKAD Exam Curriculum](https://github.com/cncf/curriculum)
5. [Kubernetes Deployment Best Practices](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

---
