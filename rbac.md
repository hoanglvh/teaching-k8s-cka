# Create user demouser with role permission developer only on the pod
#Generate the ssl certificate for demoUser
openssl genrsa -out demoUser.key 2048
openssl req -new -key demoUser.key -out demoUser.csr 

or 
#CN (Common Name) is your username, O (Organization) is the Group
openssl req -new -key demouser.key -out demouser.csr -subj "/CN=demouser"

#Get the csr file content
CSR=$(cat demoUser.csr | base64 | tr -d "\n")

#Submit the CertificateSigningRequest to the API Server
#Key elements, name, request and usages (must be client auth)
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: demouser
spec:
  groups:
  - system:authenticated  
  request: $(cat demouser.base64.csr)
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF

#Let's get the CSR to see it's current state. The CSR will delete after an hour
#This should currently be Pending, awaiting administrative approval
kubectl get certificatesigningrequests


#Approve the CSR
kubectl certificate approve demouser

#The CSR is updated with the certificate in .status.certificate
kubectl get certificatesigningrequests demouser 

#Retrieve the certificate from the CSR object, it's base64 encoded
kubectl get certificatesigningrequests demouser \
  -o jsonpath='{ .status.certificate }'  | base64 --decode


#Let's go ahead and save the certificate into a local file. 
#We're going to use this file to build a kubeconfig file to authenticate to the API Server with
kubectl get certificatesigningrequests demouser \
  -o jsonpath='{ .status.certificate }'  | base64 --decode > demouser.crt 

#Check the contents of the file
cat demouser.crt


#Read the certficate itself
#Key elements: Issuer is our CA, Validity one year, Subject CN=demousers
openssl x509 -in demouser.crt -text -noout | head -n 15

#Create the role for developer which cannot delete pod
kubectl create role developer --verb=create --verb=get --verb=list --verb=update --resource=pods

#Create rolebinding for developer
kubectl create rolebinding developer-binding-demoUser --role=developer --user=demoUser

#Test with the demo user
kubectl get pod --as demoUser
kubectl get deployment --as demoUser
