In this scenerio, we will work with k8s(Kubernetes different services such as pods, secret, pv, pvc, RBAC)

A. Install Pre-requisite:

sudo apt install docker.io -y
sudo systemctl unmask docker
sudo service docker restart
curl -L0 https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
sudo dpkg -i minikube_latest_amd64.deb
rm -rf minikube_latest_amd64.deb

B. Environment Set-up
Check whether docker & minikube are properly running installed and configured.
Start minikube and execute this command to sync host docker with minikube docker: $ minikube -p minikube docker-env and $ eval $(minikube docker-env)

C. Project Scenerios:
We are going to work on kubernetes ConfigMaps, Secrets, Persistence Storage, Persistence Storage Claims and RBAC

>>CONFIGMAP
 STEP 1:
 Create a configMap named 'config-map'.
 Add key 'SERVER_URL'
 Add value 'https://kubernetes.io'
 Verify if the ConfigMap is created.

 Ans: Create:  $ kubectl create configmap config-map --from-literal=SERVER_URL=https://kubernetes.io
  Verify: $ kubectl get configmaps

 STEP 2: 
 Create a nginx pod with the environment variable 'SERVER_URL_ENV'.
 Use the configmap created on step 1 and assign the value to it. 
 Name the pod as 'nginx-pod'
 Name the container as 'nginx-container'

 Ans: nginx-pod.yaml

 Test your configuration by executing below command: 
 kubectl exec -it nginx-pod -- sh -c env | grep SERVER_URL_ENV
 It should display: https://kubernetes.io/

>>SECRETS
 STEP 1:
 Create a secret named 'k8s-secret' using:
 data:
   user: admin
   pass: pass

 For making them into base 64 use commands: $ echo "your_key" | base64 (for e.g. admin will be replaced with YWRtaW4K)
 Ans: k8s-secret.yaml

 STEP 2:
 Modify the above created nginx pod to add the 'k8s-secret' and mountPath /etc/test:
 Use below command to check if pod and secret are successfully configured:
 kubectl exec -it nginx-pod -- sh -c "cat /etc/test/* | base64"
 It should display both username and password.

 Ans: nginx-pod-update1.yaml (you have to delete previously create then appky this new one)

>>PERSISTENCE VOLUME
 STEP 1: 
 Create a PV named 'k8s-pv'using the following parameters:
 ...
 storageClassName - manual
 capacity - 100MB
 accessMode - ReadWriteOnce
 hostPath - /tmp/k8s
 ...

 Ans: pv.yaml

 STEP 2: 
 Create a PVC named 'k8s-pvc' and request for 50MB.
 To verify successful creation, ensure it to bound to 'k8s-pv'.

 Ans: pvc.yaml

 STEP 3:
 Modify above nginx pod named 'nginx-pod' using the following parameters:
 ...
 Request for 'k8s-pvc' as a volume
 Use /usr/share/nginx/html for mount path.
 ...
 **Hint: Use 'kubectl describe pod nginx-pod' for debugging.

 Ans: delete existing nginx-pod if running. Then apply "nginx-pod-update2.yaml" by $kubectl apply -f nginx-pod-update2.yaml

>>RBAC (Ans: RBAC.txt)
 In this section, you will create a user 'emp' and assign 'read' rights on pods belonging to the namepsace 'dev'.

 Create a namespace named 'dev'
 Use 'openssl', and create a private key named 'emp.key'.

 Create a certificate sign request named 'emp.csr' using the private key generated earlier.
 use the following information:
 ...
 name: emp
 group: dev
 ...
 Generate 'emp.crt' certificate by approving the request create earlier.
 
 Create a new context pointing to the cluster 'minikube', and name it 'dev-ctx'. It should point to the namepsace 'dev', and the user should be 'emp'.

 Set Credentials for 'emp'.
 Use 'emp.key' and 'emp.crt' created earlier.

 Create a role named 'emp-role', and assign 'get', 'list' access on 'pods' and 'deployments'(use 'dev' namespace).
 Bind 'emp' to the role 'emp-role' created earlier, and name it 'emp-bind'.
 Run an 'nginx' pod under the 'dev-ctx' and 'dev' namepsace and 'nginx' name.

 Execute 'kubectl --context=dev-ctx get pods -o wide', and ensure it is deployed.

 If you try to execute 'kubectl --context=dev-ctx get pods -n context', a 'forbidden' error appears. This is because only employess are authorized to access the 'dev' namepsace.

