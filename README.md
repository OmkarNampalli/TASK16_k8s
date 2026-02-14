# TASK16_k8s
This task is all about the DEPLOYMENT in kubernetes
```
Deployment → manages → ReplicaSet → manages → Pods → run → Containers
```
---

# Deployment
**Deployment is a controler object of kubernetes which manages set of identical pods and ensures they run reliably in your cluster**
## 🔹 What a Deployment Does
A Deployment:
1. Creates Pods (via a ReplicaSet)
2. Keeps the desired number of Pods running
3. Handles rolling updates
4. Supports rollbacks to previous versions
5. Manages scaling (up/down)

---

## Replica Sets
A **ReplicaSet** ensures that a specified number of identical Pods are running at all times.
In simple words:
> Keep N copies of this Pod always running.
### 🔹 Why Do We Need ReplicaSet?
Problem without ReplicaSet:
- Pod crashes → not recreated
- Node dies → Pod lost
- No scaling

ReplicaSet solves this by:
- Monitoring Pods
- Recreating failed Pods
- Maintaining desired count

#### 🔥 Real Example
If you define:
```
replicas: 3
```
ReplicaSet ensures:
- Always 3 Pods running
- If 1 crashes → new one created
- If you scale to 5 → 2 more created
- If you scale down to 2 → 3 removed

### 🔥 What Happens Internally?
ReplicaSet Controller continuously watches:
- Current Pods count
- Desired replicas

If current < desired → create Pod
If current > desired → delete Pod

This is done via Kubernetes control loop.

Deployment internally creates ReplicaSets.

When you update image:

- Deployment creates NEW ReplicaSet
- Gradually scales down old RS
- Scales up new RS

That’s **rolling update**.

> ReplicaSet is a Kubernetes controller that ensures a specified number of pod replicas are running at all times. It maintains desired state by creating or deleting pods as necessary. Deployments use ReplicaSets internally to manage scaling and rolling updates.


### 🔥 Advanced Insight
ReplicaSet works using:

Kubernetes **reconciliation loop**.

Control plane continuously checks:

Desired state (in etcd)
vs
Current state (running pods)

If mismatch → corrective action.

---

## Rolling Update (rollout)
Rollout & Rollback are handled by Deployment, but internally they work using ReplicaSets.

Let's say you have an **update for your app** and you have to **deploy it** in kubernetes.
But you **don't want any downtime** for your app.
To solve the above problems we use **rollling update (rollout)**.
Basically we provide latest or updated container to **deployment object**.
Then Deployment:
- Creates NEW ReplicaSet v2
- Gradually increases replicas of v2
- Gradually decreases replicas of v1
- Ensures availability during transition

In one line
> Rollout = Updating application without downtime.

### 🔥 Rolling Update Strategy
Default strategy:
```
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
```

#### 🔹 maxSurge

How many extra pods can be created during update.

Example:
replicas = 3
maxSurge = 1

Kubernetes can temporarily create 4 pods.

#### 🔹 maxUnavailable

How many pods can be unavailable during update.

Example:
maxUnavailable = 1

Only 1 pod can be down at a time.

### 🔥 How To Trigger Rollout
#### Method 1: Update Image
```
kubectl set image deployment/nginx-deployment nginx=nginx:1.22
```

This triggers rollout.

#### Method 2: Edit Deployment
```
kubectl edit deployment nginx-deployment
```

Change image manually.

#### Method 3: Apply Updated YAML
```
kubectl apply -f deployment.yaml
```

If spec changes → rollout starts.

#### 🔥 Check Rollout Status
```
kubectl rollout status deployment nginx-deployment
```
#### 🔥 See Rollout History
```
kubectl rollout history deployment nginx-deployment
```

#### 🔥 Pause & Resume Rollout

Pause:
```
kubectl rollout pause deployment nginx-deployment
```
Resume:
```` Bash
kubectl rollout resume deployment nginx-deployment
````
Used in controlled releases.

---

## Rollout


