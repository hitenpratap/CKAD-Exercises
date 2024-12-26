# CKAD Exam: Application Build and Design Questions

## Overview
This document contains practice questions for the **Certified Kubernetes Application Developer (CKAD)** exam, focusing on the **Application Build and Design** section.

---

## Topics Covered
1. Designing and implementing applications in Kubernetes
2. Understanding multi-container pod design patterns
3. Configuring application configurations using ConfigMaps and Secrets
4. Managing application lifecycle and health checks
5. Creating Kubernetes resources using imperative and declarative methods
6. Exploring init containers and sidecar patterns
7. Understanding resource limits and requests
8. Utilizing labels, selectors, and annotations for application management
9. Managing Jobs and CronJobs
10. Configuring PersistentVolumeClaims for storage
11. Understanding Services and NetworkPolicies
12. Implementing SecurityContexts for Pods and containers

---

## Practice Questions

## Question 1
Create a deployment named `web-app` with 5 replicas using the `nginx:latest` image. Ensure the deployment includes a label `tier=frontend`.

<details>
<summary>Solution</summary>

### Imperative Approach:
```bash
kubectl create deployment web-app --image=nginx:latest --replicas=5 --dry-run=client -o yaml | kubectl label --local -f - tier=frontend | kubectl apply -f -
```

### Declarative Approach:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 5
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```
Apply the configuration:
```bash
kubectl apply -f deployment.yaml
```
</details>

---

## Question 2
Expose the `web-app` deployment as a ClusterIP service on port 80.

<details>
<summary>Solution</summary>

### Imperative Approach:
```bash
kubectl expose deployment web-app --type=ClusterIP --port=80
```

### Declarative Approach:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-service
spec:
  selector:
    app: web-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```
Apply the configuration:
```bash
kubectl apply -f service.yaml
```
</details>

---

## Question 3
Configure a multi-container pod named `log-processor` where the primary container runs `nginx:alpine` and a sidecar container runs `busybox`, tailing logs from `/var/log/nginx/access.log`.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: log-processor
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/nginx
  - name: log-tailer
    image: busybox
    args: ["/bin/sh", "-c", "tail -f /var/log/nginx/access.log"]
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/nginx
  volumes:
  - name: log-volume
    emptyDir: {}
```
Apply the configuration:
```bash
kubectl apply -f pod.yaml
```
</details>

---

## Question 4
Create a ConfigMap named `app-config` with the key-value pair `APP_MODE=production` and use it to configure a deployment named `config-app` with an environment variable.

<details>
<summary>Solution</summary>

### Imperative Approach:
```bash
kubectl create configmap app-config --from-literal=APP_MODE=production
kubectl create deployment config-app --image=nginx --dry-run=client -o yaml | kubectl apply -f -
kubectl set env deployment/config-app --from=configmap/app-config
```

### Declarative Approach:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
  data:
    APP_MODE: production
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: config-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: config-app
  template:
    metadata:
      labels:
        app: config-app
    spec:
      containers:
      - name: nginx
        image: nginx
        env:
        - name: APP_MODE
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_MODE
```
Apply the configuration:
```bash
kubectl apply -f configmap.yaml
data.yaml
```
</details>

---

## Question 5
Create a Secret named `db-credentials` with keys `username=admin` and `password=secure123`, and mount it into a deployment named `db-app` at `/etc/secrets`.

<details>
<summary>Solution</summary>

### Imperative Approach:
```bash
kubectl create secret generic db-credentials --from-literal=username=admin --from-literal=password=secure123
kubectl create deployment db-app --image=mysql --dry-run=client -o yaml | kubectl apply -f -
kubectl set env deployment/db-app --from=secret/db-credentials
```

### Declarative Approach:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
  namespace: default
  data:
    username: YWRtaW4=
    password: c2VjdXJlMTIz
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db-app
  template:
    metadata:
      labels:
        app: db-app
    spec:
      containers:
      - name: mysql
        image: mysql
        volumeMounts:
        - name: secret-volume
          mountPath: /etc/secrets
      volumes:
      - name: secret-volume
        secret:
          secretName: db-credentials
```
Apply the configuration:
```bash
kubectl apply -f secret.yaml
kubectl apply -f deployment.yaml
```
</details>

---

## Question 6
Configure a readiness probe for a deployment named `probe-app` that checks the HTTP endpoint `/health` on port 8080.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: probe-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: probe-app
  template:
    metadata:
      labels:
        app: probe-app
    spec:
      containers:
      - name: app
        image: custom-app:1.0
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
```
Apply the configuration:
```bash
kubectl apply -f readiness-probe.yaml
```
</details>

