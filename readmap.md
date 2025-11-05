# **🎯 REALISTIC 9-MONTH ROADMAP (2 hrs/day)**
**Philosophy: "Slow is smooth, smooth is fast"**

---

## **TIMELINE OVERVIEW**
- **Total Duration**: 9 months (36 weeks)
- **Weekly Commitment**: 14 hours (2 hrs × 7 days)
- **Focus**: Quality understanding > Speed
- **Method**: Build → Break → Fix → Explain → Repeat 5x

---

# **📅 MONTH 1-2: KUBERNETES FOUNDATIONS**
## **Goal: Master Pod → Service → Deployment progression**

### **Week 1-2: Single Pod & Namespace (First Deployment)**

**Files to Write**:
```yaml
# Day 1-3: 00-sock-shop-ns.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: sock-shop
```

**Practice (5 iterations)**:
```bash
# Iteration 1: Create and understand
kubectl create -f 00-sock-shop-ns.yaml
kubectl get ns sock-shop -o yaml
kubectl describe ns sock-shop

# Iteration 2: Delete and recreate from memory
kubectl delete ns sock-shop
# Write from scratch without looking

# Iteration 3: Add labels and annotations
metadata:
  name: sock-shop
  labels:
    env: learning
    project: sock-shop
  annotations:
    description: "My learning namespace"

# Iteration 4: Create using kubectl
kubectl create ns sock-shop-v2 --dry-run=client -o yaml

# Iteration 5: Explain to Claude
"Why do we need namespaces? What happens without them?"
```

**Then: First Pod**
```yaml
# Day 4-7: simple-pod.yaml (NOT from Sock Shop yet)
apiVersion: v1
kind: Pod
metadata:
  name: nginx-learning
  namespace: sock-shop
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
```

**Deep Dive Questions for Claude**:
1. What is a Pod vs Container?
2. Why metadata matters?
3. What happens when pod crashes?
4. How does K8s know which node to place pod?

**Practice (5 iterations)**:
```bash
# Iteration 1: Deploy and explore
kubectl apply -f simple-pod.yaml
kubectl get pods -n sock-shop
kubectl describe pod nginx-learning -n sock-shop
kubectl logs nginx-learning -n sock-shop
kubectl exec -it nginx-learning -n sock-shop -- /bin/bash

# Iteration 2: Break it
- Wrong image name → ImagePullBackOff
- Wrong namespace → Error
- Remove ports → Still works? Why?

# Iteration 3: Add resource limits
resources:
  requests:
    memory: "64Mi"
    cpu: "100m"
  limits:
    memory: "128Mi"
    cpu: "200m"

# Iteration 4: Add health checks
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 10

# Iteration 5: Write from memory, explain each field
```

---

### **Week 3-4: Service Discovery**

**Files to Write**:
```yaml
# simple-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: sock-shop
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

**Learning Path**:
```bash
# Day 1-2: ClusterIP (internal only)
kubectl apply -f simple-service.yaml
kubectl get svc -n sock-shop
kubectl describe svc nginx-service -n sock-shop

# Test from another pod
kubectl run test-pod --image=busybox -n sock-shop --rm -it -- sh
wget -O- nginx-service:80

# Day 3-4: NodePort (external access)
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080

# Day 5-7: LoadBalancer
spec:
  type: LoadBalancer
```

**Deep Questions**:
1. How does Service find Pods? (selector matching)
2. What is ClusterIP vs NodePort vs LoadBalancer?
3. What happens if selector doesn't match any pod?
4. What is kube-proxy doing?

**5 Iterations**: Each service type, break selectors, explain DNS resolution

---

### **Week 5-6: Deployments (Proper Way)**

**Now Learn from Sock Shop**:
```yaml
# Day 1-3: 09-front-end-dep.yaml (FIRST SOCK SHOP FILE)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: front-end
  namespace: sock-shop
spec:
  replicas: 1
  selector:
    matchLabels:
      name: front-end
  template:
    metadata:
      labels:
        name: front-end
    spec:
      containers:
      - name: front-end
        image: weaveworksdemos/front-end:0.3.12
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 8079
```

**Understanding Journey**:
```bash
# Iteration 1: Basic deployment
kubectl apply -f 09-front-end-dep.yaml
kubectl get deploy -n sock-shop
kubectl get rs -n sock-shop  # ReplicaSet created
kubectl get pods -n sock-shop

# Iteration 2: Scale and observe
kubectl scale deploy front-end --replicas=3 -n sock-shop
# Watch pods being created
kubectl get pods -w -n sock-shop

