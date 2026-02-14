* Cluster information     :`kubectl cluster-info`
  
* List nodes       :`kubectl get nodes`

* List deployments ;```kubectl get deploy```

* List replicaSets :`kubectl get rs`

* List pods        :`kubectl get pods`

* Detailed list pods     :`kubectl get pods -o wide`

* Inspect pods     :`kubectl get pods -o wide`

* Apply file       :`kubectl apply -f <file>.yml`

* To check file    :`kubectl apply -f <file>.yml --dry-run=server`

* Exec into the Pod:`kubectl exec -it <pod> -- /bin/sh`

* Delete deployment:`kubectl delete deployment <deployment-name>`

* Delete pod       :`kubectl delete pod <pod-name>`

* Scalling         :`kubectl scale deployment <name> --replicas=5 `

* Rollout History  :`kubectl rollout history deployment/<name>`

* Rollback         :`kubectl rollout undo deployment <name>`