## Question 7
Create a deployment named `multi-app` with two containers: one running `nginx:latest` and the other running `busybox` to periodically echo "Hello Kubernetes".

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: multi-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: multi-app
  template:
    metadata:
      labels:
        app: multi-app
    spec:
      containers:
      - name: nginx
        image: nginx:latest
      - name: busybox
        image: busybox
        args: ["/bin/sh", "-c", "while true; do echo 'Hello Kubernetes'; sleep 5; done"]
```
Apply the configuration:
```bash
kubectl apply -f multi-container.yaml
```
</details>

---

## Question 8
Create a CronJob named `backup-job` that runs every day at midnight and backs up data using the `busybox` image.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-job
spec:
  schedule: "0 0 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: busybox
            args: ["/bin/sh", "-c", "echo Backup completed"]
          restartPolicy: OnFailure
```
Apply the configuration:
```bash
kubectl apply -f cronjob.yaml
```
</details>

---

## Question 9
Create a PersistentVolumeClaim (PVC) named `storage-claim` with a storage request of 1Gi and use it in a pod running `nginx:latest` to mount the volume at `/usr/share/nginx/html`.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: storage-claim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pvc
spec:
  containers:
  - name: nginx
    image: nginx:latest
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: storage-claim
```
Apply the configuration:
```bash
kubectl apply -f pvc-pod.yaml
```
</details>

---

## Question 10
Secure a pod named `secure-app` with a SecurityContext that runs the container as a non-root user with UID 1000 and mounts the root filesystem as read-only.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
spec:
  containers:
  - name: secure-container
    image: nginx:latest
    securityContext:
      runAsUser: 1000
      readOnlyRootFilesystem: true
```
Apply the configuration:
```bash
kubectl apply -f secure-pod.yaml
```
</details>

---

## Question 11
Create a NetworkPolicy that allows traffic to the pod labeled `app: backend` only from pods labeled `app: frontend`.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```
Apply the configuration:
```bash
kubectl apply -f networkpolicy.yaml
```
</details>

---

## Question 12
Configure resource requests and limits for a pod running `nginx:latest` to request 100m CPU and 200Mi memory, with a limit of 500m CPU and 512Mi memory.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-limited-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    resources:
      requests:
        memory: "200Mi"
        cpu: "100m"
      limits:
        memory: "512Mi"
        cpu: "500m"
```
Apply the configuration:
```bash
kubectl apply -f resource-limits.yaml
```
</details>

---

## Question 13
Update a deployment named `api-server` to use the `nginx:1.21` image without downtime.

<details>
<summary>Solution</summary>

### Imperative Approach:
```bash
kubectl set image deployment/api-server nginx=nginx:1.21
```

### Declarative Approach:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-server
  template:
    metadata:
      labels:
        app: api-server
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```
Apply the configuration:
```bash
kubectl apply -f api-server-deployment.yaml
```
</details>

---

## Question 14
Create a Job named `data-loader` that completes after successfully running a single instance of `busybox` to print "Data loaded".

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-loader
spec:
  template:
    spec:
      containers:
      - name: loader
        image: busybox
        args: ["/bin/sh", "-c", "echo Data loaded"]
      restartPolicy: OnFailure
```
Apply the configuration:
```bash
kubectl apply -f job.yaml
```
</details>

---

## Question 15
Deploy a `Redis` StatefulSet with 3 replicas, ensuring each pod gets a unique hostname and persistent storage.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
spec:
  replicas: 3
  selector:
    matchLabels:
      app: redis
  serviceName: "redis"
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:6.2
        volumeMounts:
        - name: redis-data
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: redis-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```
Apply the configuration:
```bash
kubectl apply -f redis-statefulset.yaml
```
</details>

## Question 16
Implement a deployment named `web-cache` with `memcached:1.6.9` that uses a readiness probe checking the TCP socket on port 11211.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-cache
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-cache
  template:
    metadata:
      labels:
        app: web-cache
    spec:
      containers:
      - name: memcached
        image: memcached:1.6.9
        readinessProbe:
          tcpSocket:
            port: 11211
          initialDelaySeconds: 5
          periodSeconds: 10
```
Apply the configuration:
```bash
kubectl apply -f web-cache-deployment.yaml
```
</details>

