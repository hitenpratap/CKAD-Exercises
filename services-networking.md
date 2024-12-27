# CKAD Practice: Services and Networking

## Overview
The Services and Networking topic in the CKAD (Certified Kubernetes Application Developer) exam focuses on understanding and managing connectivity within Kubernetes clusters. This includes creating and managing network policies, exposing applications using services, and configuring ingress rules for external access.

## Topics Covered
- Demonstrate basic understanding of NetworkPolicies
- Provide and troubleshoot access to applications via services
- Use Ingress rules to expose applications

## Practice Questions

### 1. Create a Default Deny-All NetworkPolicy

**Scenario**:
- Create a NetworkPolicy named `deny-all` that denies all ingress and egress traffic for Pods in the `default` namespace.

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
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f deny-all.yaml
   ```
2. Verify the policy:
   ```bash
   kubectl describe networkpolicy deny-all
   ```

</details>

---

### 2. Allow Ingress Traffic from Specific Pods

**Scenario**:
- Create a NetworkPolicy named `allow-frontend` to allow traffic to Pods labeled `app=backend` only from Pods labeled `role=frontend`.

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f allow-frontend.yaml
   ```
2. Test connectivity from a frontend Pod to a backend Pod.

</details>

---

### 3. Restrict Egress Traffic to Specific IPs

**Scenario**:
- Create a NetworkPolicy named `restrict-egress` to restrict Pods labeled `app=web` to only communicate with the IP range `192.168.1.0/24`.

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-egress
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: web
  egress:
  - to:
    - ipBlock:
        cidr: 192.168.1.0/24
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f restrict-egress.yaml
   ```
2. Verify egress rules:
   ```bash
   kubectl describe networkpolicy restrict-egress
   ```

</details>

---

### 4. Allow Ingress Traffic on Specific Ports

**Scenario**:
- Create a NetworkPolicy named `allow-port` to allow traffic to Pods labeled `app=database` only on port 3306 (MySQL).

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-port
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: database
  ingress:
  - from:
    - podSelector: {}
      ports:
      - protocol: TCP
        port: 3306
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f allow-port.yaml
   ```
2. Test connectivity to the database Pod on port 3306.

</details>

---

### 5. Combine Ingress and Egress Rules

**Scenario**:
- Create a NetworkPolicy named `combined-policy` to allow ingress traffic from `role=frontend` Pods and egress traffic to `192.168.2.0/24`.

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: combined-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
  egress:
  - to:
    - ipBlock:
        cidr: 192.168.2.0/24
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f combined-policy.yaml
   ```
2. Test ingress and egress connectivity for the backend Pods.

</details>

---

### 6. Test Default Allow Behavior

**Scenario**:
- Deploy Pods in a namespace without any NetworkPolicy.
- Verify that all traffic is allowed by default.

<details>
<summary>Details</summary>

#### Steps to Test

1. Deploy two Pods:
   ```bash
   kubectl run pod1 --image=busybox --command -- sleep 3600
   kubectl run pod2 --image=busybox --command -- sleep 3600
   ```
2. Test connectivity:
   ```bash
   kubectl exec pod1 -- ping pod2
   ```

</details>

---

### 7. Deny All Egress Traffic

**Scenario**:
- Create a NetworkPolicy named `deny-egress` that blocks all egress traffic for Pods in the `prod` namespace.

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-egress
  namespace: prod
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress: []
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f deny-egress.yaml
   ```
2. Test egress connectivity from any Pod in the `prod` namespace.

</details>

---

### 8. Allow Egress to a Specific Namespace

**Scenario**:
- Create a NetworkPolicy named `namespace-egress` that allows Pods in `dev` to communicate with Pods in `prod`.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: namespace-egress
  namespace: dev
spec:
  podSelector:
    matchLabels:
      app: frontend
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          environment: prod
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f namespace-egress.yaml
   ```
2. Test connectivity between `dev` and `prod` namespaces.

</details>

---

### 9. Isolate a Namespace with a Default Deny Policy

**Scenario**:
- Apply a default deny-all NetworkPolicy to isolate all Pods in the `staging` namespace.

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: isolate-namespace
  namespace: staging
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f isolate-namespace.yaml
   ```
2. Verify that traffic is denied.