# Iteration 3: Update image (rollout)
kubectl set image deploy/front-end front-end=weaveworksdemos/front-end:0.3.13 -n sock-shop
kubectl rollout status deploy/front-end -n sock-shop
kubectl rollout history deploy/front-end -n sock-shop

# Iteration 4: Break and rollback
# Bad image tag
kubectl set image deploy/front-end front-end=weaveworksdemos/front-end:bad -n sock-shop
kubectl rollout undo deploy/front-end -n sock-shop

# Iteration 5: Add strategy
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

**Deep Questions**:
1. Why Deployment > Pod?
2. What is ReplicaSet doing?
3. How does rolling update work?
4. When to use Recreate vs RollingUpdate?

---

### **Week 7-8: Complete First Microservice**

**Deploy Front-end Completely**:
```yaml
# Day 1-4: 09-front-end-dep.yaml (already done)
# Day 5-7: 10-front-end-svc.yaml

apiVersion: v1
kind: Service
metadata:
  name: front-end
  namespace: sock-shop
  labels:
    name: front-end
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8079
    nodePort: 30001
  selector:
    name: front-end
```

**Test End-to-End**:
```bash
# Access front-end
kubectl get svc front-end -n sock-shop
# Visit http://<node-ip>:30001

# Check logs
kubectl logs -l name=front-end -n sock-shop --tail=50 -f

# Debug connection
kubectl port-forward svc/front-end 8080:80 -n sock-shop
curl localhost:8080
```

**Milestone**: You can access Sock Shop UI (even if backend fails)!

**5 Iterations**:
1. Deploy and access
2. Change service type (ClusterIP → NodePort → LoadBalancer)
3. Break selector (no traffic)
4. Add multiple replicas, see load balancing
5. Write both files from memory, explain connection

---

# **📅 MONTH 3-4: STATELESS SERVICES + CONFIG**
## **Goal: Deploy all stateless microservices**

### **Week 9-10: Catalogue Service (with ConfigMaps)**

**Files**:
```yaml
# 05-catalogue-dep.yaml
# 06-catalogue-svc.yaml
```

**New Concept: Environment Variables**
```yaml
spec:
  containers:
  - name: catalogue
    env:
    - name: ZIPKIN
      value: zipkin.jaeger.svc.cluster.local
```

**Practice**:
```bash
# Iteration 1: Basic deploy
kubectl apply -f 05-catalogue-dep.yaml
kubectl apply -f 06-catalogue-svc.yaml

# Iteration 2: Create ConfigMap
kubectl create configmap catalogue-config \
  --from-literal=LOG_LEVEL=debug \
  -n sock-shop

# Modify deployment
env:
- name: LOG_LEVEL
  valueFrom:
    configMapKeyRef:
      name: catalogue-config
      key: LOG_LEVEL

# Iteration 3: ConfigMap from file
# Create catalogue-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: catalogue-config
data:
  database.url: "catalogue-db:3306"
  log.level: "info"

# Iteration 4: Volume mount ConfigMap
volumeMounts:
- name: config
  mountPath: /config
volumes:
- name: config
  configMap:
    name: catalogue-config

# Iteration 5: Hot reload config changes
kubectl edit cm catalogue-config -n sock-shop
# Watch pod pick up changes
```

**Deep Questions**:
1. ConfigMap vs environment variables vs command args?
2. When does pod see ConfigMap changes?
3. What if ConfigMap doesn't exist?

---

### **Week 11-12: Orders, Payment, Shipping Services**

**Files (in order)**:
```yaml
# 11-orders-dep.yaml + 12-orders-svc.yaml
# 15-payment-dep.yaml + 16-payment-svc.yaml
# 23-shipping-dep.yaml + 24-shipping-svc.yaml
```

**Pattern Recognition**:
```bash
# All follow same pattern:
1. Deployment with image
2. Service with selector
3. Internal communication (ClusterIP)

# Practice: Deploy in sequence
kubectl apply -f 11-orders-dep.yaml
kubectl apply -f 12-orders-svc.yaml
# Test orders talks to payment
kubectl exec -it <orders-pod> -- wget payment:8080/health
```

**Iteration Pattern for Each Service**:
1. Deploy as-is
2. Add resource limits
3. Add health checks
4. Increase replicas to 3
5. Test inter-service communication

---

### **Week 13-14: Carts + User Services**

**Files**:
```yaml
# 01-carts-dep.yaml + 02-carts-svc.yaml
# 25-user-dep.yaml + 26-user-svc.yaml
```