---

## Question 17
Create a pod named `app-with-config` that uses a ConfigMap named `app-config` to set the environment variable `APP_MODE` to `production`.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
  data:
    APP_MODE: production
---
apiVersion: v1
kind: Pod
metadata:
  name: app-with-config
spec:
  containers:
  - name: app
    image: busybox
    env:
    - name: APP_MODE
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_MODE
```
Apply the configuration:
```bash
kubectl apply -f app-with-config.yaml
```
</details>

---

## Question 18
Deploy a StatefulSet named `mysql-db` with 2 replicas using the `mysql:5.7` image. Configure each pod to use a PersistentVolumeClaim with 10Gi of storage.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-db
spec:
  serviceName: "mysql"
  replicas: 2
  selector:
    matchLabels:
      app: mysql-db
  template:
    metadata:
      labels:
        app: mysql-db
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "rootpassword"
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```
Apply the configuration:
```bash
kubectl apply -f mysql-statefulset.yaml
```
</details>

---

## Question 19
Set up a pod named `busybox-init` with an init container that runs a setup script before the main container starts.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-init
spec:
  initContainers:
  - name: setup
    image: busybox
    command: ["/bin/sh", "-c", "echo Initializing... && sleep 5"]
  containers:
  - name: main
    image: busybox
    command: ["/bin/sh", "-c", "echo Running main container"]
```
Apply the configuration:
```bash
kubectl apply -f busybox-init.yaml
```
</details>

---

## Question 20
Create a Deployment named `web-tier` using `nginx:1.21`, and configure a Horizontal Pod Autoscaler (HPA) to scale between 2 and 10 replicas based on CPU utilization above 70%.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-tier
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-tier
  template:
    metadata:
      labels:
        app: web-tier
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        resources:
          requests:
            cpu: 200m
          limits:
            cpu: 500m
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-tier-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-tier
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```
Apply the configuration:
```bash
kubectl apply -f web-tier-deployment.yaml
kubectl apply -f web-tier-hpa.yaml
```
</details>

## Question 21
Create a deployment named `file-server` using the `nginx:latest` image. Add a liveness probe that checks the HTTP endpoint `/healthz` on port 80 every 10 seconds, starting after an initial delay of 5 seconds.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: file-server
spec:
  replicas: 2
  selector:
    matchLabels:
      app: file-server
  template:
    metadata:
      labels:
        app: file-server
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        livenessProbe:
          httpGet:
            path: /healthz
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
```
Apply the configuration:
```bash
kubectl apply -f file-server-deployment.yaml
```
</details>

---

## Question 22
Deploy a pod named `storage-pod` that uses a PVC named `shared-storage` to mount a volume at `/data`. The PVC should request 5Gi of storage.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shared-storage
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: storage-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    volumeMounts:
    - mountPath: /data
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: shared-storage
```
Apply the configuration:
```bash
kubectl apply -f pvc-pod.yaml
```
</details>

---

## Question 23
Create a pod named `frontend` that communicates with a backend service named `backend-service` over port 8080. Use environment variables to pass the backend service hostname and port.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: frontend
    image: nginx:latest
    env:
    - name: BACKEND_HOST
      value: backend-service
    - name: BACKEND_PORT
      value: "8080"
```
Apply the configuration:
```bash
kubectl apply -f frontend-pod.yaml
```
</details>

---

## Question 24
Create a NetworkPolicy named `allow-frontend` to allow ingress traffic to pods labeled `app: backend` only from pods labeled `app: frontend` on port 8080.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
  policyTypes:
  - Ingress
```
Apply the configuration:
```bash
kubectl apply -f allow-frontend-policy.yaml
```
</details>

---

## Question 25
Deploy a CronJob named `log-cleanup` that runs every day at midnight and deletes log files from `/var/log/app` using the `busybox` image.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: log-cleanup
spec:
  schedule: "0 0 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cleanup
            image: busybox
            args: ["/bin/sh", "-c", "rm -rf /var/log/app/*"]
          restartPolicy: OnFailure
```
Apply the configuration:
```bash
kubectl apply -f log-cleanup-cronjob.yaml
```
</details>

---

## Question 26
Create a deployment named `api-server` with the `python:3.8` image and configure it to use resource requests of 200m CPU and 256Mi memory, with limits of 1 CPU and 512Mi memory.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api-server
  template:
    metadata:
      labels:
        app: api-server
    spec:
      containers:
      - name: python-app
        image: python:3.8
        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
          limits:
            memory: "512Mi"
            cpu: "1"
```
Apply the configuration:
```bash
kubectl apply -f api-server-deployment.yaml
```
</details>