</details>

---

### 10. Allow DNS Traffic for Specific Pods

**Scenario**:
- Allow Pods labeled `app=web` to access DNS servers on port 53.

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: web
  egress:
  - to:
    - ipBlock:
        cidr: 8.8.8.8/32
      ports:
      - protocol: UDP
        port: 53
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f allow-dns.yaml
   ```
2. Verify DNS access for the web Pods.

</details>

---

### 11. Expose a Deployment via a ClusterIP Service

**Scenario**:
- Create a Deployment named `my-app` with 3 replicas.
- Expose it internally using a ClusterIP Service named `my-app-service`.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: busybox
        command: ["/bin/sh", "-c", "while true; do echo hello; sleep 5; done"]
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
  type: ClusterIP
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f my-app-service.yaml
   ```
2. Test the Service:
   ```bash
   kubectl exec -it <pod-name> -- curl my-app-service
   ```

</details>

---

### 12. Expose a Deployment Externally Using a NodePort Service

**Scenario**:
- Create a Deployment named `external-app`.
- Expose it externally using a NodePort Service on port 30007.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: external-app
  labels:
    app: external-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: external-app
  template:
    metadata:
      labels:
        app: external-app
    spec:
      containers:
      - name: app
        image: busybox
        command: ["/bin/sh", "-c", "while true; do echo hello; sleep 5; done"]
---
apiVersion: v1
kind: Service
metadata:
  name: external-app-service
spec:
  selector:
    app: external-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 30007
  type: NodePort
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f external-app-service.yaml
   ```
2. Test the Service externally:
   ```bash
   curl <node-ip>:30007
   ```

</details>

---

### 13. Configure a LoadBalancer Service for a Deployment

**Scenario**:
- Deploy an application named `loadbalanced-app`.
- Expose it using a LoadBalancer Service.

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: loadbalanced-app
  labels:
    app: loadbalanced-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: loadbalanced-app
  template:
    metadata:
      labels:
        app: loadbalanced-app
    spec:
      containers:
      - name: app
        image: nginx
---
apiVersion: v1
kind: Service
metadata:
  name: loadbalanced-service
spec:
  selector:
    app: loadbalanced-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f loadbalanced-service.yaml
   ```
2. Verify the external IP of the LoadBalancer:
   ```bash
   kubectl get svc loadbalanced-service
   ```
3. Test access to the application using the external IP.

</details>

---

### 14. Troubleshoot a Service Not Forwarding Traffic

**Scenario**:
- A Service named `troubleshoot-service` is not forwarding traffic to the backend Pods.
- Investigate and resolve the issue.

<details>
<summary>Details</summary>

#### Steps to Troubleshoot

1. Verify Service configuration:
   ```bash
   kubectl describe service troubleshoot-service
   ```
2. Check the endpoint mappings:
   ```bash
   kubectl get endpoints troubleshoot-service
   ```
3. Ensure the backend Pods are running and labeled correctly:
   ```bash
   kubectl get pods -l app=<label>
   ```
4. Correct any misconfigurations and test again.

</details>

---

### 15. Use ExternalName Service to Alias External Resources

**Scenario**:
- Create an ExternalName Service named `external-service` to alias `example.com`.

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  type: ExternalName
  externalName: example.com
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f external-service.yaml
   ```
2. Test the alias:
   ```bash
   kubectl exec -it <pod-name> -- curl external-service
   ```

</details>

---

### 16. Configure a Headless Service for StatefulSets

**Scenario**:
- Create a StatefulSet named `stateful-app`.
- Use a headless Service to provide direct access to each Pod.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

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
      - name: app
        image: busybox
        command: ["/bin/sh", "-c", "while true; do echo hello; sleep 5; done"]
---
apiVersion: v1
kind: Service
metadata:
  name: stateful-service
spec:
  clusterIP: None
  selector:
    app: stateful
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f stateful-service.yaml
   ```
2. Verify the DNS entries for the headless Service:
   ```bash
   kubectl exec -it <pod-name> -- nslookup stateful-service
   ```

</details>

---

### 17. Configure Session Affinity in a Service

**Scenario**:
- Create a Service named `session-service` with session affinity enabled.

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Service
metadata:
  name: session-service
