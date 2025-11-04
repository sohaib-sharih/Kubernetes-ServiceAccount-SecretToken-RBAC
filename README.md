### Project: Creating a Service Account and associating it to a POD using RBAC

1. **What is a service account?** *SA are used for internally accessing k8s resources via the apiserver. It is for robots or the system to access other resources.*
2. **The purpose of this project:** We need to check what is the purpose of creating a service account and how can it be helpful in providing ***security*** to our deployed ***microservice.***
3. **Flow of our project:**
	a. Create a **Service Account**
	b. Create a **Secret** and Link it to the Service Account to generate a ***token*** that persists.
	c. **Note:** Generating the token is important, otherwise the one that k8s auto-generates only lasts for an hour or are ***shortlived.***
	d. Check the **token** generated
	e. Create a **role**
	f. Create a **role-binding**
	g. Create a **pod** using the ***bitnami/kubectl*** image because it provides a kubectl cli to send requests to other pods.
4. **Configuration:**

```
kubectl create sa exam
---------------
kubectl create secret generic exam-token --dry-run=client -o yaml > secret.yaml

Edit secret.yaml

apiVersion: v1
kind: Secret
metadata:
  name: exam-token
  annotations:
    kubernetes.io/service-account.name: exam
type: kubernetes.io/service-account-token
------------------

vi examRole.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
----------------------

vi examRoleBind.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
subjects:
- kind: ServiceAccount
  name: exam
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io

--------------------

vi examPod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: sa-test
spec:
  serviceAccountName: exam
  containers:
  - name: busy
    image: bitnami/kubectl
    command: ["sleep", "3600"]

------------------

Test the ability of the serviceAccount to send requests to the kube-apiserver

1. kubectl exec -it sa-exam -- sh
2. Run - > kubectl get pods, kubectl get depoyments
```

5. A **ServiceAccount** can be used by:
	a. **Pods** (most common) — to access the API server.
    b. **Jobs / CronJobs** — for tasks needing limited API access.
    c. **Deployments / ReplicaSets / DaemonSets / StatefulSets** — their pods inherit the ServiceAccount.
    d. **Controllers** (e.g., custom controllers, operators) — to manage cluster resources securely.