**New Concept: Secrets**
```yaml
# Create secret for user service
apiVersion: v1
kind: Secret
metadata:
  name: user-db-password
  namespace: sock-shop
type: Opaque
data:
  password: cGFzc3dvcmQxMjM=  # base64 encoded
```

**Practice**:
```bash
# Iteration 1: Create secret imperatively
kubectl create secret generic db-password \
  --from-literal=password=mypassword \
  -n sock-shop

# Iteration 2: Use in deployment
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-password
      key: password

# Iteration 3: Secret as volume
volumeMounts:
- name: db-creds
  mountPath: /secrets
  readOnly: true
volumes:
- name: db-creds
  secret:
    secretName: db-password

# Iteration 4: Seal secrets (optional advanced)
# Iteration 5: Explain secret vs ConfigMap
```

---

# **📅 MONTH 5-6: STATEFUL SERVICES (DATABASES)**
## **Goal: Master StatefulSets, PV/PVC, Headless Services**

### **Week 15-17: MongoDB (Carts DB)**

**Files**:
```yaml
# 03-carts-db-dep.yaml (convert to StatefulSet)
# 04-carts-db-svc.yaml (headless service)
```

**Original (Deployment)**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: carts-db
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: carts-db
        image: mongo:4.4
        ports:
        - containerPort: 27017
```

**Transform to StatefulSet**:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: carts-db
spec:
  serviceName: carts-db
  replicas: 1
  selector:
    matchLabels:
      name: carts-db
  template:
    metadata:
      labels:
        name: carts-db
    spec:
      containers:
      - name: carts-db
        image: mongo:4.4
        ports:
        - containerPort: 27017
        volumeMounts:
        - name: data
          mountPath: /data/db
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

**Headless Service**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: carts-db
spec:
  clusterIP: None  # Headless!
  selector:
    name: carts-db
  ports:
  - port: 27017
```

**Deep Learning (2 weeks)**:
```bash
# Week 1: Understand StatefulSet
# Iteration 1: Deploy and observe
kubectl apply -f carts-db-statefulset.yaml
kubectl get statefulset -n sock-shop
kubectl get pvc -n sock-shop  # PVC created automatically!

# Iteration 2: Scale and observe ordering
kubectl scale statefulset carts-db --replicas=3 -n sock-shop
# Watch: carts-db-0, then carts-db-1, then carts-db-2 (ordered!)

# Iteration 3: Delete pod, see same PVC reattached
kubectl delete pod carts-db-0 -n sock-shop
# New pod gets same PVC!

# Week 2: Storage
# Iteration 4: Check PV/PVC
kubectl get pv
kubectl get pvc -n sock-shop
kubectl describe pvc data-carts-db-0 -n sock-shop

# Iteration 5: Data persistence test
# Write data to MongoDB
kubectl exec -it carts-db-0 -n sock-shop -- mongo
> use testdb
> db.test.insert({name: "persistence-test"})
> exit

# Delete pod, check data still there
kubectl delete pod carts-db-0 -n sock-shop
kubectl exec -it carts-db-0 -n sock-shop -- mongo
> use testdb
> db.test.find()  # Data persists!
```

**Deep Questions**:
1. Why StatefulSet vs Deployment for databases?
2. What is headless service (clusterIP: None)?
3. How does volumeClaimTemplate work?
4. What happens to PVC when StatefulSet deleted?

---

### **Week 18-19: MySQL (Catalogue DB)**

**Files**:
```yaml
# 07-catalogue-db-dep.yaml → StatefulSet
# 08-catalogue-db-svc.yaml
```

**Practice Same Pattern**:
```bash
# Convert to StatefulSet
# Add PVC for /var/lib/mysql
# Add secret for root password
# Test data persistence
```

**Additional Learning**:
```yaml
# Init containers (database initialization)
initContainers:
- name: init-db
  image: mysql:5.7
  command: ['sh', '-c', 'echo "Initializing DB..."; sleep 10']
```

---

### **Week 20-21: MySQL (Orders DB, User DB) + RabbitMQ**

**Files**:
```yaml
# 13-orders-db-dep.yaml
# 14-orders-db-svc.yaml
# 27-user-db-dep.yaml
# 28-user-db-svc.yaml
# 19-rabbitmq-dep.yaml + 20-rabbitmq-svc.yaml
```

**Practice**:
- Convert all to StatefulSets
- Add PVC to each
- Test message queue with RabbitMQ
- Understand different storage needs

---

### **Week 22: Session DB (In-Memory)**