spec:
  selector:
    app: web-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  sessionAffinity: ClientIP
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f session-service.yaml
   ```
2. Test session affinity:
   ```bash
   kubectl exec -it <pod-name> -- curl session-service
   ```

</details>

---

### 18. Implement a Service with Health Checks

**Scenario**:
- Create a Deployment named `health-app`.
- Expose it using a Service with a health check configured.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: health-app
  labels:
    app: health-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: health-app
  template:
    metadata:
      labels:
        app: health-app
    spec:
      containers:
      - name: app
        image: nginx
        readinessProbe:
          httpGet:
            path: /
            port: 80
---
apiVersion: v1
kind: Service
metadata:
  name: health-service
spec:
  selector:
    app: health-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

#### Steps to Apply

1. Save the YAML file and apply it:
   ```bash
   kubectl apply -f health-service.yaml
   ```
2. Verify Pod readiness:
   ```bash
   kubectl get pods -l app=health-app
   ```

</details>

---

### 19. Expose a Single Application Using Ingress

**Scenario**:
- Deploy an application named `simple-app`.
- Expose it using an Ingress resource with the hostname `simple-app.local`.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: simple-app
  labels:
    app: simple-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: simple-app
  template:
    metadata:
      labels:
        app: simple-app
    spec:
      containers:
      - name: app
        image: nginx
---
apiVersion: v1
kind: Service
metadata:
  name: simple-app-service
spec:
  selector:
    app: simple-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: simple-app-ingress
spec:
  rules:
  - host: simple-app.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: simple-app-service
            port:
              number: 80
```

#### Steps to Apply and Verify

1. Save and apply the YAML configuration:
   ```bash
   kubectl apply -f simple-app-ingress.yaml
   ```
2. Test the Ingress:
   ```bash
   curl -H "Host: simple-app.local" <ingress-controller-ip>
   ```

</details>

---

### 20. Configure TLS for an Ingress Resource

**Scenario**:
- Secure an application exposed through Ingress with TLS.
- Use a Secret named `tls-secret` for the certificate and key.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
  namespace: default
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-cert>
  tls.key: <base64-encoded-key>
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-app-ingress
spec:
  tls:
  - hosts:
    - tls-app.local
    secretName: tls-secret
  rules:
  - host: tls-app.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: tls-app-service
            port:
              number: 80
```

#### Steps to Apply and Verify

1. Save and apply the YAML configuration:
   ```bash
   kubectl apply -f tls-app-ingress.yaml
   ```
2. Test HTTPS access:
   ```bash
   curl -k https://tls-app.local --resolve tls-app.local:<ingress-controller-ip>
   ```

</details>

---

### 21. Expose Multiple Applications Using Host-Based Rules

**Scenario**:
- Expose two applications (`app1` and `app2`) through a single Ingress.
- Use host-based rules to route traffic.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-host-ingress
spec:
  rules:
  - host: app1.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
  - host: app2.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
```

#### Steps to Apply and Verify

1. Save and apply the YAML configuration:
   ```bash
   kubectl apply -f multi-host-ingress.yaml
   ```
2. Test each application:
   ```bash
   curl -H "Host: app1.local" <ingress-controller-ip>
   curl -H "Host: app2.local" <ingress-controller-ip>
   ```

</details>

---

### 22. Implement Path-Based Routing in Ingress

**Scenario**:
- Route `/app1` to `app1-service` and `/app2` to `app2-service` using an Ingress resource.

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-routing-ingress
spec:
  rules:
  - host: path-app.local
    http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
```

#### Steps to Apply and Verify

1. Save and apply the YAML configuration:
   ```bash
   kubectl apply -f path-routing-ingress.yaml
   ```
2. Test each path:
   ```bash
   curl -H "Host: path-app.local" <ingress-controller-ip>/app1
   curl -H "Host: path-app.local" <ingress-controller-ip>/app2
   ```

</details>

---

### 23. Debug an Ingress Resource Not Routing Traffic

**Scenario**:
- An Ingress resource named `debug-ingress` is not routing traffic to the backend.
- Debug and resolve the issue.

<details>
<summary>Details</summary>


#### Steps to Debug

1. Verify Ingress configuration:
   ```bash
   kubectl describe ingress debug-ingress
   ```
2. Check Service endpoints:
   ```bash
   kubectl get endpoints <service-name>
   ```
3. Ensure backend Pods are running and labeled correctly:
   ```bash
   kubectl get pods -l <label-selector>
   ```
4. Correct any misconfigurations and test again.

</details>

---

### 24. Use Default Backend for Unmatched Routes

**Scenario**:
- Configure an Ingress resource with a default backend to handle unmatched requests.

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: default-backend-ingress
spec:
  defaultBackend:
    service:
      name: default-service
      port:
        number: 80
  rules:
  - host: example.local
    http:
      paths:
      - path: /app
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
```

