# Jsonpath

## Get the name of all the pods in a namespace
To get the name of all the pods in a namespace, you can use the following command:

kubectl get pods -n <namespace> -o jsonpath='{.items[*].metadata.name}'
This command returns the name of all the pods in the specified namespace.

## Get the IP address of a pod
To get the IP address of a pod, you can use the following command:

kubectl get pod <pod-name> -o jsonpath='{.status.podIP}'
This command returns the IP address of the specified pod.

## Get the labels of a pod
To get the labels of a pod, you can use the following command:

kubectl get pod <pod-name> -o jsonpath='{.metadata.labels}'
This command returns the labels of the specified pod.

## Get the containers of a pod
To get the containers of a pod, you can use the following command:

kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].name}'
This command returns the names of all the containers in the specified pod.

## Get all pod and IP
kubectl get pod -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.podIP}{"\n"}{end}'

## Get Pod name and status
kubectl get pod -o=custom-columns="POD_NAME:.metadata.name,POD_STATUS:.status.containerStatuses[].state"