---

## Question 27
Set up a pod named `restricted-pod` with a SecurityContext that prevents privilege escalation and runs the container as a specific user with UID 1001.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: restricted-pod
spec:
  containers:
  - name: app
    image: nginx:latest
    securityContext:
      runAsUser: 1001
      allowPrivilegeEscalation: false
```
Apply the configuration:
```bash
kubectl apply -f restricted-pod.yaml
```
</details>

---

## Question 28
Create a Job named `data-fetcher` that runs a single instance of `busybox` to fetch data from an external API and store it in a mounted volume.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-fetcher
spec:
  template:
    spec:
      containers:
      - name: fetcher
        image: busybox
        args: ["/bin/sh", "-c", "wget -O /data/output.txt http://example.com/data"]
        volumeMounts:
        - name: output-volume
          mountPath: /data
      volumes:
      - name: output-volume
        emptyDir: {}
      restartPolicy: OnFailure
```
Apply the configuration:
```bash
kubectl apply -f data-fetcher-job.yaml
```
</details>

## Question 29
Deploy a StatefulSet named `zookeeper-cluster` with 3 replicas using the `zookeeper:3.6` image. Ensure each pod has its own persistent storage of 5Gi and a unique hostname.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: zookeeper-cluster
spec:
  serviceName: "zookeeper"
  replicas: 3
  selector:
    matchLabels:
      app: zookeeper
  template:
    metadata:
      labels:
        app: zookeeper
    spec:
      containers:
      - name: zookeeper
        image: zookeeper:3.6
        volumeMounts:
        - name: data
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 5Gi
```
Apply the configuration:
```bash
kubectl apply -f zookeeper-statefulset.yaml
```
</details>

---

## Question 30
Create a pod named `audit-logger` with a sidecar container that tails the application logs from `/var/log/app.log` in the main container.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: audit-logger
spec:
  containers:
  - name: app-container
    image: busybox
    command: ["/bin/sh", "-c", "while true; do echo \"App log entry\" >> /var/log/app.log; sleep 5; done"]
    volumeMounts:
    - name: log-volume
      mountPath: /var/log
  - name: sidecar-logger
    image: busybox
    command: ["/bin/sh", "-c", "tail -f /var/log/app.log"]
    volumeMounts:
    - name: log-volume
      mountPath: /var/log
  volumes:
  - name: log-volume
    emptyDir: {}
```
Apply the configuration:
```bash
kubectl apply -f audit-logger-pod.yaml
```
</details>

---

## Question 31
Configure a Deployment named `metrics-app` with the `prom/prometheus` image and expose it via a ClusterIP service on port 9090.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metrics-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: metrics-app
  template:
    metadata:
      labels:
        app: metrics-app
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus
        ports:
        - containerPort: 9090
---
apiVersion: v1
kind: Service
metadata:
  name: metrics-service
spec:
  selector:
    app: metrics-app
  ports:
  - protocol: TCP
    port: 9090
    targetPort: 9090
  type: ClusterIP
```
Apply the configuration:
```bash
kubectl apply -f metrics-app.yaml
kubectl apply -f metrics-service.yaml
```
</details>

---

## Question 32
Create a Job named `db-migrator` that runs a one-time database migration script using the `mysql:5.7` image and environment variables for database credentials.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migrator
spec:
  template:
    spec:
      containers:
      - name: migrator
        image: mysql:5.7
        command: ["/bin/sh", "-c", "echo Running migration && exit 0"]
        env:
        - name: DB_USER
          value: root
        - name: DB_PASSWORD
          value: password123
        - name: DB_HOST
          value: db-service
      restartPolicy: OnFailure
```
Apply the configuration:
```bash
kubectl apply -f db-migrator-job.yaml
```
</details>

---

