# Create user demouser with role permission developer only on the pod
#Generate the ssl certificate for demoUser
openssl genrsa -out developer.key 2048
openssl req -new -key developer.key -out developer.csr -subj "/CN=developer"

#Submit the CertificateSigningRequest to the API Server
#Key elements, name, request and usages (must be client auth)
cat <<EOF > csr_template.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: developer-csr
spec:
  request: <Base64_encoded_CSR>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF

Generate the CSR content in Base64 and create the YAML file:
CSR_CONTENT=$(cat developer.csr | base64 | tr -d '\n')
sed "s|<Base64_encoded_CSR>|$CSR_CONTENT|" csr_template.yaml > developer_csr.yaml

kubectl apply -f developer_csr.yaml

#Let's get the CSR to see it's current state. The CSR will delete after an hour
#This should currently be Pending, awaiting administrative approval
kubectl get certificatesigningrequests


#Approve the CSR
kubectl certificate approve developer-csr

#The CSR is updated with the certificate in .status.certificate
kubectl get certificatesigningrequests developer-csr 

kubectl get csr developer-csr -o jsonpath='{.status.certificate}' | base64 --decode > developer.crt
kubectl get csr

#The cluster’s Kubernetes API access details and the Cluster CA certificate
kubectl config view
ls /etc/kubernetes/pki/

# Set Cluster Configuration: (change IP server)
kubectl config set-cluster kubernetes --server=https://104.248.28.87:6443 --certificate-authority=/etc/kubernetes/pki/ca.crt --embed-certs=true --kubeconfig=developer.kubeconfig

# Set Credentials for Developer:
kubectl config set-credentials developer --client-certificate=developer.crt --client-key=developer.key --embed-certs=true --kubeconfig=developer.kubeconfig
# Set Developer Context: 
kubectl config set-context developer-context --cluster=kubernetes --namespace=default --user=developer --kubeconfig=developer.kubeconfig
# Use Developer Context:
kubectl config use-context developer-context --kubeconfig=developer.kubeconfig

#Verify the kubeconfig file’s configuration: (Get error)
kubectl --kubeconfig=developer.kubeconfig get pods

# Assign Roles and Bindings for the Developer User

# Create developer-cluster-role.yaml
cat <<EOF > developer-cluster-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: developer-role
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["pods"]
  verbs: ["*"]
EOF

# create developer-role-binding.yaml
cat <<EOF > developer-role-binding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-binding
  namespace: default
subjects:
- kind: User
  name: developer
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: developer-role #cluster-role name above
  apiGroup: rbac.authorization.k8s.io
EOF

kubectl apply -f developer-cluster-role.yaml -f developer-role-binding.yaml

# Verify developer User Rights

kubectl --kubeconfig=developer.kubeconfig get pods
kubectl --kubeconfig=developer.kubeconfig run nginx --image=nginx
kubectl --kubeconfig=developer.kubeconfig get pods

kubectl --kubeconfig=developer.kubeconfig get deployments

#Test with the demo user
kubectl get pod --as developer
kubectl get deployment --as developer

# References
* https://hbayraktar.medium.com/how-to-create-a-user-in-a-kubernetes-cluster-and-grant-access-bfeed991a0ef
* https://dev.azure.com/TOOA01/DevOps%20Basic%20Online/_git/Kubernetes-Advance-Course-02?path=/07-configuring-managing-kubernetes-security/7.2-Managing-Certificates-and-kubeconfig-Files/2-CreateUserCertificate.sh


#Create the role for developer which cannot delete pod
## kubectl create role developer --verb=create --verb=get --verb=list --verb=update --resource=pods

#Create rolebinding for developer
#kubectl create rolebinding developer-binding-demoUser --role=developer --user=demoUser

#Test with the demo user
#kubectl get pod --as demoUser
#kubectl get deployment --as demoUser