**Files**:
```yaml
# 21-session-db-dep.yaml
# 22-session-db-svc.yaml
```

**Learn**: Redis doesn't always need persistence (session store)

---

# **📅 MONTH 7: COMPLETE SYSTEM + NETWORKING**
## **Goal: Full app working, understand pod communication**

### **Week 23-24: Deploy Complete Sock Shop**

**All 28 Files Together**:
```bash
# In order:
kubectl apply -f 00-sock-shop-ns.yaml

# Databases first (they take time)
kubectl apply -f 03-carts-db-*.yaml
kubectl apply -f 07-catalogue-db-*.yaml
kubectl apply -f 13-orders-db-*.yaml
kubectl apply -f 21-session-db-*.yaml
kubectl apply -f 27-user-db-*.yaml
kubectl apply -f 19-rabbitmq-*.yaml

# Wait for DBs ready
kubectl wait --for=condition=ready pod -l name=carts-db -n sock-shop --timeout=300s

# Services
kubectl apply -f 01-carts-*.yaml
kubectl apply -f 05-catalogue-*.yaml
kubectl apply -f 11-orders-*.yaml
kubectl apply -f 15-payment-*.yaml
kubectl apply -f 23-shipping-*.yaml
kubectl apply -f 25-user-*.yaml

# Frontend last
kubectl apply -f 09-front-end-*.yaml
```

**Testing**:
```bash
# Check everything running
kubectl get all -n sock-shop

# Test frontend
kubectl port-forward svc/front-end 8080:80 -n sock-shop

# Create account, place order (full flow!)
```

**5 Iterations**:
1. Deploy all, access UI, test full user journey
2. Delete one service, see impact (dependencies!)
3. Scale all services to 3 replicas
4. Simulate DB failure, watch cascade
5. Explain architecture to Claude (draw diagram)

---

### **Week 25-26: Networking Deep Dive**

**Network Policies**:
```yaml
# Default deny all
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: sock-shop
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

# Allow frontend → catalogue
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: catalogue-policy
spec:
  podSelector:
    matchLabels:
      name: catalogue
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: front-end
    ports:
    - protocol: TCP
      port: 80
```

**Practice**:
```bash
# Iteration 1: Apply default deny, app breaks
kubectl apply -f default-deny.yaml

# Iteration 2: Add policies one by one
# Observe which services need to talk

# Iteration 3: DNS policies
# Iteration 4: External egress (DB → internet?)
# Iteration 5: Explain network flow
```

---

# **📅 MONTH 8: PRODUCTION READINESS**
## **Goal: Observability, scaling, troubleshooting**

### **Week 27-28: Resource Management**

**Add to All Deployments**:
```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

**LimitRange & ResourceQuota**:
```yaml
# LimitRange (default limits)
apiVersion: v1
kind: LimitRange
metadata:
  name: sock-shop-limits
  namespace: sock-shop
spec:
  limits:
  - default:
      cpu: 500m
      memory: 512Mi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    type: Container

# ResourceQuota (namespace total)
apiVersion: v1
kind: ResourceQuota
metadata:
  name: sock-shop-quota
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 10Gi
    limits.cpu: "20"
    limits.memory: 20Gi
```

**Practice**: Set resource requests/limits on all pods, observe scheduling

---

### **Week 29-30: Health Checks & Autoscaling**

**Add Probes**:
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```

**HPA**:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: front-end-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: front-end
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

**Load Test**:
```bash
# Install metrics-server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Generate load
kubectl run -it --rm load-generator --image=busybox -- /bin/sh
while true; do wget -q -O- http://front-end.sock-shop/; done

# Watch HPA
kubectl get hpa -n sock-shop -w
```

---

### **Week 31-32: Logging & Monitoring**

**Deploy Monitoring Stack**:
```bash
# Prometheus
kubectl apply -f deploy/kubernetes/manifests-monitoring/

# Access Grafana
kubectl port-forward svc/grafana -n sock-shop 3000:80
```

**Learn**:
- Prometheus queries (PromQL)
- Grafana dashboards
- Alert rules
- Log aggregation with Fluentd (optional)

---

# **📅 MONTH 9: ADVANCED PATTERNS**
## **Goal: Helm, GitOps basics, multi-env**

### **Week 33-34: Helm Chart**

**Create Chart**:
```bash
helm create sock-shop

# Structure:
sock-shop/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
```

**Parameterize**:
```yaml
# values.yaml
replicaCount: 2
image:
  repository: weaveworksdemos/front-end
  tag: "0.3.12"

# templates/deployment.yaml
replicas: {{ .Values.replicaCount }}
image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
```