#### Steps to Apply and Verify

1. Save and apply the YAML configuration:
   ```bash
   kubectl apply -f default-backend-ingress.yaml
   ```
2. Test default backend routing:
   ```bash
   curl -H "Host: example.local" <ingress-controller-ip>/unmatched-path
   ```

</details>

---

### 25. Configure a Rewrite Rule in an Ingress

**Scenario**:
- Use annotations to rewrite `/old-path` to `/new-path` for an application.

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rewrite-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /new-path
spec:
  rules:
  - host: rewrite-app.local
    http:
      paths:
      - path: /old-path
        pathType: Prefix
        backend:
          service:
            name: rewrite-service
            port:
              number: 80
```

#### Steps to Apply and Verify

1. Save and apply the YAML configuration:
   ```bash
   kubectl apply -f rewrite-ingress.yaml
   ```
2. Test the rewrite rule:
   ```bash
   curl -H "Host: rewrite-app.local" <ingress-controller-ip>/old-path
   ```

</details>

---

### 26. Configure CORS Using Ingress Annotations

**Scenario**:
- Add CORS headers to an Ingress resource to allow cross-origin requests.

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cors-ingress
  annotations:
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "*"
    nginx.ingress.kubernetes.io/cors-allow-methods: "PUT, GET, POST, OPTIONS"
    nginx.ingress.kubernetes.io/cors-allow-headers: "DNT,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization"
spec:
  rules:
  - host: cors-app.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: cors-service
            port:
              number: 80
```

#### Steps to Apply and Verify

1. Save and apply the YAML configuration:
   ```bash
   kubectl apply -f cors-ingress.yaml
   ```
2. Test CORS configuration:
   ```bash
   curl -H "Origin: http://example.com" -H "Access-Control-Request-Method: GET" -i http://cors-app.local
   ```

</details>

---

### 27. Implement Multi-Tenant Hosting with Ingress

**Scenario**:
- Host two applications (`tenant1-app` and `tenant2-app`) on the same Ingress with different subdomains: `tenant1.example.com` and `tenant2.example.com`.

<details>
<summary>Details</summary>

#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-tenant-ingress
spec:
  rules:
  - host: tenant1.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: tenant1-service
            port:
              number: 80
  - host: tenant2.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: tenant2-service
            port:
              number: 80
```

#### Steps to Apply and Verify

1. Save and apply the YAML configuration:
   ```bash
   kubectl apply -f multi-tenant-ingress.yaml
   ```
2. Test each subdomain:
   ```bash
   curl -H "Host: tenant1.example.com" <ingress-controller-ip>
   curl -H "Host: tenant2.example.com" <ingress-controller-ip>
   ```

</details>

---

### 28. Add Rate Limiting to an Ingress

**Scenario**:
- Configure rate limiting for an application using Ingress annotations.

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rate-limited-ingress
  annotations:
    nginx.ingress.kubernetes.io/limit-rps: "5"
    nginx.ingress.kubernetes.io/limit-burst-multiplier: "2"
spec:
  rules:
  - host: rate-limited-app.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: rate-limited-service
            port:
              number: 80
```

#### Steps to Apply and Verify

1. Save and apply the YAML configuration:
   ```bash
   kubectl apply -f rate-limited-ingress.yaml
   ```
2. Test rate limiting:
   ```bash
   ab -n 100 -c 10 -H "Host: rate-limited-app.local" http://<ingress-controller-ip>/
   ```

</details>

---

### 29. Use Ingress Annotations for Canary Deployments