Now assume you update your app using rollout.
And something goes wrong in your application (bugs, app cannot reach DB, etc) but your **previous version** was working perfectly so you want to deploy previous version with zero downtime.

To perform this operation we use **rollback**
Deployment keeps old ReplicaSets.

When you rollback:

1. Deployment scales UP old ReplicaSet
2. Scales DOWN current faulty ReplicaSet
3. Updates revision number

It does NOT rebuild from scratch.

It just switches ReplicaSets.
### 🔥 Rollback Command
Rollback to previous revision:
```
kubectl rollout undo deployment nginx-deployment
```
Rollback to specific revision:
```
kubectl rollout undo deployment nginx-deployment --to-revision=1
```

### 🔥 What Happens Internally During Rollback

Current State:

- ReplicaSet v1 → 0 pods
- ReplicaSet v2 → 3 pods

After rollback:

- ReplicaSet v1 → 3 pods
- ReplicaSet v2 → 0 pods

Same rolling strategy rules apply.

## 🔥 Full Real Flow (CI/CD)
```
Developer pushes code
↓
Docker image built
↓
Image pushed to registry
↓
Deployment YAML updated
↓
kubectl apply
↓
New ReplicaSet created
↓
Rolling update happens
↓
Old ReplicaSet scaled down
↓
Deployment successful

If failure:
kubectl rollout undo
```

Q. “How does Kubernetes perform rolling updates?”

Answer:

Deployment creates a new ReplicaSet with updated pod template and gradually scales it up while scaling down the old ReplicaSet according to maxSurge and maxUnavailable settings, ensuring zero downtime.

---

## LivenessProbe and ReadinessProbe
**🔥 Why Do We Need Probes?**

Imagine:
Your container is running
BUT your app inside is crashed

From Kubernetes point of view:
- Pod = Running
- Container = Running

But your app is broken.

Without probes → Kubernetes won’t know.

Probes allow Kubernetes to:

- Detect broken app
- Restart container
Stop traffic to unhealthy pods
### livenessProbe
#### 🔹 What is Liveness Probe?

Checks:
> “Is the container alive?”

If it fails → Kubernetes restarts the container.

#### 🔥 What Happens Internally?

If liveness probe fails:

Kubelet:
* 1️⃣ Kills container
* 2️⃣ Restarts container
* 3️⃣ Pod stays same

It does NOT recreate Pod.

It restarts container inside Pod.

### 🔥 2️⃣ Readiness Probe
#### 🔹 What is Readiness Probe?

Checks:
> “Is this pod ready to serve traffic?”

If it fails:
* Pod stays running
* BUT traffic is stopped

Service removes pod from endpoints.

#### 🔥 What Happens Internally?

If readiness probe fails:
* 1️⃣ Pod remains running
* 2️⃣ Service removes Pod IP from endpoints
* 3️⃣ No traffic routed

If readiness passes again:

Pod added back to Service endpoints.

| Feature         | Liveness           | Readiness       |
| --------------- | ------------------ | --------------- |
| Purpose         | Restart container  | Stop traffic    |
| If fails        | Container restarts | Traffic removed |
| Affects Service | No                 | Yes             |

---

| Feature                      | Pod             | ReplicaSet                  | Deployment                  |
| ---------------------------- | --------------- | --------------------------- | --------------------------- |
| What it is                   | Smallest unit   | Pod manager                 | ReplicaSet manager          |
| Purpose                      | Runs containers | Maintains desired Pod count | Manages updates & lifecycle |
| Self-healing                 | ❌ No            | ✅ Yes                       | ✅ Yes                       |
| Scaling                      | ❌ Manual        | ✅ Yes                       | ✅ Yes                       |
| Rolling updates              | ❌ No            | ❌ No                        | ✅ Yes                       |
| Rollback support             | ❌ No            | ❌ No                        | ✅ Yes                       |
| Used directly in production? | Rarely          | Rarely                      | ✅ Yes                       |

## 🎯 One-Line Summary

* **Pod** = Runs containers
* **ReplicaSet** = Keeps Pods alive
* **Deployment** = Manages updates and scaling safely
