## Step 1: Spin up the Cluster

Ensure you have `kind` and `kubectl` installed, then create the cluster using the provided configuration file from the root directory:

```
kind create cluster --config cluster.yml
```
Verify that the cluster nodes are up and running:

```
kubectl get nodes
```

## Step 2: Apply the Infrastructure and RBAC Manifests
First, create the required namespace (if not automated) and deploy the application along with its dependencies:

```
kubectl apply -f .infrastructure/ ...
```
Apply the RBAC configuration located inside the security directory:

```
kubectl apply -f .infrastructure/security/rbac.yml
```
Verify that the ServiceAccount, Role, and RoleBinding were created successfully:

```
kubectl get sa,role,rolebinding -n todoapp
```
## Step 3: Validate the ServiceAccount on the Deployment
Check the deployment description to ensure that the pods are explicitly configured to use the secrets-reader ServiceAccount:

```
kubectl describe deployment todoapp -n todoapp | grep "Service Account"
```
Expected output line: Service Account:  secrets-reader

## Step 4: Execute Curl Command from the Pod to List Secrets
To prove the RBAC permissions work natively inside the container, we need to access the pod's shell and call the Kubernetes API directly using the service account token.

Get the exact name of the running application pod:

```
kubectl get pods -n todoapp
```
Exec into the pod (replace <pod-name> with your actual pod name):

```
kubectl exec <pod-name> -it -n todoapp -- sh
```
Inside the pod's container terminal, copy and paste the following block of commands to set up variables and execute the API call:


### Define service account credentials paths
```
SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount
APISERVER=[https://kubernetes.default.svc](https://kubernetes.default.svc)
TOKEN=$(cat ${SERVICEACCOUNT}/token)
CACERT=${SERVICEACCOUNT}/ca.crt
```

### Execute the curl command to list secrets
```
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/todoapp/secrets
```