**Scenario**:
- Split traffic between two versions (`v1` and `v2`) of an application using Ingress annotations.

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: canary-ingress
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "20"
spec:
  rules:
  - host: canary-app.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: stable-service
            port:
              number: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: canary-ingress-v2
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "20"
spec:
  rules:
  - host: canary-app.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: canary-service
            port:
              number: 80
```

#### Steps to Apply and Verify

1. Save and apply the YAML configuration:
   ```bash
   kubectl apply -f canary-ingress.yaml
   ```
2. Test traffic distribution:
   ```bash
   curl -H "Host: canary-app.local" <ingress-controller-ip>
   ```

</details>

---

### 30. Redirect HTTP to HTTPS in Ingress

**Scenario**:
- Use Ingress annotations to enforce HTTPS by redirecting HTTP traffic.

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: https-redirect-ingress
  annotations:
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - https-app.local
    secretName: tls-secret
  rules:
  - host: https-app.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: https-service
            port:
              number: 80
```

#### Steps to Apply and Verify

1. Save and apply the YAML configuration:
   ```bash
   kubectl apply -f https-redirect-ingress.yaml
   ```
2. Test HTTPS redirection:
   ```bash
   curl http://https-app.local --resolve https-app.local:<ingress-controller-ip>
   ```

</details>

---

### 31. Use Wildcard Hostnames in Ingress Rules

**Scenario**:
- Configure a single Ingress to handle requests for multiple subdomains using a wildcard hostname.

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wildcard-ingress
spec:
  rules:
  - host: "*.example.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: wildcard-service
            port:
              number: 80
```

#### Steps to Apply and Verify

1. Save and apply the YAML configuration:
   ```bash
   kubectl apply -f wildcard-ingress.yaml
   ```
2. Test with subdomains:
   ```bash
   curl -H "Host: app1.example.com" <ingress-controller-ip>
   curl -H "Host: app2.example.com" <ingress-controller-ip>
   ```

</details>

---

### 32. Configure Backend Timeouts in Ingress

**Scenario**:
- Set timeout for backend responses in an Ingress resource.

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: timeout-ingress
  annotations:
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "60"
spec:
  rules:
  - host: timeout-app.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: timeout-service
            port:
              number: 80
```

#### Steps to Apply and Verify

1. Save and apply the YAML configuration:
   ```bash
   kubectl apply -f timeout-ingress.yaml
   ```
2. Test timeout behavior:
   ```bash
   curl -H "Host: timeout-app.local" <ingress-controller-ip>
   ```

</details>

---

### 33. Configure Custom Error Pages for Ingress

**Scenario**:
- Use Ingress annotations to display custom error pages for HTTP 404 errors.

<details>
<summary>Details</summary>


#### Declarative YAML Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: error-page-ingress
  annotations:
    nginx.ingress.kubernetes.io/custom-http-errors: "404"
    nginx.ingress.kubernetes.io/server-snippet: |
      error_page 404 /custom_404.html;
      location = /custom_404.html {
        internal;
        root /usr/share/nginx/html;
      }
spec:
  rules:
  - host: error-app.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: error-service
            port:
              number: 80
```

#### Steps to Apply and Verify

1. Save and apply the YAML configuration:
   ```bash
   kubectl apply -f error-page-ingress.yaml
   ```
2. Test the custom error page:
   ```bash
   curl -H "Host: error-app.local" <ingress-controller-ip>/nonexistent-path
   ```

</details>

---

## Notes and Tips
- **NetworkPolicies**:
  - NetworkPolicies are designed to control pod-level network communication. They are namespace-scoped resources.
  - Understand the default behavior (allow all traffic if no policy exists) and how to create rules to allow/deny ingress and egress traffic.
  
- **Services**:
  - Understand the types of Kubernetes Services (ClusterIP, NodePort, LoadBalancer, and ExternalName) and their use cases.
  - Be familiar with `kubectl` commands for troubleshooting services, such as checking endpoints with `kubectl describe`.

- **Ingress**:
  - Ingress rules provide HTTP and HTTPS routes to services.
  - Know how to configure `Ingress` objects with different paths, hostnames, and TLS configurations.

## Resources
- [Kubernetes Official Documentation on NetworkPolicies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Kubernetes Official Documentation on Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Kubernetes Official Documentation on Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Kubernetes Networking Tutorials](https://kubernetes.io/docs/tutorials/services/)