## Question 33
Create a pod named `secure-web` that uses a SecurityContext to enforce a read-only root filesystem and drops all Linux capabilities.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-web
spec:
  containers:
  - name: nginx
    image: nginx:latest
    securityContext:
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
```
Apply the configuration:
```bash
kubectl apply -f secure-web-pod.yaml
```
</details>

---

## Question 34
Deploy a StatefulSet named `redis-cluster` with 3 replicas, each running `redis:6.2`, and configure it with a headless service.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-headless
spec:
  clusterIP: None
  selector:
    app: redis
  ports:
  - port: 6379
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-cluster
spec:
  serviceName: "redis-headless"
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:6.2
        ports:
        - containerPort: 6379
```
Apply the configuration:
```bash
kubectl apply -f redis-headless.yaml
kubectl apply -f redis-cluster.yaml
```
</details>

## Question 35
Create a Deployment named `custom-nginx` with the `nginx:latest` image, configure a custom port 8080, and expose it with a LoadBalancer service.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: custom-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: custom-nginx
  template:
    metadata:
      labels:
        app: custom-nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: custom-nginx-service
spec:
  selector:
    app: custom-nginx
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
  type: LoadBalancer
```
Apply the configuration:
```bash
kubectl apply -f custom-nginx-deployment.yaml
```
</details>

---

## Question 36
Create a CronJob named `data-cleanup` that runs every 6 hours and clears old temporary files using the `busybox` image.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: data-cleanup
spec:
  schedule: "0 */6 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cleaner
            image: busybox
            args: ["/bin/sh", "-c", "rm -rf /tmp/*"]
          restartPolicy: OnFailure
```
Apply the configuration:
```bash
kubectl apply -f data-cleanup-cronjob.yaml
```
</details>

---

## Question 37
Create a Deployment named `flask-app` using `python:3.9` and configure a ConfigMap to provide the environment variable `FLASK_ENV=development`.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: flask-config
  namespace: default
  data:
    FLASK_ENV: development
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
      - name: flask-container
        image: python:3.9
        env:
        - name: FLASK_ENV
          valueFrom:
            configMapKeyRef:
              name: flask-config
              key: FLASK_ENV
```
Apply the configuration:
```bash
kubectl apply -f flask-app-deployment.yaml
```
</details>

---

## Question 38
Create a Pod named `nginx-pod` that mounts a Secret named `tls-secret` at `/etc/nginx/ssl` to provide SSL certificates.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
  namespace: default
  type: kubernetes.io/tls
data:
  tls.crt: BASE64_ENCODED_CERTIFICATE
  tls.key: BASE64_ENCODED_KEY
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    volumeMounts:
    - name: tls-volume
      mountPath: /etc/nginx/ssl
  volumes:
  - name: tls-volume
    secret:
      secretName: tls-secret
```
Apply the configuration:
```bash
kubectl apply -f nginx-pod.yaml
```
</details>

---

## Question 39
Set up a Horizontal Pod Autoscaler (HPA) for a Deployment named `worker-app` that scales between 1 and 5 replicas based on 80% memory utilization.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: worker-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: worker-app
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```
Apply the configuration:
```bash
kubectl apply -f worker-app-hpa.yaml
```
</details>

---

## Question 40
Create a StatefulSet named `cassandra-cluster` with 3 replicas running the `cassandra:latest` image. Use a headless service for inter-pod communication.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: cassandra-headless
spec:
  clusterIP: None
  selector:
    app: cassandra
  ports:
  - port: 9042
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: cassandra-cluster
spec:
  serviceName: "cassandra-headless"
  replicas: 3
  selector:
    matchLabels:
      app: cassandra
  template:
    metadata:
      labels:
        app: cassandra
    spec:
      containers:
      - name: cassandra
        image: cassandra:latest
        ports:
        - containerPort: 9042
```
Apply the configuration:
```bash
kubectl apply -f cassandra-headless.yaml
kubectl apply -f cassandra-cluster.yaml
```
</details>

## Question 41
Deploy a ReplicaSet named `web-replicaset` with 3 replicas running the `nginx:1.19` image and ensure pods have the label `tier=frontend`.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: web-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
```
Apply the configuration:
```bash
kubectl apply -f web-replicaset.yaml
```
</details>

---

## Question 42
Create a pod named `db-pod` that uses an emptyDir volume to store temporary data at `/data`. The container should run `mysql:5.7`.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: db-pod
spec:
  containers:
  - name: mysql
    image: mysql:5.7
    volumeMounts:
    - mountPath: /data
      name: temp-storage
  volumes:
  - name: temp-storage
    emptyDir: {}
```
Apply the configuration:
```bash
kubectl apply -f db-pod.yaml
```
</details>

---

