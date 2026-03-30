 

### Core Concept
Orchestration automates container deployment, scaling, networking, and lifecycle management. Kubernetes dominates, but Docker Swarm and Nomad exist for simpler needs. Orchestration handles scheduling, service discovery, load balancing, rolling updates, self-healing, and secret management across clusters.

### Key Terminologies
- **Control Plane**: Master components (API server, scheduler, controller manager)
- **Worker Node**: Container runtime + kubelet
- **Pod**: Smallest deployable unit (1+ containers sharing namespace)
- **Service**: Stable endpoint for pods with load balancing
- **Ingress**: HTTP/HTTPS routing to services
- **Operator**: Custom controller encoding operational knowledge

### How it Works
Kubernetes reconciliation loop:
1. User declares desired state (Deployment, Service, Ingress)
2. API server stores in etcd
3. Controller manager watches for changes
4. Scheduler assigns pods to nodes
5. Kubelet on node creates containers
6. Controllers maintain desired state (restart failed pods, scale replicas)

Deployment strategies:
- **RollingUpdate**: Replace pods gradually (default)
- **Recreate**: Kill all, recreate (downtime)
- **Blue-Green**: Two environments, switch traffic
- **Canary**: Gradual traffic shift to new version
- **A/B Testing**: Route based on headers

### CLI Commands/Syntax
```yaml
# Kubernetes Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:latest
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: 100m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        capabilities:
          drop: ["ALL"]

---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
```

```bash
# Deployment commands
kubectl apply -f deployment.yaml
kubectl rollout status deployment/myapp
kubectl rollout history deployment/myapp
kubectl rollout undo deployment/myapp --to-revision=2
kubectl set image deployment/myapp app=myapp:v2
kubectl scale deployment/myapp --replicas=5

# Docker Swarm (simpler alternative)
docker swarm init
docker service create --name myapp --replicas 3 -p 8080:80 myapp:latest
docker service update --image myapp:v2 --update-parallelism 2 --update-delay 10s myapp
```

### Best Practices
- **Set resource requests/limits** for all containers (critical for scheduling)
- **Implement PodDisruptionBudgets** for voluntary disruptions (node drains)
- **Use HorizontalPodAutoscaler** based on CPU/memory/custom metrics
- **Configure PodTopologySpreadConstraints** for zone resilience
- **Implement network policies** for zero-trust communication
- **Use GitOps** (ArgoCD, Flux) for declarative deployments
- **Enable cluster autoscaling** (CA) and node autoscaling
- **Regularly upgrade Kubernetes version** (within N-2)
- **Backup etcd regularly** for cluster recovery

---