**Practice**:
```bash
# Iteration 1: Basic chart
helm install sock-shop ./sock-shop -n sock-shop

# Iteration 2: Override values
helm install sock-shop ./sock-shop \
  --set replicaCount=5 \
  -n sock-shop

# Iteration 3: Multiple environments
# values-dev.yaml
# values-prod.yaml

# Iteration 4: Helm upgrade
helm upgrade sock-shop ./sock-shop -n sock-shop

# Iteration 5: Create umbrella chart
```

---

### **Week 35-36: GitOps Foundation (ArgoCD Basics)**

**Install ArgoCD**:
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

**Create Git Repo**:
```
my-sock-shop-gitops/
├── base/
│   └── (all 28 yamls)
├── dev/
│   └── kustomization.yaml
├── prod/
    └── kustomization.yaml
```

**ArgoCD Application**:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: sock-shop
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/yourusername/sock-shop-gitops
    targetRevision: HEAD
    path: base
  destination:
    server: https://kubernetes.default.svc
    namespace: sock-shop
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

**Practice**: Push changes to Git, watch ArgoCD sync automatically

---

# **📊 FINAL MILESTONE CHECKLIST**

After 9 months, you should:

✅ **Core Kubernetes**
- [ ] Write any manifest from memory
- [ ] Explain Pod → ReplicaSet → Deployment hierarchy
- [ ] Understand Service types and when to use each
- [ ] Configure StatefulSets with persistence
- [ ] Manage ConfigMaps and Secrets
- [ ] Set resource requests/limits/quotas

✅ **Networking**
- [ ] Explain ClusterIP, DNS, kube-proxy
- [ ] Write Network Policies
- [ ] Debug service connectivity issues

✅ **Storage**
- [ ] Create PV/PVC
- [ ] Understand StorageClass
- [ ] Choose StatefulSet vs Deployment

✅ **Observability**
- [ ] Add health checks to all apps
- [ ] Query Prometheus metrics
- [ ] Create Grafana dashboards
- [ ] Debug with logs and events

✅ **Advanced**
- [ ] Build Helm charts
- [ ] Setup GitOps with ArgoCD
- [ ] Configure HPA
- [ ] Multi-environment deployments

✅ **Troubleshooting**
- [ ] Fix any CrashLoopBackOff in <5 min
- [ ] Debug ImagePullBackOff, OOMKilled
- [ ] Solve service discovery issues
- [ ] Recover from node failures

---

# **📝 DAILY ROUTINE (2 hours)**

**Structure Each Day**:
```
Hour 1: Hands-on Practice
- Write YAML
- Apply to cluster
- Test functionality
- Break something
- Fix it

Hour 2: Understanding & Documentation
- Ask Claude "Why does X work?"
- Draw architecture diagrams
- Write explanations in your own words
- Update personal runbook
```

**Example Day**:
```
9:00-10:00: Deploy catalogue-db StatefulSet
           - Write YAML from scratch
           - Apply and verify PVC created
           - Test data persistence

10:00-11:00: Deep dive with Claude
            "Explain StatefulSet vs Deployment"
            "Why does volumeClaimTemplate work?"
            "What is a headless service?"
            Document answers
```

---

# **🎯 SKILL LEVEL TRACKING**

| Month | Level | Milestone |
|-------|-------|-----------|
| 1-2 | 3→4 | Pod, Service, Deployment mastered |
| 3-4 | 4→5 | All stateless services running |
| 5-6 | 5→6 | StatefulSets & persistence clear |
| 7 | 6→6.5 | Full app deployed, networking understood |
| 8 | 6.5→7 | Production patterns implemented |
| 9 | 7→7.5 | Helm, GitOps basics |

**After 9 months**: Solid Level 7, ready for advanced topics (Istio, multi-cluster, operators)

---

# **💡 SUCCESS TIPS**

1. **Never skip iterations**: 5 times per concept is non-negotiable
2. **Explain to Claude daily**: Teaching is learning twice
3. **Keep a learning journal**: Write what you learned each day
4. **Draw diagrams**: Visualize pod-to-pod communication
5. **Break things intentionally**: Delete pods, corrupt configs
6. **Compare before/after**: Use `kubectl diff` before applying
7. **Read official docs**: When stuck, K8s docs are gold
8. **Join communities**: Ask questions in CNCF Slack

---

**Ready to start Week 1? Ask me for detailed exercises on namespace + first pod!** 🚀