## Question 43
Deploy a Job named `batch-processor` that processes a batch file by running a command in the `busybox` image.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-processor
spec:
  template:
    spec:
      containers:
      - name: processor
        image: busybox
        args: ["/bin/sh", "-c", "echo Processing batch file"]
      restartPolicy: OnFailure
```
Apply the configuration:
```bash
kubectl apply -f batch-processor-job.yaml
```
</details>

---

## Question 44
Create a Deployment named `web-cache` with the `memcached:1.6` image. Add a liveness probe to check the TCP socket on port 11211 every 10 seconds, with an initial delay of 5 seconds.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-cache
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-cache
  template:
    metadata:
      labels:
        app: web-cache
    spec:
      containers:
      - name: memcached
        image: memcached:1.6
        livenessProbe:
          tcpSocket:
            port: 11211
          initialDelaySeconds: 5
          periodSeconds: 10
```
Apply the configuration:
```bash
kubectl apply -f web-cache-deployment.yaml
```
</details>

---

## Question 45
Create a NetworkPolicy named `deny-all` that denies all ingress traffic to pods labeled `app: secure-app`.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector:
    matchLabels:
      app: secure-app
  policyTypes:
  - Ingress
```
Apply the configuration:
```bash
kubectl apply -f deny-all-policy.yaml
```
</details>

---

## Question 46
Deploy a StatefulSet named `etcd-cluster` with 3 replicas running the `quay.io/coreos/etcd:v3.4.15` image and configure persistent storage for each pod.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: etcd-cluster
spec:
  serviceName: "etcd-service"
  replicas: 3
  selector:
    matchLabels:
      app: etcd
  template:
    metadata:
      labels:
        app: etcd
    spec:
      containers:
      - name: etcd
        image: quay.io/coreos/etcd:v3.4.15
        volumeMounts:
        - name: etcd-data
          mountPath: /var/lib/etcd
  volumeClaimTemplates:
  - metadata:
      name: etcd-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 5Gi
```
Apply the configuration:
```bash
kubectl apply -f etcd-cluster.yaml
```
</details>

---

## Question 47
Create a ConfigMap named `app-settings` with multiple keys (`key1=value1`, `key2=value2`) and mount it as environment variables in a pod named `config-pod`.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-settings
  namespace: default
  data:
    key1: value1
    key2: value2
---
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
spec:
  containers:
  - name: app
    image: busybox
    envFrom:
    - configMapRef:
        name: app-settings
```
Apply the configuration:
```bash
kubectl apply -f app-settings-configmap.yaml
```
</details>

---

## Question 48
Deploy a pod named `frontend` that uses a `NodePort` service to expose it on port 30080.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: nginx
    image: nginx:latest
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```
Apply the configuration:
```bash
kubectl apply -f frontend-service.yaml
```
</details>

---

## Question 49
Create a PersistentVolumeClaim (PVC) named `shared-pvc` requesting 1Gi of storage and use it in a pod named `shared-pod` at `/mnt/shared`.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shared-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: shared-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    volumeMounts:
    - name: shared-volume
      mountPath: /mnt/shared
  volumes:
  - name: shared-volume
    persistentVolumeClaim:
      claimName: shared-pvc
```
Apply the configuration:
```bash
kubectl apply -f shared-pvc-pod.yaml
```
</details>

---

## Question 50
Set up a pod named `audit-pod` that logs audit information to `/var/log/audit` and ensure the logs are accessible using a sidecar container running `busybox`.

<details>
<summary>Solution</summary>

### Declarative Approach:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: audit-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/audit
  - name: sidecar
    image: busybox
    command: ["/bin/sh", "-c", "tail -f /var/log/audit"]
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/audit
  volumes:
  - name: log-volume
    emptyDir: {}
```
Apply the configuration:
```bash
kubectl apply -f audit-pod.yaml
```
</details>

---

## Notes and Tips
1. Always use `kubectl explain <resource>` for resource reference.
2. Use `kubectl dry-run` to validate your YAML before applying changes.
3. For faster debugging, use imperative commands and export YAML for declarative configurations.
4. Understand the difference between ConfigMaps and Secrets.
5. Practice creating and managing multi-container Pods (init containers and sidecars).
6. Explore SecurityContexts, Jobs, CronJobs, PersistentVolumeClaims, and NetworkPolicies.

---

## Resources
- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
- [CKAD Exam Curriculum](https://github.com/cncf/curriculum/blob/main/CKAD_Curriculum_v1.31.pdf)
