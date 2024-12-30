# CKA Mock Exam 1
## **Table of Contents**

1. [Q1: List Kubectl Contexts and Display Current Context](#q1)
2. [Q2: Create a Pod Scheduled Only on Controlplane Nodes](#q2)
3. [Q3: Scale Down `o3db-*` Pods in Namespace `project-c13`](#q3)
4. [Q4: Configure Liveness and Readiness Probes](#q4)
5. [Q5: List Pods Sorted by Age and UID](#q5)
6. [Q6: Create PersistentVolume, PersistentVolumeClaim, and Deployment](#q6)
7. [Q7: Kubectl Metrics Commands](#q7)
8. [Q8: Identify Control Plane Components](#q8)
9. [Q9: Manual Scheduling of Pods by Stopping kube-scheduler](#q9)
10. [Q10: Create ServiceAccount, Role, and RoleBinding](#q10)
11. [Q11: Create a DaemonSet](#q11)
12. [Q12: Create a Deployment with Specific Constraints](#q12)
13. [Q13: Create a Multi-Container Pod with Shared Volume](#q13)
14. [Q14: Gather Cluster Information](#q14)
15. [Q15: Monitor and Log Cluster Events](#q15)
16. [Q16: List Namespaced Resources and Identify Crowded Namespace](#q16)
17. [Q17: Retrieve Containerd Information for a Pod](#q17)
18. [Q18: Troubleshoot kubelet on `cluster3-node1`](#q18)
19. [Q19: Manage Secrets in a Pod](#q19)
20. [Q20: Update and Join Node to Cluster](#q20)
21. [Q21: Backup and Expose Static Pod via NodePort Service](#q21)
22. [Q22: Check and Renew kube-apiserver Certificate](#q22)
23. [Q23: Inspect Kubelet Certificates](#q23)
24. [Q24: Create NetworkPolicy to Restrict Pod Communication](#q24)
25. [Q25: Backup and Restore etcd and Verify Cluster State](#q25)

---

### <a name="q1"></a>**Q1: List Kubectl Contexts and Display Current Context**

**Task:**
1. Write all `kubectl` context names into `/opt/course/1/contexts`.
2. Write a command to display the current context into `/opt/course/1/context_default_kubectl.sh` using `kubectl`.
3. Write a second command to display the current context into `/opt/course/1/context_default_no_kubectl.sh` without using `kubectl`.

**Solution:**

1. **List All Context Names and Save to File:**
   
   ```bash
   kubectl config get-contexts -o name | sudo tee /opt/course/1/contexts
   ```
   
   **Explanation:**
   - `kubectl config get-contexts -o name`: Lists all context names.
   - `sudo tee /opt/course/1/contexts`: Writes the output to the specified file with elevated permissions.

2. **Command to Display Current Context Using `kubectl`:**
   
   ```bash
   echo "#!/bin/bash" | sudo tee /opt/course/1/context_default_kubectl.sh
   echo "kubectl config current-context" | sudo tee -a /opt/course/1/context_default_kubectl.sh
   sudo chmod +x /opt/course/1/context_default_kubectl.sh
   ```
   
   **Explanation:**
   - Creates a shell script that echoes the current context using `kubectl`.
   - Makes the script executable.

3. **Command to Display Current Context Without Using `kubectl`:**
   
   ```bash
   echo "#!/bin/bash" | sudo tee /opt/course/1/context_default_no_kubectl.sh
   echo "grep 'current-context' ~/.kube/config | awk '{print \$2}'" | sudo tee -a /opt/course/1/context_default_no_kubectl.sh
   sudo chmod +x /opt/course/1/context_default_no_kubectl.sh
   ```
   
   **Explanation:**
   - Creates a shell script that parses the kubeconfig file directly to find the current context.
   - Uses `grep` and `awk` to extract the context name.
   - Makes the script executable.

**Summary of Commands:**

```bash
# Q1: List all contexts
kubectl config get-contexts -o name | sudo tee /opt/course/1/contexts

# Q1: Create script to display current context using kubectl
echo "#!/bin/bash" | sudo tee /opt/course/1/context_default_kubectl.sh
echo "kubectl config current-context" | sudo tee -a /opt/course/1/context_default_kubectl.sh
sudo chmod +x /opt/course/1/context_default_kubectl.sh

# Q1: Create script to display current context without using kubectl
echo "#!/bin/bash" | sudo tee /opt/course/1/context_default_no_kubectl.sh
echo "grep 'current-context' ~/.kube/config | awk '{print \$2}'" | sudo tee -a /opt/course/1/context_default_no_kubectl.sh
sudo chmod +x /opt/course/1/context_default_no_kubectl.sh
```

---

### <a name="q2"></a>**Q2: Create a Pod Scheduled Only on Controlplane Nodes**

**Task:**
- Use context: `k8s-c1-H`.
- Create a single Pod of image `httpd:2.4.41-alpine` in Namespace `default`.
- Pod Name: `pod1`.
- Container Name: `pod1-container`.
- Pod should only be scheduled on controlplane nodes.
- Do not add new labels to any nodes.

**Solution:**

Since we cannot add new labels to nodes, we need to use existing labels to target controlplane nodes. Typically, controlplane nodes have the label `node-role.kubernetes.io/controlplane=true` or similar. We'll verify the existing labels and use them.

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c1-H
   ```

2. **Identify Controlplane Nodes Labels:**

   First, list all nodes and their labels to identify the label that marks controlplane nodes.

   ```bash
   kubectl get nodes --show-labels
   ```

   **Sample Output:**

   ```
   NAME                  STATUS   ROLES                  AGE   VERSION   LABELS
   cluster1-controlplane1 Ready    controlplane,master    10d   v1.31.0   node-role.kubernetes.io/controlplane=true,...
   cluster1-node1        Ready    <none>                 10d   v1.31.0   ...
   cluster1-node2        Ready    <none>                 10d   v1.31.0   ...
   ```

   Assume the controlplane node has the label `node-role.kubernetes.io/controlplane=true`.

3. **Create Pod Manifest with Node Selector:**

   We'll use a `nodeSelector` to target controlplane nodes based on their existing label.

   ```yaml
   # pod1.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: pod1
     namespace: default
   spec:
     containers:
       - name: pod1-container
         image: httpd:2.4.41-alpine
     nodeSelector:
       node-role.kubernetes.io/controlplane: "true"
   ```

4. **Apply the Pod Manifest:**

   ```bash
   kubectl apply -f pod1.yaml
   ```

5. **Verify Pod Scheduling:**

   ```bash
   kubectl get pod pod1 -n default -o wide
   ```

   **Expected Output:**

   ```
   NAME    READY   STATUS    RESTARTS   AGE   IP           NODE                     NOMINATED NODE   READINESS GATES
   pod1    1/1     Running   0          2m    10.244.1.5   cluster1-controlplane1   <none>           <none>
   ```

**Explanation:**

- The `nodeSelector` ensures that the Pod is only scheduled on nodes labeled as controlplane nodes.
- No new labels are added to any nodes; existing labels are utilized.

**Full Commands and Manifest:**

```bash
# Q2: Set Kubernetes context
kubectl config use-context k8s-c1-H

# Q2: Create Pod manifest
cat <<EOF | sudo tee pod1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: default
spec:
  containers:
    - name: pod1-container
      image: httpd:2.4.41-alpine
  nodeSelector:
    node-role.kubernetes.io/controlplane: "true"
EOF

# Q2: Apply the Pod manifest
kubectl apply -f pod1.yaml

# Q2: Verify Pod is running on controlplane node
kubectl get pod pod1 -n default -o wide
```

---

### <a name="q3"></a>**Q3: Scale Down `o3db-*` Pods in Namespace `project-c13`**

**Task:**
- Use context: `k8s-c1-H`.
- Namespace: `project-c13`.
- Pods named `o3db-*`.
- Scale down Pods to **one replica** to save resources.

**Solution:**

Assuming `o3db-*` Pods are managed by a Deployment, ReplicaSet, or StatefulSet, we need to identify the controller managing them and scale it down to one replica.

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c1-H
   ```

2. **Identify the Controller Managing `o3db-*` Pods:**

   List all controllers in `project-c13` to find which one manages `o3db-*` Pods.

   ```bash
   kubectl get deployments,statefulsets,replicasets -n project-c13
   ```

   **Sample Output:**

   ```
   NAME         READY   UP-TO-DATE   AVAILABLE   AGE
   deploy-o3db   3/3     3            3           5d
   ```

   Assume a Deployment named `deploy-o3db` manages the `o3db-*` Pods.

3. **Scale Down the Deployment to One Replica:**

   ```bash
   kubectl scale deployment deploy-o3db --replicas=1 -n project-c13
   ```

4. **Verify Scaling:**

   ```bash
   kubectl get deployments -n project-c13
   kubectl get pods -n project-c13 -l app=o3db
   ```

   **Expected Output:**

   ```
   NAME        READY   UP-TO-DATE   AVAILABLE   AGE
   deploy-o3db 1/1     1            1           5d
   ```

   And for Pods:

   ```
   NAME       READY   STATUS    RESTARTS   AGE
   o3db-pod1   1/1     Running   0          10m
   ```

**Explanation:**

- Identifies the Deployment managing the `o3db-*` Pods.
- Scales the Deployment down to one replica, effectively reducing resource usage.

**Full Commands:**

```bash
# Q3: Set Kubernetes context
kubectl config use-context k8s-c1-H

# Q3: Identify controllers managing `o3db-*` Pods
kubectl get deployments,statefulsets,replicasets -n project-c13

# Q3: Scale down the Deployment to one replica
kubectl scale deployment deploy-o3db --replicas=1 -n project-c13

# Q3: Verify scaling
kubectl get deployments -n project-c13
kubectl get pods -n project-c13 -l app=o3db
```

---

### <a name="q4"></a>**Q4: Configure Liveness and Readiness Probes**

**Task:**
- Use context: `k8s-c1-H`.
- Namespace: `default`.
- Create a Pod named `ready-if-service-ready` with image `nginx:1.16.1-alpine`.
- Configure:
  - **LivenessProbe:** Executes `command: true`.
  - **ReadinessProbe:** Checks URL `http://service-am-i-ready:80` using `wget -T2 -O- http://service-am-i-ready:80`.
- Start the Pod and confirm it isn't ready because of the ReadinessProbe.
- Create a second Pod named `am-i-ready` with image `nginx:1.16.1-alpine` and label `id: cross-server-ready`.
- Ensure the existing Service `service-am-i-ready` now has the second Pod as an endpoint.
- Confirm that `ready-if-service-ready` Pod is now in a ready state.

**Solution:**

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c1-H
   ```

2. **Create the Pod `ready-if-service-ready` with Probes:**

   ```yaml
   # ready-if-service-ready.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: ready-if-service-ready
     namespace: default
   spec:
     containers:
       - name: nginx-container
         image: nginx:1.16.1-alpine
         livenessProbe:
           exec:
             command:
               - sh
               - -c
               - true
           initialDelaySeconds: 5
           periodSeconds: 10
         readinessProbe:
           exec:
             command:
               - sh
               - -c
               - wget -T2 -O- http://service-am-i-ready:80
           initialDelaySeconds: 5
           periodSeconds: 10
   ```

   **Explanation:**
   - **LivenessProbe:** Runs `true` command, always succeeds, keeping the Pod alive.
   - **ReadinessProbe:** Attempts to fetch `http://service-am-i-ready:80`. Initially, since no Pod backs the service, it should fail, marking the Pod as not ready.

3. **Apply the Pod Manifest:**

   ```bash
   kubectl apply -f ready-if-service-ready.yaml
   ```

4. **Verify Pod is Not Ready:**

   ```bash
   kubectl get pod ready-if-service-ready -n default
   ```

   **Expected Output:**

   ```
   NAME                      READY   STATUS    RESTARTS   AGE
   ready-if-service-ready    0/1     Running   0          1m
   ```

   The `READY` column shows `0/1`, indicating it's not ready.

5. **Create the Pod `am-i-ready` and Label It:**

   ```yaml
   # am-i-ready.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: am-i-ready
     namespace: default
     labels:
       id: cross-server-ready
   spec:
     containers:
       - name: nginx-container
         image: nginx:1.16.1-alpine
   ```

   **Explanation:**
   - This Pod is labeled to be selected by the Service `service-am-i-ready`.

6. **Apply the Pod Manifest:**

   ```bash
   kubectl apply -f am-i-ready.yaml
   ```

7. **Verify the Service `service-am-i-ready` has the Endpoint:**

   Assuming `service-am-i-ready` exists and selects Pods with label `id: cross-server-ready`.

   ```bash
   kubectl get endpoints service-am-i-ready -n default
   ```

   **Expected Output:**

   ```
   NAME                  ENDPOINTS             AGE
   service-am-i-ready    <IP_of_am-i-ready>:80   5m
   ```

8. **Confirm the `ready-if-service-ready` Pod is Ready:**

   Once `service-am-i-ready` has an endpoint, the ReadinessProbe should succeed.

   ```bash
   kubectl get pod ready-if-service-ready -n default
   ```

   **Expected Output:**

   ```
   NAME                      READY   STATUS    RESTARTS   AGE
   ready-if-service-ready    1/1     Running   0          6m
   ```

   The `READY` column shows `1/1`, indicating it's now ready.

**Full Commands and Manifests:**

```bash
# Q4: Set Kubernetes context
kubectl config use-context k8s-c1-H

# Q4: Create Pod with Probes
cat <<EOF | sudo tee ready-if-service-ready.yaml
apiVersion: v1
kind: Pod
metadata:
  name: ready-if-service-ready
  namespace: default
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16.1-alpine
      livenessProbe:
        exec:
          command:
            - sh
            - -c
            - true
        initialDelaySeconds: 5
        periodSeconds: 10
      readinessProbe:
        exec:
          command:
            - sh
            - -c
            - wget -T2 -O- http://service-am-i-ready:80
        initialDelaySeconds: 5
        periodSeconds: 10
EOF

# Q4: Apply the Pod manifest
kubectl apply -f ready-if-service-ready.yaml

# Q4: Verify Pod is not ready
kubectl get pod ready-if-service-ready -n default

# Q4: Create the second Pod `am-i-ready`
cat <<EOF | sudo tee am-i-ready.yaml
apiVersion: v1
kind: Pod
metadata:
  name: am-i-ready
  namespace: default
  labels:
    id: cross-server-ready
spec:
  containers:
    - name: nginx-container
      image: nginx:1.16.1-alpine
EOF

# Q4: Apply the Pod manifest
kubectl apply -f am-i-ready.yaml

# Q4: Verify Service Endpoints
kubectl get endpoints service-am-i-ready -n default

# Q4: Confirm `ready-if-service-ready` Pod is ready
kubectl get pod ready-if-service-ready -n default
```

---

### <a name="q5"></a>**Q5: List Pods Sorted by Age and UID**

**Task:**
- Use context: `k8s-c1-H`.
- Namespace: `default`.
- Write a command into `/opt/course/5/find_pods.sh` that lists all Pods sorted by their AGE (`metadata.creationTimestamp`).
- Write a second command into `/opt/course/5/find_pods_uid.sh` that lists all Pods sorted by `metadata.uid`.
- Use `kubectl` sorting for both commands.

**Solution:**

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c1-H
   ```

2. **Create the Shell Script `find_pods.sh` to List Pods Sorted by Age:**

   ```bash
   #!/bin/bash
   kubectl get pods --all-namespaces --sort-by=.metadata.creationTimestamp
   ```

   **Command to Create the Script:**

   ```bash
   echo "#!/bin/bash" | sudo tee /opt/course/5/find_pods.sh
   echo "kubectl get pods --all-namespaces --sort-by=.metadata.creationTimestamp" | sudo tee -a /opt/course/5/find_pods.sh
   sudo chmod +x /opt/course/5/find_pods.sh
   ```

3. **Create the Shell Script `find_pods_uid.sh` to List Pods Sorted by UID:**

   ```bash
   #!/bin/bash
   kubectl get pods --all-namespaces --sort-by=.metadata.uid
   ```

   **Command to Create the Script:**

   ```bash
   echo "#!/bin/bash" | sudo tee /opt/course/5/find_pods_uid.sh
   echo "kubectl get pods --all-namespaces --sort-by=.metadata.uid" | sudo tee -a /opt/course/5/find_pods_uid.sh
   sudo chmod +x /opt/course/5/find_pods_uid.sh
   ```

4. **Verify the Scripts:**

   ```bash
   cat /opt/course/5/find_pods.sh
   cat /opt/course/5/find_pods_uid.sh
   ```

**Explanation:**

- `--sort-by`: Kubernetes allows sorting resources based on a specified field.
- `.metadata.creationTimestamp`: Sorts Pods by their creation time (Age).
- `.metadata.uid`: Sorts Pods by their unique identifier.

**Full Commands:**

```bash
# Q5: Set Kubernetes context
kubectl config use-context k8s-c1-H

# Q5: Create find_pods.sh
echo "#!/bin/bash" | sudo tee /opt/course/5/find_pods.sh
echo "kubectl get pods --all-namespaces --sort-by=.metadata.creationTimestamp" | sudo tee -a /opt/course/5/find_pods.sh
sudo chmod +x /opt/course/5/find_pods.sh

# Q5: Create find_pods_uid.sh
echo "#!/bin/bash" | sudo tee /opt/course/5/find_pods_uid.sh
echo "kubectl get pods --all-namespaces --sort-by=.metadata.uid" | sudo tee -a /opt/course/5/find_pods_uid.sh
sudo chmod +x /opt/course/5/find_pods_uid.sh

# Q5: Verify the scripts
cat /opt/course/5/find_pods.sh
cat /opt/course/5/find_pods_uid.sh
```

---

### <a name="q6"></a>**Q6: Create PersistentVolume, PersistentVolumeClaim, and Deployment**

**Task:**
- Use context: `k8s-c1-H`.
- Namespace: `project-tiger`.
- Create:
  1. PersistentVolume named `safari-pv` with:
     - Capacity: `2Gi`
     - AccessMode: `ReadWriteOnce`
     - hostPath: `/Volumes/Data`
     - No `storageClassName` defined.
  2. PersistentVolumeClaim named `safari-pvc` with:
     - Requests: `2Gi` storage
     - AccessMode: `ReadWriteOnce`
     - No `storageClassName` defined.
     - Should bind to `safari-pv` correctly.
  3. Deployment named `safari` in `project-tiger` with:
     - Image: `httpd:2.4.41-alpine`
     - Mounts the PVC at `/tmp/safari-data`.

**Solution:**

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c1-H
   ```

2. **Create the PersistentVolume `safari-pv`:**

   ```yaml
   # safari-pv.yaml
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: safari-pv
   spec:
     capacity:
       storage: 2Gi
     accessModes:
       - ReadWriteOnce
     hostPath:
       path: /Volumes/Data
     storageClassName: ""
   ```

   **Explanation:**
   - `hostPath`: Directory on the host node.
   - `storageClassName: ""`: Makes the PV unbound by StorageClass, requiring the PVC to have `storageClassName: ""` or omitted.

3. **Apply the PersistentVolume:**

   ```bash
   kubectl apply -f safari-pv.yaml
   ```

4. **Create the PersistentVolumeClaim `safari-pvc`:**

   ```yaml
   # safari-pvc.yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: safari-pvc
     namespace: project-tiger
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 2Gi
     storageClassName: ""
   ```

   **Explanation:**
   - `storageClassName: ""`: Matches the PV without a StorageClass.

5. **Apply the PersistentVolumeClaim:**

   ```bash
   kubectl apply -f safari-pvc.yaml
   ```

6. **Verify PVC Binding:**

   ```bash
   kubectl get pvc -n project-tiger
   ```

   **Expected Output:**

   ```
   NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
   safari-pvc  Bound    safari-pv  2Gi        RWO            <none>         1m
   ```

7. **Create the Deployment `safari`:**

   ```yaml
   # safari-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: safari
     namespace: project-tiger
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: safari
     template:
       metadata:
         labels:
           app: safari
       spec:
         containers:
           - name: httpd
             image: httpd:2.4.41-alpine
             ports:
               - containerPort: 80
             volumeMounts:
               - mountPath: /tmp/safari-data
                 name: safari-storage
         volumes:
           - name: safari-storage
             persistentVolumeClaim:
               claimName: safari-pvc
   ```

8. **Apply the Deployment:**

   ```bash
   kubectl apply -f safari-deployment.yaml
   ```

9. **Verify Deployment and Pod:**

   ```bash
   kubectl get deployments -n project-tiger
   kubectl get pods -n project-tiger -l app=safari
   ```

   **Expected Output:**

   ```
   NAME     READY   UP-TO-DATE   AVAILABLE   AGE
   safari   1/1     1            1           1m
   ```

   ```
   NAME                    READY   STATUS    RESTARTS   AGE
   safari-xxxxxxxxxx-abcde   1/1     Running   0          1m
   ```

**Full Commands and Manifests:**

```bash
# Q6: Set Kubernetes context
kubectl config use-context k8s-c1-H

# Q6: Create PersistentVolume manifest
cat <<EOF | sudo tee safari-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: safari-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /Volumes/Data
  storageClassName: ""
EOF

# Q6: Apply PersistentVolume
kubectl apply -f safari-pv.yaml

# Q6: Create PersistentVolumeClaim manifest
cat <<EOF | sudo tee safari-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: safari-pvc
  namespace: project-tiger
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  storageClassName: ""
EOF

# Q6: Apply PersistentVolumeClaim
kubectl apply -f safari-pvc.yaml

# Q6: Verify PVC binding
kubectl get pvc -n project-tiger

# Q6: Create Deployment manifest
cat <<EOF | sudo tee safari-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: safari
  namespace: project-tiger
spec:
  replicas: 1
  selector:
    matchLabels:
      app: safari
  template:
    metadata:
      labels:
        app: safari
    spec:
      containers:
        - name: httpd
          image: httpd:2.4.41-alpine
          ports:
            - containerPort: 80
          volumeMounts:
            - mountPath: /tmp/safari-data
              name: safari-storage
      volumes:
        - name: safari-storage
          persistentVolumeClaim:
            claimName: safari-pvc
EOF

# Q6: Apply Deployment
kubectl apply -f safari-deployment.yaml

# Q6: Verify Deployment and Pod
kubectl get deployments -n project-tiger
kubectl get pods -n project-tiger -l app=safari
```

---

### <a name="q7"></a>**Q7: Kubectl Metrics Commands**

**Task:**
- Use context: `k8s-c1-H`.
- Metrics-server is installed.
- Write commands into:
  1. `/opt/course/7/node.sh` to show Nodes resource usage.
  2. `/opt/course/7/pod.sh` to show Pods and their containers resource usage.

**Solution:**

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c1-H
   ```

2. **Create Script `node.sh` to Show Nodes Resource Usage:**

   **Content:**

   ```bash
   #!/bin/bash
   kubectl top nodes
   ```

   **Command to Create the Script:**

   ```bash
   echo "#!/bin/bash" | sudo tee /opt/course/7/node.sh
   echo "kubectl top nodes" | sudo tee -a /opt/course/7/node.sh
   sudo chmod +x /opt/course/7/node.sh
   ```

3. **Create Script `pod.sh` to Show Pods and Containers Resource Usage:**

   **Content:**

   ```bash
   #!/bin/bash
   kubectl top pods --all-namespaces
   ```

   **Command to Create the Script:**

   ```bash
   echo "#!/bin/bash" | sudo tee /opt/course/7/pod.sh
   echo "kubectl top pods --all-namespaces" | sudo tee -a /opt/course/7/pod.sh
   sudo chmod +x /opt/course/7/pod.sh
   ```

4. **Verify the Scripts:**

   ```bash
   cat /opt/course/7/node.sh
   cat /opt/course/7/pod.sh
   ```

**Explanation:**

- `kubectl top nodes`: Displays CPU and memory usage for nodes.
- `kubectl top pods --all-namespaces`: Displays CPU and memory usage for all Pods across namespaces.

**Full Commands:**

```bash
# Q7: Set Kubernetes context
kubectl config use-context k8s-c1-H

# Q7: Create node.sh
echo "#!/bin/bash" | sudo tee /opt/course/7/node.sh
echo "kubectl top nodes" | sudo tee -a /opt/course/7/node.sh
sudo chmod +x /opt/course/7/node.sh

# Q7: Create pod.sh
echo "#!/bin/bash" | sudo tee /opt/course/7/pod.sh
echo "kubectl top pods --all-namespaces" | sudo tee -a /opt/course/7/pod.sh
sudo chmod +x /opt/course/7/pod.sh

# Q7: Verify scripts
cat /opt/course/7/node.sh
cat /opt/course/7/pod.sh
```

---

### <a name="q8"></a>**Q8: Identify Control Plane Components**

**Task:**
- Use context: `k8s-c1-H`.
- SSH into `cluster1-controlplane1`.
- Check how controlplane components (`kubelet`, `kube-apiserver`, `kube-scheduler`, `kube-controller-manager`, `etcd`) are started/installed.
- Find the DNS application name and its installation method.
- Write findings into `/opt/course/8/controlplane-components.txt` with the structure:

  ```
  # /opt/course/8/controlplane-components.txt
  kubelet: [TYPE]
  kube-apiserver: [TYPE]
  kube-scheduler: [TYPE]
  kube-controller-manager: [TYPE]
  etcd: [TYPE]
  dns: [TYPE] [NAME]
  ```

  **Choices for [TYPE]:** `not-installed`, `process`, `static-pod`, `pod`.

**Solution:**

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c1-H
   ```

2. **SSH into `cluster1-controlplane1`:**

   ```bash
   ssh user@cluster1-controlplane1
   ```

3. **Determine How Each Component is Started:**

   Kubernetes control plane components can be run as system services (`process`), static Pods (`static-pod`), or containerized Pods (`pod`).

   - **Check Static Pods:**

     Static Pods are usually defined in `/etc/kubernetes/manifests/`.

     ```bash
     ls /etc/kubernetes/manifests/
     ```

     **Sample Output:**

     ```
     kube-apiserver.yaml
     kube-controller-manager.yaml
     kube-scheduler.yaml
     etcd.yaml
     kube-proxy.yaml
     coredns.yaml
     ```

     The presence of YAML files indicates that these components are running as **static Pods**.

   - **Check kubelet:**

     kubelet runs as a systemd service.

     ```bash
     systemctl status kubelet
     ```

     **Sample Output:**

     ```
     ● kubelet.service - kubelet: The Kubernetes Node Agent
        Loaded: loaded (/etc/systemd/system/kubelet.service; enabled; vendor preset: enabled)
        Active: active (running) since ...
     ```

     Indicates that `kubelet` is running as a **process**.

   - **Check DNS Application:**

     Typically, CoreDNS is the DNS application.

     ```bash
     kubectl get pods -n kube-system -l k8s-app=kube-dns
     ```

     **Sample Output:**

     ```
     NAME                       READY   STATUS    RESTARTS   AGE
     coredns-66bff467f8-abcde   1/1     Running   0          10d
     coredns-66bff467f8-fghij   1/1     Running   0          10d
     ```

     **Installation Method:**

     CoreDNS is deployed as a **Deployment (pod)**.

4. **Compile Findings:**

   - `kubelet`: **process**
   - `kube-apiserver`: **static-pod**
   - `kube-scheduler`: **static-pod**
   - `kube-controller-manager`: **static-pod**
   - `etcd`: **static-pod**
   - `dns`: **pod** `coredns`

5. **Write Findings to File:**

   ```bash
   cat <<EOF | sudo tee /opt/course/8/controlplane-components.txt
# /opt/course/8/controlplane-components.txt
kubelet: process
kube-apiserver: static-pod
kube-scheduler: static-pod
kube-controller-manager: static-pod
etcd: static-pod
dns: pod coredns
EOF
   ```

6. **Verify the File Content:**

   ```bash
   cat /opt/course/8/controlplane-components.txt
   ```

   **Expected Content:**

   ```
   # /opt/course/8/controlplane-components.txt
   kubelet: process
   kube-apiserver: static-pod
   kube-scheduler: static-pod
   kube-controller-manager: static-pod
   etcd: static-pod
   dns: pod coredns
   ```

**Explanation:**

- Most controlplane components are managed as **static Pods** via YAML manifests.
- `kubelet` runs as a **system process** managed by `systemd`.
- DNS (CoreDNS) is deployed as a **Deployment** managing **Pods**.

**Full Commands:**

```bash
# Q8: Set Kubernetes context
kubectl config use-context k8s-c1-H

# Q8: SSH into controlplane node
ssh user@cluster1-controlplane1

# Q8: Check controlplane components
# List manifest files
ls /etc/kubernetes/manifests/

# Q8: Check kubelet status
systemctl status kubelet

# Q8: Identify DNS application
kubectl get pods -n kube-system -l k8s-app=kube-dns

# Q8: Write findings to file
cat <<EOF | sudo tee /opt/course/8/controlplane-components.txt
# /opt/course/8/controlplane-components.txt
kubelet: process
kube-apiserver: static-pod
kube-scheduler: static-pod
kube-controller-manager: static-pod
etcd: static-pod
dns: pod coredns
EOF

# Q8: Verify the file
cat /opt/course/8/controlplane-components.txt
```

---

### <a name="q9"></a>**Q9: Manual Scheduling of Pods by Stopping kube-scheduler**

**Task:**
- Use context: `k8s-c2-AC`.
- SSH into `cluster2-controlplane1`.
- Temporarily stop the `kube-scheduler`.
- Create a Pod named `manual-schedule` of image `httpd:2.4-alpine`.
- Confirm it's created but not scheduled.
- Manually schedule the Pod on node `cluster2-controlplane1`.
- Start the `kube-scheduler` again.
- Confirm it's running correctly by creating a second Pod named `manual-schedule2` and check if it's running on `cluster2-node1`.

**Solution:**

**Note:** Directly stopping `kube-scheduler` can lead to instability. Proceed with caution and preferably in a controlled environment.

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c2-AC
   ```

2. **SSH into `cluster2-controlplane1`:**

   ```bash
   ssh user@cluster2-controlplane1
   ```

3. **Stop the `kube-scheduler`:**

   The `kube-scheduler` typically runs as a static Pod or a systemd service.

   - **Check if running as static Pod:**
   
     ```bash
     ls /etc/kubernetes/manifests/
     ```
     
     If `kube-scheduler.yaml` exists, it's running as a **static Pod**.

     **Stop it by renaming the manifest:**
     
     ```bash
     sudo mv /etc/kubernetes/manifests/kube-scheduler.yaml /etc/kubernetes/manifests/kube-scheduler.yaml.bak
     ```

   - **If running as systemd service:**
   
     ```bash
     sudo systemctl stop kube-scheduler
     ```
     
     **(Not common in kubeadm setups.)**

4. **Confirm `kube-scheduler` is Stopped:**

   ```bash
   kubectl get pods -n kube-system | grep kube-scheduler
   ```

   **Expected Output:**

   ```
   kube-scheduler-cluster2-controlplane1   0/1     Pending   0          <AGE>
   ```

5. **Create the Pod `manual-schedule`:**

   ```yaml
   # manual-schedule.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: manual-schedule
     namespace: default
   spec:
     containers:
       - name: httpd
         image: httpd:2.4-alpine
   ```

   Apply the Pod:

   ```bash
   kubectl apply -f manual-schedule.yaml
   ```

6. **Verify Pod is Created but Not Scheduled:**

   ```bash
   kubectl get pod manual-schedule -n default -o wide
   ```

   **Expected Output:**

   ```
   NAME            READY   STATUS    RESTARTS   AGE   IP       NODE   NOMINATED NODE   READINESS GATES
   manual-schedule 0/1     Pending   0          10s   <none>   <none> <none>           <none>
   ```

7. **Manually Schedule the Pod on `cluster2-controlplane1`:**

   Since the `kube-scheduler` is stopped, we need to manually assign the Pod to a node using `kubectl`'s `--overrides` or by editing the Pod.

   **Option 1: Using `kubectl patch`:**

   ```bash
   kubectl patch pod manual-schedule -n default -p '{"spec":{"nodeName":"cluster2-controlplane1"}}'
   ```

   **Option 2: Delete and Recreate with Node Selector:**

   Alternatively, edit the Pod to include `nodeName`:

   ```bash
   kubectl edit pod manual-schedule -n default
   ```

   Add `nodeName` under `spec`:

   ```yaml
   spec:
     nodeName: cluster2-controlplane1
     containers:
       - name: httpd
         image: httpd:2.4-alpine
   ```

   Save and exit.

8. **Verify Pod is Running on `cluster2-controlplane1`:**

   ```bash
   kubectl get pod manual-schedule -n default -o wide
   ```

   **Expected Output:**

   ```
   NAME            READY   STATUS    RESTARTS   AGE   IP           NODE                   NOMINATED NODE   READINESS GATES
   manual-schedule 1/1     Running   0          1m    10.244.1.10 cluster2-controlplane1   <none>           <none>
   ```

9. **Restart the `kube-scheduler`:**

   - **If running as static Pod:**
   
     ```bash
     sudo mv /etc/kubernetes/manifests/kube-scheduler.yaml.bak /etc/kubernetes/manifests/kube-scheduler.yaml
     ```
     
     Kubelet will detect the manifest and recreate the `kube-scheduler` Pod.

   - **If running as systemd service:**
   
     ```bash
     sudo systemctl start kube-scheduler
     ```

10. **Verify `kube-scheduler` is Running:**

    ```bash
    kubectl get pods -n kube-system | grep kube-scheduler
    ```

    **Expected Output:**

    ```
    kube-scheduler-cluster2-controlplane1   1/1     Running   0          1m
    ```

11. **Create the Second Pod `manual-schedule2`:**

    ```yaml
    # manual-schedule2.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: manual-schedule2
      namespace: default
    spec:
      containers:
        - name: httpd
          image: httpd:2.4-alpine
    ```

    Apply the Pod:

    ```bash
    kubectl apply -f manual-schedule2.yaml
    ```

12. **Verify `manual-schedule2` is Scheduled on `cluster2-node1`:**

    ```bash
    kubectl get pod manual-schedule2 -n default -o wide
    ```

    **Expected Output:**

    ```
    NAME                 READY   STATUS    RESTARTS   AGE   IP           NODE             NOMINATED NODE   READINESS GATES
    manual-schedule2     1/1     Running   0          30s   10.244.2.5   cluster2-node1   <none>           <none>
    ```

**Explanation:**

- Stopping the `kube-scheduler` prevents automatic scheduling.
- Manually assigning the Pod using `nodeName` ensures it runs on a specific node.
- Restarting the `kube-scheduler` restores normal scheduling behavior, allowing new Pods to be scheduled automatically.

**Full Commands:**

```bash
# Q9: Set Kubernetes context
kubectl config use-context k8s-c2-AC

# Q9: SSH into controlplane node
ssh user@cluster2-controlplane1

# Q9: Stop kube-scheduler (assuming static Pod)
sudo mv /etc/kubernetes/manifests/kube-scheduler.yaml /etc/kubernetes/manifests/kube-scheduler.yaml.bak

# Q9: Create the Pod `manual-schedule`
cat <<EOF | sudo tee manual-schedule.yaml
apiVersion: v1
kind: Pod
metadata:
  name: manual-schedule
  namespace: default
spec:
  containers:
    - name: httpd
      image: httpd:2.4-alpine
EOF

# Q9: Apply the Pod manifest
kubectl apply -f manual-schedule.yaml

# Q9: Verify Pod is Pending
kubectl get pod manual-schedule -n default -o wide

# Q9: Manually schedule the Pod on `cluster2-controlplane1`
kubectl patch pod manual-schedule -n default -p '{"spec":{"nodeName":"cluster2-controlplane1"}}'

# Alternatively, edit the Pod to add nodeName
# kubectl edit pod manual-schedule -n default

# Q9: Verify Pod is Running on controlplane node
kubectl get pod manual-schedule -n default -o wide

# Q9: Restart kube-scheduler
sudo mv /etc/kubernetes/manifests/kube-scheduler.yaml.bak /etc/kubernetes/manifests/kube-scheduler.yaml

# Q9: Verify kube-scheduler is Running
kubectl get pods -n kube-system | grep kube-scheduler

# Q9: Create the second Pod `manual-schedule2`
cat <<EOF | sudo tee manual-schedule2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: manual-schedule2
  namespace: default
spec:
  containers:
    - name: httpd
      image: httpd:2.4-alpine
EOF

# Q9: Apply the Pod manifest
kubectl apply -f manual-schedule2.yaml

# Q9: Verify Pod is scheduled on `cluster2-node1`
kubectl get pod manual-schedule2 -n default -o wide
```

---

### <a name="q10"></a>**Q10: Create ServiceAccount, Role, and RoleBinding**

**Task:**
- Use context: `k8s-c1-H`.
- Namespace: `project-hamster`.
- Create:
  1. ServiceAccount named `processor`.
  2. Role named `processor` allowing only creation of Secrets and ConfigMaps in the Namespace.
  3. RoleBinding named `processor` binding the Role to the ServiceAccount.

**Solution:**

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c1-H
   ```

2. **Create the Namespace `project-hamster` (if not exists):**

   ```bash
   kubectl create namespace project-hamster
   ```

3. **Create the ServiceAccount `processor`:**

   ```bash
   kubectl create serviceaccount processor -n project-hamster
   ```

4. **Create the Role `processor` with Permissions to Create Secrets and ConfigMaps:**

   ```yaml
   # processor-role.yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     namespace: project-hamster
     name: processor
   rules:
     - apiGroups: [""]
       resources: ["secrets", "configmaps"]
       verbs: ["create"]
   ```

   Apply the Role:

   ```bash
   kubectl apply -f processor-role.yaml
   ```

5. **Create the RoleBinding `processor` Binding the Role to the ServiceAccount:**

   ```yaml
   # processor-rolebinding.yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: processor
     namespace: project-hamster
   subjects:
     - kind: ServiceAccount
       name: processor
       namespace: project-hamster
   roleRef:
     kind: Role
     name: processor
     apiGroup: rbac.authorization.k8s.io
   ```

   Apply the RoleBinding:

   ```bash
   kubectl apply -f processor-rolebinding.yaml
   ```

6. **Verify the Setup:**

   ```bash
   kubectl get serviceaccount processor -n project-hamster
   kubectl get role processor -n project-hamster
   kubectl get rolebinding processor -n project-hamster
   ```

   **Expected Output:**

   ```
   kubectl get serviceaccount processor -n project-hamster
   NAME        SECRETS   AGE
   processor   1         10m

   kubectl get role processor -n project-hamster
   NAME        CREATED AT
   processor   <timestamp>

   kubectl get rolebinding processor -n project-hamster
   NAME        ROLE    AGE
   processor   processor   10m
   ```

**Explanation:**

- **ServiceAccount:** Provides an identity for processes that run in a Pod.
- **Role:** Defines permissions (verbs) on resources within a Namespace.
- **RoleBinding:** Binds the Role to the ServiceAccount, granting it the defined permissions.

**Full Commands and Manifests:**

```bash
# Q10: Set Kubernetes context
kubectl config use-context k8s-c1-H

# Q10: Create Namespace if not exists
kubectl create namespace project-hamster

# Q10: Create ServiceAccount
kubectl create serviceaccount processor -n project-hamster

# Q10: Create Role manifest
cat <<EOF | sudo tee processor-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: project-hamster
  name: processor
rules:
  - apiGroups: [""]
    resources: ["secrets", "configmaps"]
    verbs: ["create"]
EOF

# Q10: Apply Role
kubectl apply -f processor-role.yaml

# Q10: Create RoleBinding manifest
cat <<EOF | sudo tee processor-rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: processor
  namespace: project-hamster
subjects:
  - kind: ServiceAccount
    name: processor
    namespace: project-hamster
roleRef:
  kind: Role
  name: processor
  apiGroup: rbac.authorization.k8s.io
EOF

# Q10: Apply RoleBinding
kubectl apply -f processor-rolebinding.yaml

# Q10: Verify the setup
kubectl get serviceaccount processor -n project-hamster
kubectl get role processor -n project-hamster
kubectl get rolebinding processor -n project-hamster
```

---

### <a name="q11"></a>**Q11: Create a DaemonSet**

**Task:**
- Use context: `k8s-c1-H`.
- Namespace: `project-tiger`.
- Create a DaemonSet named `ds-important` with:
  - Image: `httpd:2.4-alpine`.
  - Labels:
    - `id=ds-important`
    - `uuid=18426a0b-5f59-4e10-923f-c0e078e82462`.
  - Resource Requests:
    - CPU: `10m`
    - Memory: `10Mi`.
  - Pods should run on all nodes, including controlplane nodes.

**Solution:**

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c1-H
   ```

2. **Create the DaemonSet Manifest `ds-important.yaml`:**

   ```yaml
   # ds-important.yaml
   apiVersion: apps/v1
   kind: DaemonSet
   metadata:
     name: ds-important
     namespace: project-tiger
     labels:
       id: ds-important
       uuid: 18426a0b-5f59-4e10-923f-c0e078e82462
   spec:
     selector:
       matchLabels:
         id: ds-important
     template:
       metadata:
         labels:
           id: ds-important
           uuid: 18426a0b-5f59-4e10-923f-c0e078e82462
       spec:
         containers:
           - name: httpd
             image: httpd:2.4-alpine
             resources:
               requests:
                 cpu: "10m"
                 memory: "10Mi"
             ports:
               - containerPort: 80
         nodeSelector:
           node-role.kubernetes.io/controlplane: "true"
   ```

   **Explanation:**
   - **DaemonSet:** Ensures one Pod per node.
   - **Labels:** As specified.
   - **Resource Requests:** Ensures Pods request minimal resources.
   - **nodeSelector:** To include controlplane nodes, no need if no labels are specified. Since requirement is to run on all nodes including controlplanes, no nodeSelector is needed unless controlplane nodes are tainted. However, the problem states not to add labels, so the default DaemonSet behavior suffices.

   **Correction:** To run on all nodes, including controlplane nodes, ensure that controlplane nodes are **not tainted** to prevent DaemonSet Pods from being scheduled. If they are tainted, use tolerations.

   **Assumption:** Controlplane nodes are schedulable (no taints like `node-role.kubernetes.io/controlplane`).

3. **Adjust Manifest Without nodeSelector:**

   To ensure Pods run on all nodes, including controlplane nodes, **omit `nodeSelector`**.

   ```yaml
   # ds-important.yaml
   apiVersion: apps/v1
   kind: DaemonSet
   metadata:
     name: ds-important
     namespace: project-tiger
     labels:
       id: ds-important
       uuid: 18426a0b-5f59-4e10-923f-c0e078e82462
   spec:
     selector:
       matchLabels:
         id: ds-important
     template:
       metadata:
         labels:
           id: ds-important
           uuid: 18426a0b-5f59-4e10-923f-c0e078e82462
       spec:
         containers:
           - name: httpd
             image: httpd:2.4-alpine
             resources:
               requests:
                 cpu: "10m"
                 memory: "10Mi"
             ports:
               - containerPort: 80
   ```

4. **Apply the DaemonSet:**

   ```bash
   kubectl apply -f ds-important.yaml
   ```

5. **Verify DaemonSet and Pods:**

   ```bash
   kubectl get daemonsets -n project-tiger
   kubectl get pods -n project-tiger -l id=ds-important -o wide
   ```

   **Expected Output:**

   ```
   kubectl get daemonsets -n project-tiger
   NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
   ds-important   <number_of_nodes>   <same_number>   <same_number>   <same_number>   <same_number>   <none>          10m
   ```

   ```
   kubectl get pods -n project-tiger -l id=ds-important -o wide
   NAME                   READY   STATUS    RESTARTS   AGE   IP           NODE                  NOMINATED NODE   READINESS GATES
   ds-important-xxxxx     1/1     Running   0          5m    10.244.x.x   cluster1-node1         <none>           <none>
   ds-important-yyyyy     1/1     Running   0          5m    10.244.y.y   cluster1-node2         <none>           <none>
   # And so on for all nodes
   ```

**Explanation:**

- **DaemonSet:** Ensures that each node runs one instance of the Pod.
- **Labels:** Useful for identification and management.
- **Resource Requests:** Minimal to ensure low resource usage.
- **No `nodeSelector`:** Pods are scheduled on all nodes unless node taints prevent it.

**Full Commands and Manifest:**

```bash
# Q11: Set Kubernetes context
kubectl config use-context k8s-c1-H

# Q11: Create DaemonSet manifest
cat <<EOF | sudo tee ds-important.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-important
  namespace: project-tiger
  labels:
    id: ds-important
    uuid: 18426a0b-5f59-4e10-923f-c0e078e82462
spec:
  selector:
    matchLabels:
      id: ds-important
  template:
    metadata:
      labels:
        id: ds-important
        uuid: 18426a0b-5f59-4e10-923f-c0e078e82462
    spec:
      containers:
        - name: httpd
          image: httpd:2.4-alpine
          resources:
            requests:
              cpu: "10m"
              memory: "10Mi"
          ports:
            - containerPort: 80
EOF

# Q11: Apply the DaemonSet
kubectl apply -f ds-important.yaml

# Q11: Verify DaemonSet
kubectl get daemonsets -n project-tiger

# Q11: Verify Pods
kubectl get pods -n project-tiger -l id=ds-important -o wide
```

---

### <a name="q12"></a>**Q12: Create a Deployment with Specific Constraints**

**Task:**
- Use context: `k8s-c1-H`.
- Namespace: `project-tiger`.
- Implement:
  - **Deployment Name:** `deploy-important`.
  - **Replicas:** 3.
  - **Label for Deployment and Pods:** `id=very-important`.
  - **Containers:**
    1. `container1`:
       - Image: `nginx:1.17.6-alpine`.
    2. `container2`:
       - Image: `google/pause`.
  - **Pod Scheduling Constraint:** Only one Pod per worker node using `topologyKey: kubernetes.io/hostname`.

**Solution:**

To ensure only one Pod per node, use a **Pod Anti-Affinity** rule.

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c1-H
   ```

2. **Create the Deployment Manifest `deploy-important.yaml`:**

   ```yaml
   # deploy-important.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: deploy-important
     namespace: project-tiger
     labels:
       id: very-important
   spec:
     replicas: 3
     selector:
       matchLabels:
         id: very-important
     template:
       metadata:
         labels:
           id: very-important
       spec:
         affinity:
           podAntiAffinity:
             requiredDuringSchedulingIgnoredDuringExecution:
               - labelSelector:
                   matchLabels:
                     id: very-important
                 topologyKey: kubernetes.io/hostname
         containers:
           - name: container1
             image: nginx:1.17.6-alpine
           - name: container2
             image: google/pause
   ```

   **Explanation:**
   - **Pod Anti-Affinity:** Ensures Pods with label `id=very-important` are not scheduled on the same node.
   - **`topologyKey: kubernetes.io/hostname`:** Applies the anti-affinity per node.

3. **Apply the Deployment:**

   ```bash
   kubectl apply -f deploy-important.yaml
   ```

4. **Verify the Deployment and Pods:**

   ```bash
   kubectl get deployment deploy-important -n project-tiger
   kubectl get pods -n project-tiger -l id=very-important -o wide
   ```

   **Expected Output:**

   ```
   kubectl get deployment deploy-important -n project-tiger
   NAME             READY   UP-TO-DATE   AVAILABLE   AGE
   deploy-important   3/3     3            3           5m
   ```

   ```
   kubectl get pods -n project-tiger -l id=very-important -o wide
   NAME                             READY   STATUS    RESTARTS   AGE   IP           NODE             NOMINATED NODE   READINESS GATES
   deploy-important-xxxxx            2/2     Running   0          5m    10.244.x.x   cluster1-node1   <none>           <none>
   deploy-important-yyyyy            2/2     Running   0          5m    10.244.y.y   cluster1-node2   <none>           <none>
   deploy-important-zzzzz            2/2     Running   0          5m    10.244.z.z   cluster1-node3   <none>           <none>
   ```

   - **Note:** If there are only two worker nodes, the third Pod (`deploy-important-zzzzz`) will remain in `Pending` state due to anti-affinity constraints, simulating the desired behavior.

5. **Verify Pending Pod (If Applicable):**

   ```bash
   kubectl get pods -n project-tiger -l id=very-important
   ```

   **Expected Output:**

   ```
   NAME                             READY   STATUS     RESTARTS   AGE
   deploy-important-xxxxx            2/2     Running    0          5m
   deploy-important-yyyyy            2/2     Running    0          5m
   deploy-important-zzzzz            0/2     Pending    0          1m
   ```

**Explanation:**

- **Pod Anti-Affinity:** Ensures that only one Pod runs per node based on the `topologyKey`.
- **Deployments:** Manages the desired state, automatically handling Pod creation and scaling.

**Full Commands and Manifest:**

```bash
# Q12: Set Kubernetes context
kubectl config use-context k8s-c1-H

# Q12: Create Deployment manifest
cat <<EOF | sudo tee deploy-important.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-important
  namespace: project-tiger
  labels:
    id: very-important
spec:
  replicas: 3
  selector:
    matchLabels:
      id: very-important
  template:
    metadata:
      labels:
        id: very-important
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchLabels:
                  id: very-important
              topologyKey: kubernetes.io/hostname
      containers:
        - name: container1
          image: nginx:1.17.6-alpine
        - name: container2
          image: google/pause
EOF

# Q12: Apply the Deployment
kubectl apply -f deploy-important.yaml

# Q12: Verify Deployment
kubectl get deployment deploy-important -n project-tiger

# Q12: Verify Pods
kubectl get pods -n project-tiger -l id=very-important -o wide
```

---

### <a name="q13"></a>**Q13: Create a Multi-Container Pod with Shared Volume**

**Task:**
- Use context: `k8s-c1-H`.
- Namespace: `default`.
- Create a Pod named `multi-container-playground` with three containers: `c1`, `c2`, `c3`.
- Volume:
  - Shared among all containers.
  - Not persisted or shared with other Pods.
  - Mounted at a path (e.g., `/shared`).
- Container Details:
  1. **c1:**
     - Image: `nginx:1.17.6-alpine`.
     - Environment Variable: `MY_NODE_NAME` with the name of the node where the Pod is running.
  2. **c2:**
     - Image: `busybox:1.31.1`.
     - Command: Continuously write the output of the `date` command every second to `/shared/date.log`.
  3. **c3:**
     - Image: `busybox:1.31.1`.
     - Command: Continuously `tail -f /shared/date.log` to stdout.
- Verify logs of `c3` to confirm correct setup.

**Solution:**

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c1-H
   ```

2. **Create the Pod Manifest `multi-container-playground.yaml`:**

   ```yaml
   # multi-container-playground.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: multi-container-playground
     namespace: default
   spec:
     containers:
       - name: c1
         image: nginx:1.17.6-alpine
         env:
           - name: MY_NODE_NAME
             valueFrom:
               fieldRef:
                 fieldPath: spec.nodeName
         volumeMounts:
           - mountPath: /shared
             name: shared-volume
       - name: c2
         image: busybox:1.31.1
         command:
           - sh
           - -c
           - while true; do date >> /shared/date.log; sleep 1; done
         volumeMounts:
           - mountPath: /shared
             name: shared-volume
       - name: c3
         image: busybox:1.31.1
         command:
           - sh
           - -c
           - tail -f /shared/date.log
         volumeMounts:
           - mountPath: /shared
             name: shared-volume
     volumes:
       - name: shared-volume
         emptyDir: {}
   ```

   **Explanation:**
   - **emptyDir:** Provides a temporary directory shared among containers within the Pod. Data is stored on the node's filesystem and is deleted when the Pod is removed.
   - **Environment Variable `MY_NODE_NAME`:** Uses `fieldRef` to inject the node's name.
   - **Commands:**
     - **c2:** Appends the current date every second to `/shared/date.log`.
     - **c3:** Tails the `date.log` file to stdout, allowing you to observe the dates being written by `c2`.

3. **Apply the Pod Manifest:**

   ```bash
   kubectl apply -f multi-container-playground.yaml
   ```

4. **Verify the Pod is Running:**

   ```bash
   kubectl get pod multi-container-playground -n default
   ```

   **Expected Output:**

   ```
   NAME                          READY   STATUS    RESTARTS   AGE
   multi-container-playground    3/3     Running   0          1m
   ```

5. **Check Logs of Container `c3`:**

   ```bash
   kubectl logs multi-container-playground -c c3 -n default
   ```

   **Expected Output:**

   ```
   Mon Apr 25 12:00:00 UTC 2024
   Mon Apr 25 12:00:01 UTC 2024
   Mon Apr 25 12:00:02 UTC 2024
   ...
   ```

   **Explanation:**
   - The logs should continuously display the dates being written by `c2`.

**Full Commands and Manifest:**

```bash
# Q13: Set Kubernetes context
kubectl config use-context k8s-c1-H

# Q13: Create Pod manifest
cat <<EOF | sudo tee multi-container-playground.yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-playground
  namespace: default
spec:
  containers:
    - name: c1
      image: nginx:1.17.6-alpine
      env:
        - name: MY_NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
      volumeMounts:
        - mountPath: /shared
          name: shared-volume
    - name: c2
      image: busybox:1.31.1
      command:
        - sh
        - -c
        - while true; do date >> /shared/date.log; sleep 1; done
      volumeMounts:
        - mountPath: /shared
          name: shared-volume
    - name: c3
      image: busybox:1.31.1
      command:
        - sh
        - -c
        - tail -f /shared/date.log
      volumeMounts:
        - mountPath: /shared
          name: shared-volume
  volumes:
    - name: shared-volume
      emptyDir: {}
EOF

# Q13: Apply the Pod manifest
kubectl apply -f multi-container-playground.yaml

# Q13: Verify Pod is running
kubectl get pod multi-container-playground -n default

# Q13: Check logs of container c3
kubectl logs multi-container-playground -c c3 -n default
```

---

### <a name="q14"></a>**Q14: Gather Cluster Information**

**Task:**
- Use context: `k8s-c1-H`.
- Find the following about cluster `k8s-c1-H`:
  1. Number of controlplane nodes.
  2. Number of worker nodes.
  3. Service CIDR.
  4. Networking (CNI) Plugin configured and its config file location.
  5. Suffix of static Pods running on `cluster1-node1`.
- Write answers into `/opt/course/14/cluster-info` with structure:

  ```
  # /opt/course/14/cluster-info
  1: [ANSWER]
  2: [ANSWER]
  3: [ANSWER]
  4: [ANSWER]
  5: [ANSWER]
  ```

**Solution:**

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c1-H
   ```

2. **Gather Information:**

   **1. Number of Controlplane Nodes:**

   Controlplane nodes usually have the label `node-role.kubernetes.io/controlplane=true` or similar.

   ```bash
   kubectl get nodes -l node-role.kubernetes.io/controlplane=true --no-headers | wc -l
   ```

   **Alternative Label:**

   If labels differ, identify controlplane nodes based on labels:

   ```bash
   kubectl get nodes --show-labels | grep -i controlplane | wc -l
   ```

   **2. Number of Worker Nodes:**

   Worker nodes typically do not have controlplane labels.

   ```bash
   kubectl get nodes --no-headers | grep -v 'controlplane' | wc -l
   ```

   **3. Service CIDR:**

   Extract from cluster configuration. This is often set in the Pod Network (CNI) configuration or cluster's network settings.

   - **Check Pod Network CIDR:**

     ```bash
     kubectl cluster-info dump | grep -m 1 service-cluster-ip-range
     ```

   - **Alternatively, Inspect Network Plugin Config:**

     For example, Calico:

     ```bash
     kubectl get configmap -n kube-system calico-config -o yaml | grep -i service-cluster-ip-range
     ```

   **Sample Output:**

   ```
   service-cluster-ip-range: 10.96.0.0/12
   ```

   **4. Networking (CNI) Plugin and Config File:**

   - **Identify CNI Plugin:**

     Check installed network plugins, usually via Pods in `kube-system`.

     ```bash
     kubectl get pods -n kube-system -o wide
     ```

     Look for Pods like `calico`, `weave-net`, `cilium`, `flannel`, etc.

   - **Find Config File Location:**

     Network plugins typically have ConfigMaps or manifest files.

     ```bash
     kubectl get configmaps -n kube-system
     ```

     For Calico, for example:

     ```bash
     kubectl get configmap calico-config -n kube-system -o yaml
     ```

     The config file is often at `/etc/cni/net.d/` on nodes.

   **5. Suffix of Static Pods on `cluster1-node1`:**

   Static Pods are named with a suffix `-controlplane` or similar.

   - **List Static Pods on `cluster1-node1`:**

     ```bash
     ssh user@cluster1-node1
     ls /etc/kubernetes/manifests/
     ```

     **Sample Output:**

     ```
     kube-apiserver.yaml
     kube-controller-manager.yaml
     kube-scheduler.yaml
     etcd.yaml
     kube-proxy.yaml
     coredns.yaml
     ```

     **Suffix:** Typically `.yaml`.

     Alternatively, the Pod names may include node names.

     - **Check Pod Names:**

       ```bash
       kubectl get pods -n kube-system -o wide | grep cluster1-node1
       ```

       **Sample Output:**

       ```
       kube-apiserver-cluster1-controlplane1   1/1     Running   0          10d   cluster1-controlplane1
       kube-controller-manager-cluster1-controlplane1   1/1     Running   0          10d   cluster1-controlplane1
       kube-scheduler-cluster1-controlplane1       1/1     Running   0          10d   cluster1-controlplane1
       etcd-cluster1-controlplane1                 1/1     Running   0          10d   cluster1-controlplane1
       ```

       **Suffix:** `-controlplane1`.

3. **Compile Findings:**

   Suppose the following:
   - **Number of Controlplane Nodes:** 1.
   - **Number of Worker Nodes:** 2.
   - **Service CIDR:** `10.96.0.0/12`.
   - **CNI Plugin:** `Calico`.
   - **Config File Location:** `/etc/cni/net.d/calico.conf`.
   - **Suffix of Static Pods:** `-controlplane1`.

4. **Write to File:**

   ```bash
   cat <<EOF | sudo tee /opt/course/14/cluster-info
# /opt/course/14/cluster-info
1: 1
2: 2
3: 10.96.0.0/12
4: Calico /etc/cni/net.d/calico.conf
5: -controlplane1
EOF
   ```

5. **Verify the File Content:**

   ```bash
   cat /opt/course/14/cluster-info
   ```

   **Expected Content:**

   ```
   # /opt/course/14/cluster-info
   1: 1
   2: 2
   3: 10.96.0.0/12
   4: Calico /etc/cni/net.d/calico.conf
   5: -controlplane1
   ```

**Explanation:**

- **Controlplane Nodes:** Identified via labels.
- **Worker Nodes:** Counted as nodes without controlplane labels.
- **Service CIDR:** Determined from cluster or network plugin config.
- **CNI Plugin:** Identified by deployed network Pods.
- **Static Pods Suffix:** Based on Pod naming conventions.

**Full Commands:**

```bash
# Q14: Set Kubernetes context
kubectl config use-context k8s-c1-H

# Q14: Count Controlplane Nodes
controlplane_count=$(kubectl get nodes -l node-role.kubernetes.io/controlplane=true --no-headers | wc -l)

# Q14: Count Worker Nodes
worker_count=$(kubectl get nodes --no-headers | grep -v 'controlplane' | wc -l)

# Q14: Find Service CIDR
service_cidr=$(kubectl cluster-info dump | grep -m 1 service-cluster-ip-range | awk '{print $2}')

# Q14: Identify CNI Plugin and Config File
cni_plugin=$(kubectl get pods -n kube-system -o wide | grep -i 'calico\|weave\|cilium\|flannel' | awk '{print $1}' | head -n1 | cut -d'-' -f1)
if [ "$cni_plugin" == "calico" ]; then
    cni_config="/etc/cni/net.d/calico.conf"
elif [ "$cni_plugin" == "weave" ]; then
    cni_config="/etc/cni/net.d/weave.conf"
elif [ "$cni_plugin" == "cilium" ]; then
    cni_config="/etc/cni/net.d/cilium.conf"
elif [ "$cni_plugin" == "flannel" ]; then
    cni_config="/etc/cni/net.d/flannel.conf"
else
    cni_plugin="Unknown"
    cni_config="N/A"
fi

# Q14: Determine Static Pods Suffix
static_pod_suffix=$(kubectl get pods -n kube-system | grep cluster1-controlplane1 | awk '{print $1}' | awk -F'-' '{print "-"$3}')

# Q14: Write to File
cat <<EOF | sudo tee /opt/course/14/cluster-info
# /opt/course/14/cluster-info
1: $controlplane_count
2: $worker_count
3: $service_cidr
4: $cni_plugin $cni_config
5: $static_pod_suffix
EOF

# Q14: Verify the file
cat /opt/course/14/cluster-info
```

---

### <a name="q15"></a>**Q15: Monitor and Log Cluster Events**

**Task:**
- Use context: `k8s-c2-AC`.
- Write a command into `/opt/course/15/cluster_events.sh` to show the latest events in the whole cluster, ordered by time (`metadata.creationTimestamp`) using `kubectl`.
- Delete the `kube-proxy` Pod running on node `cluster2-node1` and write the events caused into `/opt/course/15/pod_kill.log`.
- Kill the `kube-proxy` container in `cluster2-node1` using `containerd` and write the events into `/opt/course/15/container_kill.log`.
- Observe differences in events caused by both actions.

**Solution:**

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c2-AC
   ```

2. **Create Script `cluster_events.sh` to Show Latest Events Sorted by Time:**

   ```bash
   #!/bin/bash
   kubectl get events --all-namespaces --sort-by=.metadata.creationTimestamp
   ```

   **Command to Create the Script:**

   ```bash
   echo "#!/bin/bash" | sudo tee /opt/course/15/cluster_events.sh
   echo "kubectl get events --all-namespaces --sort-by=.metadata.creationTimestamp" | sudo tee -a /opt/course/15/cluster_events.sh
   sudo chmod +x /opt/course/15/cluster_events.sh
   ```

3. **Delete the `kube-proxy` Pod on `cluster2-node1`:**

   **Identify the Pod:**

   ```bash
   kubectl get pods -n kube-system -o wide | grep kube-proxy | grep cluster2-node1
   ```

   **Sample Output:**

   ```
   kube-proxy-cluster2-node1   1/1     Running   0          10d   cluster2-node1
   ```

   **Delete the Pod:**

   ```bash
   kubectl delete pod kube-proxy-cluster2-node1 -n kube-system
   ```

4. **Capture Events After Pod Deletion:**

   ```bash
   /opt/course/15/cluster_events.sh | grep kube-proxy | tail -n 10 | sudo tee /opt/course/15/pod_kill.log
   ```

5. **Kill the `kube-proxy` Container Using `containerd`:**

   **SSH into `cluster2-node1`:**

   ```bash
   ssh user@cluster2-node1
   ```

   **List Containers and Identify `kube-proxy` Container:**

   ```bash
   sudo crictl ps | grep kube-proxy
   ```

   **Sample Output:**

   ```
   CONTAINER ID   POD ID        IMAGE                      CREATED         STATE           NAME                ATTEMPT
   abcdef123456   pod-xyz       k8s.gcr.io/kube-proxy:v1.31.0   10d ago         Running         kube-proxy          0
   ```

   **Kill the Container:**

   ```bash
   sudo crictl kill abcdef123456
   ```

   **Exit the Node:**

   ```bash
   exit
   ```

6. **Capture Events After Container Kill:**

   ```bash
   /opt/course/15/cluster_events.sh | grep kube-proxy | tail -n 10 | sudo tee /opt/course/15/container_kill.log
   ```

7. **Compare the Logs:**

   ```bash
   diff /opt/course/15/pod_kill.log /opt/course/15/container_kill.log
   ```

   **Expected Outcome:**

   - **Pod Deletion (`pod_kill.log`):** Event indicating the Pod was deleted, possibly with reason `Deleted`.
   - **Container Kill (`container_kill.log`):** Events related to container termination, possibly with reason `Killing` or `ContainerKilled`.

**Explanation:**

- **Deleting the Pod:** Triggers Kubernetes to terminate and potentially restart the Pod.
- **Killing the Container:** Simulates an abrupt container termination without Pod deletion, prompting Kubernetes to restart the container within the Pod.

**Full Commands:**

```bash
# Q15: Set Kubernetes context
kubectl config use-context k8s-c2-AC

# Q15: Create cluster_events.sh
echo "#!/bin/bash" | sudo tee /opt/course/15/cluster_events.sh
echo "kubectl get events --all-namespaces --sort-by=.metadata.creationTimestamp" | sudo tee -a /opt/course/15/cluster_events.sh
sudo chmod +x /opt/course/15/cluster_events.sh

# Q15: Identify and delete kube-proxy Pod on cluster2-node1
kubectl get pods -n kube-system -o wide | grep kube-proxy | grep cluster2-node1
kubectl delete pod kube-proxy-cluster2-node1 -n kube-system

# Q15: Capture events after Pod deletion
/opt/course/15/cluster_events.sh | grep kube-proxy | tail -n 10 | sudo tee /opt/course/15/pod_kill.log

# Q15: SSH into node to kill kube-proxy container
ssh user@cluster2-node1

# Q15: List kube-proxy containers
sudo crictl ps | grep kube-proxy

# Q15: Kill the kube-proxy container (replace CONTAINER_ID with actual ID)
sudo crictl kill abcdef123456

# Q15: Exit the node
exit

# Q15: Capture events after container kill
/opt/course/15/cluster_events.sh | grep kube-proxy | tail -n 10 | sudo tee /opt/course/15/container_kill.log

# Q15: Compare the logs
diff /opt/course/15/pod_kill.log /opt/course/15/container_kill.log
```

---

### <a name="q16"></a>**Q16: List Namespaced Resources and Identify Crowded Namespace**

**Task:**
- Use context: `k8s-c1-H`.
- Namespace: All namespaces.
- Create:
  1. Command in `/opt/course/16/resources.txt` listing all namespaced Kubernetes resources (e.g., Pod, Secret, ConfigMap).
  2. Find the `project-*` Namespace with the highest number of Roles defined and write its name and count into `/opt/course/16/crowded-namespace.txt`.

**Solution:**

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c1-H
   ```

2. **List All Namespaced Resources and Save to `resources.txt`:**

   - **Retrieve all API Resources and Filter Namespaced:**

     ```bash
     kubectl api-resources --namespaced=true -o name | tr '\n' ' ' | sudo tee /opt/course/16/resources.txt
     ```

     **Alternatively, List in YAML Format:**

     ```bash
     kubectl api-resources --namespaced=true -o wide | awk '{print $1}' | grep -v NAME | sort | uniq | tr '\n' ' ' | sudo tee /opt/course/16/resources.txt
     ```

   - **Example Content:**

     ```
     pods secrets configmaps services deployments replicasets statefulsets daemonsets jobs cronjobs ingress networkpolicies roles rolebindings
     ```

3. **Find `project-*` Namespace with Highest Number of Roles:**

   - **List Namespaces Matching `project-*`:**

     ```bash
     namespaces=$(kubectl get namespaces -o jsonpath='{.items[*].metadata.name}' | tr ' ' '\n' | grep '^project-')
     ```

   - **Iterate Over Namespaces to Count Roles:**

     ```bash
     max_count=0
     max_ns=""
     for ns in $namespaces; do
       count=$(kubectl get roles -n "$ns" --no-headers | wc -l)
       if [ "$count" -gt "$max_count" ]; then
         max_count=$count
         max_ns=$ns
       fi
     done
     ```

   - **Write to `crowded-namespace.txt`:**

     ```bash
     echo "# /opt/course/16/crowded-namespace.txt" | sudo tee /opt/course/16/crowded-namespace.txt
     echo "Namespace: $max_ns, Roles Count: $max_count" | sudo tee -a /opt/course/16/crowded-namespace.txt
     ```

4. **Combine Commands:**

   **Full Script:**

   ```bash
   #!/bin/bash

   # List all namespaced resources
   kubectl api-resources --namespaced=true -o name | tr '\n' ' ' | sudo tee /opt/course/16/resources.txt

   # Find the project-* Namespace with the highest number of Roles
   namespaces=$(kubectl get namespaces -o jsonpath='{.items[*].metadata.name}' | tr ' ' '\n' | grep '^project-')

   max_count=0
   max_ns=""

   for ns in $namespaces; do
     count=$(kubectl get roles -n "$ns" --no-headers | wc -l)
     if [ "$count" -gt "$max_count" ]; then
       max_count=$count
       max_ns=$ns
     fi
   done

   # Write to crowded-namespace.txt
   echo "# /opt/course/16/crowded-namespace.txt" | sudo tee /opt/course/16/crowded-namespace.txt
   echo "Namespace: $max_ns, Roles Count: $max_count" | sudo tee -a /opt/course/16/crowded-namespace.txt
   ```

5. **Run the Commands:**

   ```bash
   # Q16: Set Kubernetes context
   kubectl config use-context k8s-c1-H

   # Q16: List namespaced resources
   kubectl api-resources --namespaced=true -o name | tr '\n' ' ' | sudo tee /opt/course/16/resources.txt

   # Q16: Find the crowded namespace
   namespaces=$(kubectl get namespaces -o jsonpath='{.items[*].metadata.name}' | tr ' ' '\n' | grep '^project-')

   max_count=0
   max_ns=""

   for ns in $namespaces; do
     count=$(kubectl get roles -n "$ns" --no-headers | wc -l)
     if [ "$count" -gt "$max_count" ]; then
       max_count=$count
       max_ns=$ns
     fi
   done

   # Write to crowded-namespace.txt
   echo "# /opt/course/16/crowded-namespace.txt" | sudo tee /opt/course/16/crowded-namespace.txt
   echo "Namespace: $max_ns, Roles Count: $max_count" | sudo tee -a /opt/course/16/crowded-namespace.txt
   ```

6. **Verify the Files:**

   ```bash
   cat /opt/course/16/resources.txt
   cat /opt/course/16/crowded-namespace.txt
   ```

**Explanation:**

- **Listing Namespaced Resources:** Utilizes `kubectl api-resources` to fetch all resources that are namespaced.
- **Identifying Crowded Namespace:** Iterates through `project-*` Namespaces to find which has the most Roles.

**Full Commands:**

```bash
# Q16: Set Kubernetes context
kubectl config use-context k8s-c1-H

# Q16: List namespaced resources
kubectl api-resources --namespaced=true -o name | tr '\n' ' ' | sudo tee /opt/course/16/resources.txt

# Q16: Find the crowded namespace
namespaces=$(kubectl get namespaces -o jsonpath='{.items[*].metadata.name}' | tr ' ' '\n' | grep '^project-')

max_count=0
max_ns=""

for ns in $namespaces; do
  count=$(kubectl get roles -n "$ns" --no-headers | wc -l)
  if [ "$count" -gt "$max_count" ]; then
    max_count=$count
    max_ns=$ns
  fi
done

# Q16: Write to crowded-namespace.txt
echo "# /opt/course/16/crowded-namespace.txt" | sudo tee /opt/course/16/crowded-namespace.txt
echo "Namespace: $max_ns, Roles Count: $max_count" | sudo tee -a /opt/course/16/crowded-namespace.txt

# Q16: Verify the files
cat /opt/course/16/resources.txt
cat /opt/course/16/crowded-namespace.txt
```

---

### <a name="q17"></a>**Q17: Retrieve Containerd Information for a Pod**

**Task:**
- Use context: `k8s-c1-H`.
- Namespace: `project-tiger`.
- Create a Pod named `tigers-reunite` with image `httpd:2.4.41-alpine` and labels `pod=container` and `container=pod`.
- Find out on which node the Pod is scheduled.
- SSH into that node and find the `containerd` container belonging to that Pod.
- Using `crictl`:
  1. Write the ID of the container and the `info.runtimeType` into `/opt/course/17/pod-container.txt`.
  2. Write the logs of the container into `/opt/course/17/pod-container.log`.

**Solution:**

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c1-H
   ```

2. **Create the Pod `tigers-reunite`:**

   ```yaml
   # tigers-reunite.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: tigers-reunite
     namespace: project-tiger
     labels:
       pod: container
       container: pod
   spec:
     containers:
       - name: httpd-container
         image: httpd:2.4.41-alpine
         ports:
           - containerPort: 80
   ```

   **Apply the Pod Manifest:**

   ```bash
   kubectl apply -f tigers-reunite.yaml
   ```

3. **Verify the Pod is Running and Identify the Node:**

   ```bash
   kubectl get pod tigers-reunite -n project-tiger -o wide
   ```

   **Sample Output:**

   ```
   NAME            READY   STATUS    RESTARTS   AGE   IP           NODE                NOMINATED NODE   READINESS GATES
   tigers-reunite   1/1     Running   0          5m    10.244.x.x   cluster1-node1      <none>           <none>
   ```

   **Node:** `cluster1-node1`

4. **SSH into the Node:**

   ```bash
   ssh user@cluster1-node1
   ```

5. **Install `crictl` (If Not Already Installed):**

   ```bash
   # Download the latest crictl (replace version as needed)
   VERSION="v1.25.0"
   curl -LO https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
   sudo tar -C /usr/local/bin -xzf crictl-$VERSION-linux-amd64.tar.gz
   rm crictl-$VERSION-linux-amd64.tar.gz

   # Verify installation
   crictl version
   ```

6. **Find the Container ID for `tigers-reunite`:**

   - **List Containers:**

     ```bash
     crictl ps | grep tigers-reunite
     ```

     **Sample Output:**

     ```
     CONTAINER ID   POD ID        IMAGE                    CREATED          STATE           NAME                ATTEMPT
     abcdef123456   pod-xyz       httpd:2.4.41-alpine      5m ago           Running         httpd-container     0
     ```

     - **Container ID:** `abcdef123456`

7. **Retrieve `runtimeType` of the Container:**

   ```bash
   crictl inspect $CONTAINER_ID | grep -i runtimeType
   ```

   **Sample Output:**

   ```
   "runtimeType": "containerd",
   ```

8. **Write Container ID and `runtimeType` to `pod-container.txt`:**

   ```bash
   echo "Container ID: $CONTAINER_ID" | sudo tee /opt/course/17/pod-container.txt
   crictl inspect $CONTAINER_ID | grep -i runtimeType | awk -F'"' '{print $4}' | sudo tee -a /opt/course/17/pod-container.txt
   ```

   **Sample Content:**

   ```
   Container ID: abcdef123456
   containerd
   ```

9. **Retrieve Logs of the Container and Write to `pod-container.log`:**

   ```bash
   crictl logs $CONTAINER_ID | sudo tee /opt/course/17/pod-container.log
   ```

10. **Exit the Node:**

    ```bash
    exit
    ```

**Explanation:**

- **`crictl`:** A CLI for CRI-compatible container runtimes like `containerd`.
- **Identifying Container:** Uses `crictl ps` to list containers and find the relevant one.
- **Inspecting Container:** Retrieves detailed information, including `runtimeType`.
- **Logs:** Fetches container logs for analysis.

**Full Commands:**

```bash
# Q17: Set Kubernetes context
kubectl config use-context k8s-c1-H

# Q17: Create Pod manifest
cat <<EOF | sudo tee tigers-reunite.yaml
apiVersion: v1
kind: Pod
metadata:
  name: tigers-reunite
  namespace: project-tiger
  labels:
    pod: container
    container: pod
spec:
  containers:
    - name: httpd-container
      image: httpd:2.4.41-alpine
      ports:
        - containerPort: 80
EOF

# Q17: Apply the Pod manifest
kubectl apply -f tigers-reunite.yaml

# Q17: Get Pod details to find the node
kubectl get pod tigers-reunite -n project-tiger -o wide

# Q17: SSH into the node (replace with actual node name)
ssh user@cluster1-node1

# Q17: Install crictl if not present
VERSION="v1.25.0"
curl -LO https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
sudo tar -C /usr/local/bin -xzf crictl-$VERSION-linux-amd64.tar.gz
rm crictl-$VERSION-linux-amd64.tar.gz
crictl version

# Q17: Find the container ID for the Pod's container
crictl ps | grep tigers-reunite

# Q17: Assuming Container ID is abcdef123456
CONTAINER_ID=abcdef123456

# Q17: Retrieve runtimeType and write to pod-container.txt
echo "Container ID: $CONTAINER_ID" | sudo tee /opt/course/17/pod-container.txt
crictl inspect $CONTAINER_ID | grep -i runtimeType | awk -F'"' '{print $4}' | sudo tee -a /opt/course/17/pod-container.txt

# Q17: Retrieve logs and write to pod-container.log
crictl logs $CONTAINER_ID | sudo tee /opt/course/17/pod-container.log

# Q17: Exit the node
exit
```

---

### <a name="q18"></a>**Q18: Troubleshoot kubelet on `cluster3-node1`**

**Task:**
- Use context: `k8s-c3-CCC`.
- Fix the issue with `kubelet` not running on `cluster3-node1`.
- Confirm the cluster has node `cluster3-node1` in `Ready` state.
- Schedule a Pod on `cluster3-node1`.
- Write the reason for the issue into `/opt/course/18/reason.txt`.

**Solution:**

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c3-CCC
   ```

2. **Check Node Status:**

   ```bash
   kubectl get nodes
   ```

   **Sample Output:**

   ```
   NAME                STATUS     ROLES                  AGE   VERSION
   cluster3-controlplane1 Ready      controlplane,master    10d   v1.31.0
   cluster3-node1         NotReady   <none>                 10d   v1.31.0
   ```

3. **SSH into `cluster3-node1`:**

   ```bash
   ssh user@cluster3-node1
   ```

4. **Check kubelet Service Status:**

   ```bash
   sudo systemctl status kubelet
   ```

   **Possible Scenarios:**

   - **kubelet Service Not Running:**

     ```
     ● kubelet.service - kubelet: The Kubernetes Node Agent
        Loaded: loaded (/etc/systemd/system/kubelet.service; enabled; vendor preset: enabled)
        Active: inactive (dead) since ...
     ```

   - **kubelet Service Failing:**

     ```
     ● kubelet.service - kubelet: The Kubernetes Node Agent
        Loaded: loaded (/etc/systemd/system/kubelet.service; enabled; vendor preset: enabled)
        Active: failed (Result: exit-code) since ...
     ```

5. **Investigate kubelet Logs:**

   ```bash
   sudo journalctl -u kubelet -xe
   ```

   **Common Issues:**
   - **Configuration Errors:** Incorrect kubelet config or flags.
   - **Certificate Issues:** Expired or missing certificates.
   - **Network Issues:** Unable to communicate with the API server.

6. **Possible Fixes Based on Common Issues:**

   - **Restart kubelet Service:**

     ```bash
     sudo systemctl restart kubelet
     ```

     **Check Status:**

     ```bash
     sudo systemctl status kubelet
     ```

   - **Check kubelet Configuration:**

     Ensure `/var/lib/kubelet/config.yaml` is correct.

     ```bash
     sudo cat /var/lib/kubelet/config.yaml
     ```

   - **Check Certificates:**

     Verify that certificates are present and not expired.

     ```bash
     sudo ls /etc/kubernetes/pki/
     sudo openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -enddate
     ```

   - **Reinstall or Reconfigure kubelet:**

     If configuration is corrupted, reinitialize kubelet.

7. **Assuming the Issue was kubelet Service Not Running:**

   **Fix: Restart kubelet and Verify:**

   ```bash
   sudo systemctl restart kubelet
   sudo systemctl status kubelet
   ```

   **Assuming it starts successfully, mark the reason:**

   ```bash
   echo "kubelet service was not running due to it being stopped accidentally. Restarting the service resolved the issue." | sudo tee /opt/course/18/reason.txt
   ```

8. **Exit the Node and Verify Node is Ready:**

   ```bash
   exit
   kubectl get nodes
   ```

   **Expected Output:**

   ```
   NAME                  STATUS   ROLES                  AGE   VERSION
   cluster3-controlplane1 Ready    controlplane,master    10d   v1.31.0
   cluster3-node1         Ready    <none>                 10d   v1.31.0
   ```

9. **Schedule a Pod on `cluster3-node1`:**

   Create a simple Pod with node affinity or specify the node.

   **Option 1: Using `nodeName`:**

   ```yaml
   # test-pod-cluster3-node1.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: test-pod-cluster3-node1
     namespace: default
   spec:
     containers:
       - name: nginx
         image: nginx:1.17.6-alpine
     nodeName: cluster3-node1
   ```

   Apply the Pod:

   ```bash
   kubectl apply -f test-pod-cluster3-node1.yaml
   ```

   **Option 2: Using Node Affinity:**

   ```yaml
   # test-pod-cluster3-node1.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: test-pod-cluster3-node1
     namespace: default
   spec:
     affinity:
       nodeAffinity:
         requiredDuringSchedulingIgnoredDuringExecution:
           nodeSelectorTerms:
             - matchExpressions:
                 - key: kubernetes.io/hostname
                   operator: In
                   values:
                     - cluster3-node1
     containers:
       - name: nginx
         image: nginx:1.17.6-alpine
   ```

   Apply the Pod:

   ```bash
   kubectl apply -f test-pod-cluster3-node1.yaml
   ```

10. **Verify the Pod is Running on `cluster3-node1`:**

    ```bash
    kubectl get pod test-pod-cluster3-node1 -n default -o wide
    ```

    **Expected Output:**

    ```
    NAME                           READY   STATUS    RESTARTS   AGE   IP           NODE             NOMINATED NODE   READINESS GATES
    test-pod-cluster3-node1        1/1     Running   0          1m    10.244.3.5   cluster3-node1   <none>           <none>
    ```

**Explanation:**

- **kubelet Issues:** Could be due to the kubelet service being stopped or misconfigured.
- **Fix:** Restarting the kubelet service often resolves the issue.
- **Verification:** Ensuring the node is `Ready` and scheduling Pods confirms kubelet functionality.

**Full Commands:**

```bash
# Q18: Set Kubernetes context
kubectl config use-context k8s-c3-CCC

# Q18: SSH into node
ssh user@cluster3-node1

# Q18: Check kubelet status
sudo systemctl status kubelet

# Q18: If kubelet not running, restart it
sudo systemctl restart kubelet

# Q18: Check kubelet status again
sudo systemctl status kubelet

# Q18: Investigate logs if needed
sudo journalctl -u kubelet -xe

# Q18: Assuming fix was restarting kubelet, write reason
echo "kubelet service was not running due to being stopped accidentally. Restarting the service resolved the issue." | sudo tee /opt/course/18/reason.txt

# Q18: Exit the node
exit

# Q18: Verify node is Ready
kubectl get nodes

# Q18: Create Pod scheduled on cluster3-node1 using nodeName
cat <<EOF | sudo tee test-pod-cluster3-node1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod-cluster3-node1
  namespace: default
spec:
  containers:
    - name: nginx
      image: nginx:1.17.6-alpine
  nodeName: cluster3-node1
EOF

# Q18: Apply the Pod
kubectl apply -f test-pod-cluster3-node1.yaml

# Q18: Verify Pod is running on cluster3-node1
kubectl get pod test-pod-cluster3-node1 -n default -o wide
```

---

### <a name="q19"></a>**Q19: Manage Secrets in a Pod**

**Task:**
- Use context: `k8s-c3-CCC`.
- Namespace: `secret`.
- Create a Pod named `secret-pod` with image `busybox:1.31.1` that keeps running.
- Existing Secret at `/opt/course/19/secret1.yaml`:
  - Create it in Namespace `secret`.
  - Mount it readonly into the Pod at `/tmp/secret1`.
- Create a new Secret in Namespace `secret` called `secret2` containing `user=user1` and `pass=1234`.
  - Expose these as environment variables `APP_USER` and `APP_PASS` in the Pod's container.
- Confirm everything is working.

**Solution:**

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c3-CCC
   ```

2. **Create the Namespace `secret` (if not exists):**

   ```bash
   kubectl create namespace secret
   ```

3. **Create the Pod `secret-pod` with Mounted Secret1:**

   - **Assuming `/opt/course/19/secret1.yaml` exists and defines a Secret named `secret1`.**

   - **Apply the Existing Secret:**

     ```bash
     kubectl apply -f /opt/course/19/secret1.yaml -n secret
     ```

   - **Create the Pod Manifest `secret-pod.yaml`:**

     ```yaml
     # secret-pod.yaml
     apiVersion: v1
     kind: Pod
     metadata:
       name: secret-pod
       namespace: secret
     spec:
       containers:
         - name: busybox
           image: busybox:1.31.1
           command: ["sleep", "3600"]
           volumeMounts:
             - name: secret1-volume
               mountPath: /tmp/secret1
               readOnly: true
       volumes:
         - name: secret1-volume
           secret:
             secretName: secret1
     ```

   - **Apply the Pod Manifest:**

     ```bash
     kubectl apply -f secret-pod.yaml
     ```

4. **Create the New Secret `secret2` with `user` and `pass`:**

   ```bash
   kubectl create secret generic secret2 --from-literal=user=user1 --from-literal=pass=1234 -n secret
   ```

5. **Update the Pod to Include Environment Variables from `secret2`:**

   **Option 1: Edit the Existing Pod (Not recommended for static Pods).**

   **Option 2: Recreate the Pod with Updated Manifest:**

   - **Delete Existing Pod:**

     ```bash
     kubectl delete pod secret-pod -n secret
     ```

   - **Create Updated Pod Manifest `secret-pod.yaml`:**

     ```yaml
     # secret-pod.yaml
     apiVersion: v1
     kind: Pod
     metadata:
       name: secret-pod
       namespace: secret
     spec:
       containers:
         - name: busybox
           image: busybox:1.31.1
           command: ["sleep", "3600"]
           env:
             - name: APP_USER
               valueFrom:
                 secretKeyRef:
                   name: secret2
                   key: user
             - name: APP_PASS
               valueFrom:
                 secretKeyRef:
                   name: secret2
                   key: pass
           volumeMounts:
             - name: secret1-volume
               mountPath: /tmp/secret1
               readOnly: true
       volumes:
         - name: secret1-volume
           secret:
             secretName: secret1
     ```

   - **Apply the Updated Pod Manifest:**

     ```bash
     kubectl apply -f secret-pod.yaml
     ```

6. **Verify the Pod is Running and Environment Variables are Set:**

   ```bash
   # Check Pod status
   kubectl get pod secret-pod -n secret

   # Execute a command in the Pod to print environment variables
   kubectl exec -n secret secret-pod -- env | grep APP_
   ```

   **Expected Output:**

   ```
   APP_USER=user1
   APP_PASS=1234
   ```

   **Verify Mounted Secret1:**

   ```bash
   kubectl exec -n secret secret-pod -- ls /tmp/secret1
   # Should list the keys in secret1
   ```

**Explanation:**

- **Secrets:**
  - **secret1:** Existing Secret, mounted as a volume.
  - **secret2:** New Secret, exposed as environment variables.
- **Pods:** Configured to mount Secrets and use them as environment variables.

**Full Commands and Manifests:**

```bash
# Q19: Set Kubernetes context
kubectl config use-context k8s-c3-CCC

# Q19: Create Namespace if not exists
kubectl create namespace secret

# Q19: Apply existing Secret1
kubectl apply -f /opt/course/19/secret1.yaml -n secret

# Q19: Create Pod manifest with Secret1 mounted
cat <<EOF | sudo tee secret-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
  namespace: secret
spec:
  containers:
    - name: busybox
      image: busybox:1.31.1
      command: ["sleep", "3600"]
      volumeMounts:
        - name: secret1-volume
          mountPath: /tmp/secret1
          readOnly: true
  volumes:
    - name: secret1-volume
      secret:
        secretName: secret1
EOF

# Q19: Apply the Pod manifest
kubectl apply -f secret-pod.yaml

# Q19: Create new Secret2
kubectl create secret generic secret2 --from-literal=user=user1 --from-literal=pass=1234 -n secret

# Q19: Delete existing Pod to recreate with environment variables
kubectl delete pod secret-pod -n secret

# Q19: Create updated Pod manifest with Secret2 as env vars
cat <<EOF | sudo tee secret-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
  namespace: secret
spec:
  containers:
    - name: busybox
      image: busybox:1.31.1
      command: ["sleep", "3600"]
      env:
        - name: APP_USER
          valueFrom:
            secretKeyRef:
              name: secret2
              key: user
        - name: APP_PASS
          valueFrom:
            secretKeyRef:
              name: secret2
              key: pass
      volumeMounts:
        - name: secret1-volume
          mountPath: /tmp/secret1
          readOnly: true
  volumes:
    - name: secret1-volume
      secret:
        secretName: secret1
EOF

# Q19: Apply the updated Pod manifest
kubectl apply -f secret-pod.yaml

# Q19: Verify Pod is running
kubectl get pod secret-pod -n secret

# Q19: Check environment variables
kubectl exec -n secret secret-pod -- env | grep APP_

# Q19: Verify mounted Secret1
kubectl exec -n secret secret-pod -- ls /tmp/secret1
```

---

### <a name="q20"></a>**Q20: Update and Join Node to Cluster**

**Task:**
- Use context: `k8s-c3-CCC`.
- Node `cluster2-node2` is running an older Kubernetes version and is not part of the cluster.
- Update Kubernetes on `cluster2-node2` to the exact version running on `cluster3-controlplane1`.
- Join `cluster2-node2` to the cluster using `kubeadm` with TLS bootstrapping.

**Solution:**

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c3-CCC
   ```

2. **Determine Kubernetes Version on `cluster3-controlplane1`:**

   ```bash
   kubectl get nodes -o wide
   ```

   **Sample Output:**

   ```
   NAME                    STATUS   ROLES                  AGE   VERSION
   cluster3-controlplane1   Ready    controlplane,master    10d   v1.31.0
   ```

   **Kubernetes Version:** `v1.31.0`

3. **Prepare `cluster2-node2`:**

   - **SSH into `cluster2-node2`:**

     ```bash
     ssh user@cluster2-node2
     ```

   - **Check Current Kubernetes Version:**

     ```bash
     kubectl version --short
     ```

   - **Update Kubernetes Components:**

     Assuming `kubeadm`, `kubelet`, and `kubectl` are installed via package manager (e.g., `apt`).

     ```bash
     sudo apt-get update
     sudo apt-get install -y kubeadm=1.31.0-00 kubelet=1.31.0-00 kubectl=1.31.0-00
     sudo apt-mark hold kubeadm kubelet kubectl
     ```

     **Explanation:**
     - Installs the exact version `v1.31.0`.
     - Holds the packages to prevent automatic updates.

4. **Retrieve the `kubeadm join` Command:**

   On the controlplane node (`cluster3-controlplane1`):

   ```bash
   ssh user@cluster3-controlplane1
   kubeadm token create --print-join-command
   ```

   **Sample Output:**

   ```
   kubeadm join 192.168.1.100:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:...
   ```

   **Copy the Join Command.**

5. **Join `cluster2-node2` to the Cluster:**

   On `cluster2-node2`:

   ```bash
   sudo kubeadm join 192.168.1.100:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:...
   ```

   **Explanation:**
   - This command connects the node to the cluster, bootstrapping it with TLS.

6. **Verify Node is Joined and Ready:**

   Back on the main terminal:

   ```bash
   kubectl get nodes
   ```

   **Expected Output:**

   ```
   NAME                    STATUS   ROLES                  AGE   VERSION
   cluster3-controlplane1   Ready    controlplane,master    10d   v1.31.0
   cluster2-node1           Ready    <none>                 5d    v1.31.0
   cluster2-node2           Ready    <none>                 1m    v1.31.0
   ```

7. **Schedule a Pod on `cluster2-node2`:**

   Create a Pod specifying `nodeName`:

   ```yaml
   # test-pod-cluster2-node2.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: test-pod-cluster2-node2
     namespace: default
   spec:
     containers:
       - name: nginx
         image: nginx:1.17.6-alpine
     nodeName: cluster2-node2
   ```

   Apply the Pod:

   ```bash
   kubectl apply -f test-pod-cluster2-node2.yaml
   ```

   **Verify Pod is Running on `cluster2-node2`:**

   ```bash
   kubectl get pod test-pod-cluster2-node2 -n default -o wide
   ```

   **Expected Output:**

   ```
   NAME                           READY   STATUS    RESTARTS   AGE   IP           NODE               NOMINATED NODE   READINESS GATES
   test-pod-cluster2-node2        1/1     Running   0          1m    10.244.4.5   cluster2-node2     <none>           <none>
   ```

8. **Exit `cluster2-node2`:**

   ```bash
   exit
   ```

**Explanation:**

- Ensures that `cluster2-node2` runs the same Kubernetes version as the controlplane node.
- Uses `kubeadm` to join the node with TLS bootstrapping.
- Verifies the node is part of the cluster and schedules Pods on it.

**Full Commands:**

```bash
# Q20: Set Kubernetes context
kubectl config use-context k8s-c3-CCC

# Q20: Determine Kubernetes version on controlplane
kubectl get nodes -o wide

# Q20: SSH into cluster2-node2
ssh user@cluster2-node2

# Q20: Update Kubernetes components to match controlplane version (replace with actual version)
sudo apt-get update
sudo apt-get install -y kubeadm=1.31.0-00 kubelet=1.31.0-00 kubectl=1.31.0-00
sudo apt-mark hold kubeadm kubelet kubectl

# Q20: Retrieve kubeadm join command from controlplane
# SSH into controlplane node
ssh user@cluster3-controlplane1
kubeadm token create --print-join-command
# Copy the join command

# Q20: Join the node to the cluster (on cluster2-node2)
sudo kubeadm join 192.168.1.100:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:...

# Q20: Verify node is joined and Ready
exit
kubectl get nodes

# Q20: Create Pod scheduled on cluster2-node2
cat <<EOF | sudo tee test-pod-cluster2-node2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod-cluster2-node2
  namespace: default
spec:
  containers:
    - name: nginx
      image: nginx:1.17.6-alpine
  nodeName: cluster2-node2
EOF

# Q20: Apply the Pod manifest
kubectl apply -f test-pod-cluster2-node2.yaml

# Q20: Verify Pod is running on cluster2-node2
kubectl get pod test-pod-cluster2-node2 -n default -o wide
```

---

### <a name="q21"></a>**Q21: Backup and Expose Static Pod via NodePort Service**

**Task:**
- Use context: `k8s-c3-CCC`.
- Create a Static Pod named `my-static-pod` in Namespace `default` on `cluster3-controlplane1`:
  - Image: `nginx:1.16-alpine`.
  - Resource Requests: `10m` CPU and `20Mi` memory.
- Create a NodePort Service named `static-pod-service`:
  - Exposes the Static Pod on port `80`.
  - Assigns NodePort `30080`.
- Check if the Service has Endpoints.
- Confirm it's reachable through the internal IP of `cluster3-controlplane1`.

**Solution:**

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c3-CCC
   ```

2. **SSH into `cluster3-controlplane1`:**

   ```bash
   ssh user@cluster3-controlplane1
   ```

3. **Create the Static Pod Manifest `my-static-pod.yaml`:**

   ```yaml
   # my-static-pod.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: my-static-pod
     namespace: default
     labels:
       app: my-static-pod
   spec:
     containers:
       - name: nginx
         image: nginx:1.16-alpine
         resources:
           requests:
             cpu: "10m"
             memory: "20Mi"
         ports:
           - containerPort: 80
   ```

4. **Deploy the Static Pod:**

   ```bash
   sudo mv my-static-pod.yaml /etc/kubernetes/manifests/
   ```

   **Explanation:**
   - Static Pods are defined in `/etc/kubernetes/manifests/`.
   - kubelet automatically creates the Pod from the manifest.

5. **Verify the Static Pod is Running:**

   ```bash
   kubectl get pod my-static-pod -n default -o wide
   ```

   **Expected Output:**

   ```
   NAME            READY   STATUS    RESTARTS   AGE   IP           NODE                   NOMINATED NODE   READINESS GATES
   my-static-pod    1/1     Running   0          2m    10.244.x.x   cluster3-controlplane1   <none>           <none>
   ```

6. **Create the NodePort Service `static-pod-service.yaml`:**

   ```yaml
   # static-pod-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: static-pod-service
     namespace: default
   spec:
     type: NodePort
     selector:
       app: my-static-pod
     ports:
       - protocol: TCP
         port: 80
         targetPort: 80
         nodePort: 30080
   ```

7. **Apply the Service Manifest:**

   ```bash
   kubectl apply -f static-pod-service.yaml
   ```

8. **Verify the Service and Endpoints:**

   ```bash
   kubectl get service static-pod-service -n default
   kubectl get endpoints static-pod-service -n default
   ```

   **Expected Output:**

   ```
   kubectl get service static-pod-service -n default
   NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
   static-pod-service    NodePort    10.96.0.1      <none>        80:30080/TCP   2m
   ```

   ```
   kubectl get endpoints static-pod-service -n default
   NAME                 ENDPOINTS             AGE
   static-pod-service    10.244.x.x:80        2m
   ```

9. **Determine Internal IP of `cluster3-controlplane1`:**

   ```bash
   kubectl get node cluster3-controlplane1 -o wide
   ```

   **Sample Output:**

   ```
   NAME                    STATUS   ROLES                  AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
   cluster3-controlplane1   Ready    controlplane,master    10d   v1.31.0   192.168.1.100   <none>        Ubuntu 20.04.5 LTS   5.4.0-104-generic   containerd://1.6.0
   ```

   **Internal IP:** `192.168.1.100`

10. **Test Service Connectivity from Main Terminal:**

    ```bash
    curl http://192.168.1.100:30080
    ```

    **Expected Output:**

    ```
    <!DOCTYPE html>
    <html>
    ...
    </html>
    ```

    **Explanation:**
    - The `curl` command should return the default Nginx welcome page, indicating successful connectivity.

11. **Exit the Node:**

    ```bash
    exit
    ```

**Explanation:**

- **Static Pod:** Managed by kubelet via manifests; not controlled by Deployments or ReplicaSets.
- **Service:** Exposes the Static Pod via NodePort, making it accessible externally via the node's IP and assigned port.
- **Endpoints:** Should reflect the Static Pod's IP and port.

**Full Commands and Manifests:**

```bash
# Q21: Set Kubernetes context
kubectl config use-context k8s-c3-CCC

# Q21: SSH into controlplane node
ssh user@cluster3-controlplane1

# Q21: Create Static Pod manifest
cat <<EOF | sudo tee my-static-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-static-pod
  namespace: default
  labels:
    app: my-static-pod
spec:
  containers:
    - name: nginx
      image: nginx:1.16-alpine
      resources:
        requests:
          cpu: "10m"
          memory: "20Mi"
      ports:
        - containerPort: 80
EOF

# Q21: Deploy the Static Pod by moving it to manifests directory
sudo mv my-static-pod.yaml /etc/kubernetes/manifests/

# Q21: Verify the Pod is running
kubectl get pod my-static-pod -n default -o wide

# Q21: Create Service manifest
cat <<EOF | sudo tee static-pod-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: static-pod-service
  namespace: default
spec:
  type: NodePort
  selector:
    app: my-static-pod
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
EOF

# Q21: Apply the Service
kubectl apply -f static-pod-service.yaml

# Q21: Verify the Service and Endpoints
kubectl get service static-pod-service -n default
kubectl get endpoints static-pod-service -n default

# Q21: Get internal IP of controlplane node
kubectl get node cluster3-controlplane1 -o wide

# Q21: Test Service connectivity
curl http://192.168.1.100:30080

# Q21: Exit the node
exit
```

---

### <a name="q22"></a>**Q22: Check and Renew kube-apiserver Certificate**

**Task:**
- Use context: `k8s-c2-AC`.
- On `cluster2-controlplane1`, check the expiration date of the `kube-apiserver` server certificate using `openssl` or `cfssl`.
- Write the expiration date into `/opt/course/22/expiration`.
- Run the correct `kubeadm` command to list the expiration dates and confirm both methods show the same date.
- Write the `kubeadm` command that would renew the `kube-apiserver` certificate into `/opt/course/22/kubeadm-renew-certs.sh`.

**Solution:**

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c2-AC
   ```

2. **SSH into `cluster2-controlplane1`:**

   ```bash
   ssh user@cluster2-controlplane1
   ```

3. **Check Certificate Expiration Using OpenSSL:**

   ```bash
   # Extract expiration date
   openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -enddate | awk -F= '{print $2}' | sudo tee /opt/course/22/expiration
   ```

   **Sample Output in `expiration`:**

   ```
   Dec 31 23:59:59 2024 GMT
   ```

4. **Check Certificate Expiration Using kubeadm:**

   ```bash
   sudo kubeadm certs check-expiration | grep apiserver.crt | awk '{print $2 " " $3 ", " $4}' | sudo tee -a /opt/course/22/expiration
   ```

   **Sample Output in `expiration`:**

   ```
   Dec 31, 2024
   ```

   **Note:** The date formats differ slightly.

5. **Compare Both Methods:**

   - **View the File:**

     ```bash
     cat /opt/course/22/expiration
     ```

     **Sample Content:**

     ```
     Dec 31 23:59:59 2024 GMT
     Dec 31, 2024
     ```

   - **Manual Comparison:** The dates match, albeit formatted differently.

6. **Create the Script `kubeadm-renew-certs.sh`:**

   ```bash
   #!/bin/bash

   # Renew the kube-apiserver certificate
   sudo kubeadm certs renew apiserver

   # Check certificate expiration after renewal
   sudo kubeadm certs check-expiration

   # Restart kubelet to apply the new certificate
   sudo systemctl restart kubelet

   # Verify kube-apiserver Pod is running with the new certificate
   kubectl get pods -n kube-system -l component=kube-apiserver
   ```

7. **Write the Script:**

   ```bash
   cat <<EOF | sudo tee /opt/course/22/kubeadm-renew-certs.sh
#!/bin/bash

# Renew the kube-apiserver certificate
sudo kubeadm certs renew apiserver

# Check certificate expiration after renewal
sudo kubeadm certs check-expiration

# Restart kubelet to apply the new certificate
sudo systemctl restart kubelet

# Verify kube-apiserver Pod is running with the new certificate
kubectl get pods -n kube-system -l component=kube-apiserver
EOF

   # Make the script executable
   sudo chmod +x /opt/course/22/kubeadm-renew-certs.sh
   ```

8. **Verify the Script Content:**

   ```bash
   cat /opt/course/22/kubeadm-renew-certs.sh
   ```

   **Expected Content:**

   ```bash
   #!/bin/bash

   # Renew the kube-apiserver certificate
   sudo kubeadm certs renew apiserver

   # Check certificate expiration after renewal
   sudo kubeadm certs check-expiration

   # Restart kubelet to apply the new certificate
   sudo systemctl restart kubelet

   # Verify kube-apiserver Pod is running with the new certificate
   kubectl get pods -n kube-system -l component=kube-apiserver
   ```

**Explanation:**

- **OpenSSL:** Directly inspects the certificate file.
- **kubeadm:** Provides a high-level overview of certificate statuses.
- **Script:** Automates the renewal process and verification.

**Full Commands and Scripts:**

```bash
# Q22: Set Kubernetes context
kubectl config use-context k8s-c2-AC

# Q22: SSH into controlplane node
ssh user@cluster2-controlplane1

# Q22: Check certificate expiration using OpenSSL
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -enddate | awk -F= '{print $2}' | sudo tee /opt/course/22/expiration

# Q22: Check certificate expiration using kubeadm and append
sudo kubeadm certs check-expiration | grep apiserver.crt | awk '{print $2 " " $3 ", " $4}' | sudo tee -a /opt/course/22/expiration

# Q22: View expiration file
cat /opt/course/22/expiration

# Q22: Create kubeadm-renew-certs.sh
cat <<EOF | sudo tee /opt/course/22/kubeadm-renew-certs.sh
#!/bin/bash

# Renew the kube-apiserver certificate
sudo kubeadm certs renew apiserver

# Check certificate expiration after renewal
sudo kubeadm certs check-expiration

# Restart kubelet to apply the new certificate
sudo systemctl restart kubelet

# Verify kube-apiserver Pod is running with the new certificate
kubectl get pods -n kube-system -l component=kube-apiserver
EOF

# Q22: Make the script executable
sudo chmod +x /opt/course/22/kubeadm-renew-certs.sh

# Q22: Verify the script content
cat /opt/course/22/kubeadm-renew-certs.sh
```

---

### <a name="q23"></a>**Q23: Inspect Kubelet Certificates**

**Task:**
- Use context: `k8s-c2-AC`.
- Node: `cluster2-node1`.
- Find "Issuer" and "Extended Key Usage" values for:
  1. Kubelet Client Certificate (outgoing connections).
  2. Kubelet Server Certificate (incoming connections).
- Write the information into `/opt/course/23/certificate-info.txt`.
- Compare the fields and explain their significance.

**Solution:**

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c2-AC
   ```

2. **SSH into `cluster2-node1`:**

   ```bash
   ssh user@cluster2-node1
   ```

3. **Locate Kubelet Certificates:**

   - **Client Certificate:**
     
     Path: `/var/lib/kubelet/pki/kubelet-client-current.pem`
   
   - **Server Certificate:**
     
     Path: `/var/lib/kubelet/pki/kubelet-server-current.pem`

4. **Extract "Issuer" and "Extended Key Usage" Using OpenSSL:**

   ```bash
   # Create the output file
   echo "# /opt/course/23/certificate-info.txt" | sudo tee /opt/course/23/certificate-info.txt

   # Kubelet Client Certificate
   echo "Kubelet Client Certificate:" | sudo tee -a /opt/course/23/certificate-info.txt
   openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -noout -issuer -ext extendedKeyUsage | sudo tee -a /opt/course/23/certificate-info.txt
   echo "" | sudo tee -a /opt/course/23/certificate-info.txt

   # Kubelet Server Certificate
   echo "Kubelet Server Certificate:" | sudo tee -a /opt/course/23/certificate-info.txt
   openssl x509 -in /var/lib/kubelet/pki/kubelet-server-current.pem -noout -issuer -ext extendedKeyUsage | sudo tee -a /opt/course/23/certificate-info.txt
   ```

   **Sample Content of `/opt/course/23/certificate-info.txt`:**

   ```
   # /opt/course/23/certificate-info.txt
   Kubelet Client Certificate:
   issuer=CN=kubernetes
   X509v3 Extended Key Usage:
       TLS Web Client Authentication, TLS Web Server Authentication

   Kubelet Server Certificate:
   issuer=CN=kubernetes
   X509v3 Extended Key Usage:
       TLS Web Server Authentication
   ```

5. **Compare the Fields and Explain:**

   - **Issuer:**
     - Both certificates are issued by the Kubernetes CA (`CN=kubernetes`), ensuring trust within the cluster.
   
   - **Extended Key Usage (EKU):**
     - **Client Certificate:**
       - `TLS Web Client Authentication`: Allows the kubelet to authenticate to the API server.
       - `TLS Web Server Authentication`: Allows the kubelet to also act as a server, if needed.
     - **Server Certificate:**
       - `TLS Web Server Authentication`: Allows the kubelet to serve HTTPS requests, typically from the API server.

   **Significance:**
   - **Client Certificate:** Grants the kubelet the ability to authenticate itself when communicating with the API server.
   - **Server Certificate:** Enables the kubelet to securely serve endpoints to the API server or other components.
   - **Issuer Consistency:** Ensures that all certificates within the cluster are trusted and signed by the same authority, maintaining cluster security.

6. **Exit the Node:**

   ```bash
   exit
   ```

**Explanation:**

- **Issuer:** Both certificates share the same CA, ensuring mutual trust.
- **EKU:** Differentiates the role of each certificate—client vs. server authentication.

**Full Commands and Script:**

```bash
# Q23: Set Kubernetes context
kubectl config use-context k8s-c2-AC

# Q23: SSH into node
ssh user@cluster2-node1

# Q23: Create the certificate-info.txt file with headers
echo "# /opt/course/23/certificate-info.txt" | sudo tee /opt/course/23/certificate-info.txt

# Q23: Extract info from kubelet-client certificate
echo "Kubelet Client Certificate:" | sudo tee -a /opt/course/23/certificate-info.txt
openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -noout -issuer -ext extendedKeyUsage | sudo tee -a /opt/course/23/certificate-info.txt
echo "" | sudo tee -a /opt/course/23/certificate-info.txt

# Q23: Extract info from kubelet-server certificate
echo "Kubelet Server Certificate:" | sudo tee -a /opt/course/23/certificate-info.txt
openssl x509 -in /var/lib/kubelet/pki/kubelet-server-current.pem -noout -issuer -ext extendedKeyUsage | sudo tee -a /opt/course/23/certificate-info.txt

# Q23: Verify the file content
cat /opt/course/23/certificate-info.txt

# Q23: Compare and analyze the fields manually
# (Explanation provided above)

# Q23: Exit the node
exit
```

---

### <a name="q24"></a>**Q24: Create NetworkPolicy to Restrict Pod Communication**

**Task:**
- Use context: `k8s-c1-H`.
- Security incident: Intruder accessed the cluster via a hacked `backend-*` Pod.
- Create a NetworkPolicy named `np-backend` in Namespace `project-snake` to allow `backend-*` Pods only to:
  - Connect to `db1-*` Pods on port `1111`.
  - Connect to `db2-*` Pods on port `2222`.
- Use the `app` label of Pods in the policy.
- After implementation, connections from `backend-*` Pods to `vault-*` Pods on port `3333` should be denied.

**Solution:**

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c1-H
   ```

2. **Ensure the Namespace `project-snake` Exists:**

   ```bash
   kubectl get namespace project-snake || kubectl create namespace project-snake
   ```

3. **Label the Pods Appropriately:**

   - **Label `backend-*` Pods:**

     ```bash
     kubectl label pods -n project-snake -l app=backend- role=backend --overwrite
     ```

     **Note:** Kubernetes does not support wildcard labels directly. Replace `app=backend-` with specific labels or use multiple label selectors.

     **Alternative: Label each `backend-*` Pod individually:**

     ```bash
     kubectl label pod -n project-snake backend-pod1 app=backend role=backend --overwrite
     kubectl label pod -n project-snake backend-pod2 app=backend role=backend --overwrite
     # Repeat for all backend Pods
     ```

   - **Label `db1-*` Pods:**

     ```bash
     kubectl label pod -n project-snake db1-pod1 app=db1 role=db1 --overwrite
     kubectl label pod -n project-snake db1-pod2 app=db1 role=db1 --overwrite
     # Repeat for all db1 Pods
     ```

   - **Label `db2-*` Pods:**

     ```bash
     kubectl label pod -n project-snake db2-pod1 app=db2 role=db2 --overwrite
     kubectl label pod -n project-snake db2-pod2 app=db2 role=db2 --overwrite
     # Repeat for all db2 Pods
     ```

   - **Label `vault-*` Pods (Optional for Verification):**

     ```bash
     kubectl label pod -n project-snake vault-pod1 app=vault role=vault --overwrite
     kubectl label pod -n project-snake vault-pod2 app=vault role=vault --overwrite
     # Repeat for all vault Pods
     ```

4. **Create the NetworkPolicy `np-backend.yaml`:**

   ```yaml
   # np-backend.yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: np-backend
     namespace: project-snake
   spec:
     podSelector:
       matchLabels:
         role: backend
     policyTypes:
       - Egress
     egress:
       - to:
           - podSelector:
               matchLabels:
                 role: db1
         ports:
           - protocol: TCP
             port: 1111
       - to:
           - podSelector:
               matchLabels:
                 role: db2
         ports:
           - protocol: TCP
             port: 2222
   ```

   **Explanation:**
   - **podSelector:** Targets Pods with `role=backend`.
   - **policyTypes:** Specifies that this policy controls `Egress` traffic.
   - **egress Rules:**
     - Allow traffic to Pods with `role=db1` on port `1111`.
     - Allow traffic to Pods with `role=db2` on port `2222`.
   - **Implicit Deny:** All other egress traffic is denied, including to `vault-*` Pods on port `3333`.

5. **Apply the NetworkPolicy:**

   ```bash
   kubectl apply -f np-backend.yaml
   ```

6. **Verify the NetworkPolicy:**

   ```bash
   kubectl describe networkpolicy np-backend -n project-snake
   ```

   **Expected Output:**

   ```
   Name:         np-backend
   Namespace:    project-snake
   ...
   Spec:
     PodSelector:
       role=backend
     PolicyTypes:
       Egress
     Egress:
       To:
         PodSelector:
           role=db1
         Ports:
           TCP  1111
       To:
         PodSelector:
           role=db2
         Ports:
           TCP  2222
   ```

7. **Test Connectivity:**

   - **Create a Temporary Pod for Testing:**

     ```bash
     kubectl run -n project-snake -i --tty test-pod --image=busybox --restart=Never -- sh
     ```

   - **Within the Temporary Pod:**

     ```sh
     # Attempt to connect to db1-pod on port 1111 (should succeed)
     wget -qO- http://db1-pod1:1111

     # Attempt to connect to db2-pod on port 2222 (should succeed)
     wget -qO- http://db2-pod1:2222

     # Attempt to connect to vault-pod on port 3333 (should fail)
     wget -qO- http://vault-pod1:3333

     # Exit the Pod
     exit
     ```

   **Expected Outcomes:**
   - Connections to `db1-pod1:1111` and `db2-pod1:2222` should succeed.
   - Connection to `vault-pod1:3333` should fail (no response or timeout).

**Explanation:**

- **NetworkPolicy `np-backend`:** Restricts outbound traffic from `backend-*` Pods to only specified Pods and ports.
- **Implicit Deny:** Kubernetes NetworkPolicies are **default deny** for specified traffic types once policies are applied.

**Full Commands and Manifest:**

```bash
# Q24: Set Kubernetes context
kubectl config use-context k8s-c1-H

# Q24: Ensure Namespace exists
kubectl get namespace project-snake || kubectl create namespace project-snake

# Q24: Label backend Pods
kubectl label pod -n project-snake backend-pod1 app=backend role=backend --overwrite
kubectl label pod -n project-snake backend-pod2 app=backend role=backend --overwrite
# Repeat for all backend Pods

# Q24: Label db1 Pods
kubectl label pod -n project-snake db1-pod1 app=db1 role=db1 --overwrite
kubectl label pod -n project-snake db1-pod2 app=db1 role=db1 --overwrite
# Repeat for all db1 Pods

# Q24: Label db2 Pods
kubectl label pod -n project-snake db2-pod1 app=db2 role=db2 --overwrite
kubectl label pod -n project-snake db2-pod2 app=db2 role=db2 --overwrite
# Repeat for all db2 Pods

# Q24: Label vault Pods
kubectl label pod -n project-snake vault-pod1 app=vault role=vault --overwrite
kubectl label pod -n project-snake vault-pod2 app=vault role=vault --overwrite
# Repeat for all vault Pods

# Q24: Create NetworkPolicy manifest
cat <<EOF | sudo tee np-backend.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-backend
  namespace: project-snake
spec:
  podSelector:
    matchLabels:
      role: backend
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              role: db1
      ports:
        - protocol: TCP
          port: 1111
    - to:
        - podSelector:
            matchLabels:
              role: db2
      ports:
        - protocol: TCP
          port: 2222
EOF

# Q24: Apply the NetworkPolicy
kubectl apply -f np-backend.yaml

# Q24: Verify the NetworkPolicy
kubectl describe networkpolicy np-backend -n project-snake

# Q24: Create a temporary Pod for testing
kubectl run -n project-snake -i --tty test-pod --image=busybox --restart=Never -- sh

# Q24: Within the temporary Pod, perform connectivity tests
# (As detailed in the solution)

# Q24: Exit the temporary Pod
exit
```

---

### <a name="q25"></a>**Q25: Backup and Restore etcd and Verify Cluster State**

**Task:**
- Use context: `k8s-c3-CCC`.
- Make a backup of etcd running on `cluster3-controlplane1` and save it at `/tmp/etcd-backup.db`.
- Create any kind of Pod in the cluster.
- Restore the backup.
- Confirm the cluster is still working and the created Pod is no longer present.

**Solution:**

1. **Set the Kubernetes Context:**

   ```bash
   kubectl config use-context k8s-c3-CCC
   ```

2. **SSH into `cluster3-controlplane1`:**

   ```bash
   ssh user@cluster3-controlplane1
   ```

3. **Ensure `etcdctl` is Installed:**

   ```bash
   etcdctl version
   ```

   **If Not Installed:**

   ```bash
   VERSION="v1.25.0"
   wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
   sudo tar -C /usr/local/bin -xzf crictl-$VERSION-linux-amd64.tar.gz
   rm crictl-$VERSION-linux-amd64.tar.gz
   ```

4. **Set Environment Variables for `etcdctl`:**

   ```bash
   export ETCDCTL_API=3
   export ETCDCTL_CERT=/etc/kubernetes/pki/etcd/healthcheck-client.crt
   export ETCDCTL_KEY=/etc/kubernetes/pki/etcd/healthcheck-client.key
   export ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
   export ETCDCTL_ENDPOINTS=https://127.0.0.1:2379
   ```

5. **Create etcd Backup:**

   ```bash
   etcdctl snapshot save /tmp/etcd-backup.db
   ```

6. **Verify the Backup:**

   ```bash
   ls -lh /tmp/etcd-backup.db
   etcdctl snapshot status /tmp/etcd-backup.db
   ```

7. **Create a Test Pod:**

   ```yaml
   # test-pod.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: test-pod
     namespace: default
   spec:
     containers:
       - name: nginx
         image: nginx:1.17.6-alpine
   ```

   Apply the Pod:

   ```bash
   kubectl apply -f test-pod.yaml
   ```

   **Verify Pod is Running:**

   ```bash
   kubectl get pod test-pod -n default -o wide
   ```

8. **Restore etcd from Backup:**

   **Important:** Restoring etcd will revert the cluster state to the backup, removing any resources created after the backup (e.g., `test-pod`).

   **Steps:**

   - **Stop kube-apiserver Pod:**

     ```bash
     kubectl delete pod -n kube-system kube-apiserver-cluster3-controlplane1
     ```

   - **Stop etcd by Renaming the Manifest:**

     ```bash
     sudo mv /etc/kubernetes/manifests/etcd.yaml /etc/kubernetes/manifests/etcd.yaml.bak
     ```

   - **Restore the Snapshot:**

     ```bash
     etcdctl snapshot restore /tmp/etcd-backup.db --data-dir /var/lib/etcd
     ```

   - **Reapply etcd Manifest:**

     ```bash
     sudo mv /etc/kubernetes/manifests/etcd.yaml.bak /etc/kubernetes/manifests/etcd.yaml
     ```

     Kubelet will detect the manifest and recreate the etcd Pod.

9. **Verify etcd is Running:**

   ```bash
   kubectl get pods -n kube-system | grep etcd
   ```

   **Expected Output:**

   ```
   etcd-cluster3-controlplane1   1/1     Running   0          5m
   ```

10. **Verify Cluster State and Pod Deletion:**

    ```bash
    kubectl get pods -n default
    ```

    **Expected Output:**

    ```
    NAME       READY   STATUS    RESTARTS   AGE
    # 'test-pod' should not be listed
    existing-pod1   1/1     Running   0          10d
    existing-pod2   1/1     Running   0          10d
    ```

11. **Confirm Cluster is Operational by Creating Another Pod:**

    ```yaml
    # verify-pod.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: verify-pod
      namespace: default
    spec:
      containers:
        - name: nginx
          image: nginx:1.17.6-alpine
    ```

    Apply the Pod:

    ```bash
    kubectl apply -f verify-pod.yaml
    ```

    **Verify Pod is Running:**

    ```bash
    kubectl get pod verify-pod -n default -o wide
    ```

    **Expected Output:**

    ```
    NAME         READY   STATUS    RESTARTS   AGE   IP           NODE                   NOMINATED NODE   READINESS GATES
    verify-pod    1/1     Running   0          1m    10.244.x.x   cluster3-controlplane1   <none>           <none>
    ```

    **Note:** The previously created `test-pod` is no longer present, confirming the restoration.

12. **Exit the Node:**

    ```bash
    exit
    ```

**Explanation:**

- **Backup:** Captures the current state of etcd.
- **Pod Creation:** Adds a resource to the cluster.
- **Restoration:** Reverts the cluster to the backup state, removing the new Pod.
- **Verification:** Ensures the cluster functions correctly post-restoration.

**Full Commands and Manifests:**

```bash
# Q25: Set Kubernetes context
kubectl config use-context k8s-c3-CCC

# Q25: SSH into controlplane node
ssh user@cluster3-controlplane1

# Q25: Set environment variables for etcdctl
export ETCDCTL_API=3
export ETCDCTL_CERT=/etc/kubernetes/pki/etcd/healthcheck-client.crt
export ETCDCTL_KEY=/etc/kubernetes/pki/etcd/healthcheck-client.key
export ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
export ETCDCTL_ENDPOINTS=https://127.0.0.1:2379

# Q25: Create etcd backup
etcdctl snapshot save /tmp/etcd-backup.db

# Q25: Verify the backup
ls -lh /tmp/etcd-backup.db
etcdctl snapshot status /tmp/etcd-backup.db

# Q25: Create and apply a test Pod
cat <<EOF | sudo tee test-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  namespace: default
spec:
  containers:
    - name: nginx
      image: nginx:1.17.6-alpine
EOF

kubectl apply -f test-pod.yaml

# Q25: Verify the Pod is running
kubectl get pod test-pod -n default -o wide

# Q25: Restore etcd from backup
# Stop kube-apiserver Pod
kubectl delete pod -n kube-system kube-apiserver-cluster3-controlplane1

# Stop etcd by renaming the manifest
sudo mv /etc/kubernetes/manifests/etcd.yaml /etc/kubernetes/manifests/etcd.yaml.bak

# Restore the etcd snapshot
etcdctl snapshot restore /tmp/etcd-backup.db --data-dir /var/lib/etcd

# Reapply the etcd manifest to restart etcd
sudo mv /etc/kubernetes/manifests/etcd.yaml.bak /etc/kubernetes/manifests/etcd.yaml

# Q25: Verify etcd is running
kubectl get pods -n kube-system | grep etcd

# Q25: Verify cluster state and Pod deletion
kubectl get pods -n default

# Q25: Create another Pod to confirm cluster functionality
cat <<EOF | sudo tee verify-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: verify-pod
  namespace: default
spec:
  containers:
    - name: nginx
      image: nginx:1.17.6-alpine
EOF

kubectl apply -f verify-pod.yaml

# Q25: Verify the new Pod is running
kubectl get pod verify-pod -n default -o wide

# Q25: Confirm the `test-pod` no longer exists
kubectl get pod test-pod -n default

# Q25: Exit the node
exit
```

---

## **Final Notes**

- **Permissions:** Ensure that the user executing these commands has the necessary permissions, especially when writing to `/opt/course/` directories.
- **Cluster Access:** SSH access to controlplane nodes is crucial for operations involving `etcd` and static Pods.
- **Backup Caution:** Restoring etcd can lead to data loss. Always ensure that backups are recent and stored securely.
- **NetworkPolicies:** Ensure that your cluster's network plugin supports NetworkPolicies (e.g., Calico, Cilium).
- **Kubernetes Version:** Adjust commands if using a version significantly different from `v1.31.0`, as some APIs and commands may have changed.

**Good Luck with Your CKA Preparation!** If you have any further questions or need additional assistance, feel free to ask.
