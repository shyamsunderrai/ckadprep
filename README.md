# ckadprep
Course sessions and output from killer.sh

### Question 1: Task Weight 2%
```text
Create a single Pod of image httpd:2.4.41-alpine in Namespace default. 
The Pod should be named pod1 and the container should be named pod1-container.

Your manager would like to run a command manually on occasion to output the status of that exact Pod. 
Please write a command that does this into /opt/course/2/pod1-status-command.sh. The command should use kubectl.
```

#### Answer
```bash
kubectl get pod earth-2x3-api-686fc9cb8f-rfww7 -n earth -o yaml > pod1.yaml   //// Take any running pod and copy the template

---  //// Edit the pod and keep what is needed
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: default
spec:
  containers:
  - image: httpd:2.4.41-alpine
    imagePullPolicy: IfNotPresent
    name: pod1-container
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File

--- //// Generate the command needed to check the status
echo "kubectl get pod pod1 -o wide" >> /opt/course/2/pod1-status-command.sh
```

### Question 2: Task Weight 1%
```text
The DevOps team would like to get the list of all Namespaces in the cluster. 
Get the list and save it to /opt/course/1/namespaces.
```
#### Answer
```bash
////To get the list
kubectl get namespaces
NAME              STATUS   AGE
default           Active   88d
earth             Active   88d
jupiter           Active   88d
kube-node-lease   Active   88d
kube-public       Active   88d
kube-system       Active   88d
mars              Active   88d
mercury           Active   88d
moon              Active   88d
neptune           Active   88d
pluto             Active   88d
saturn            Active   88d
shell-intern      Active   88d
sun               Active   88d
venus             Active   88d

////To get the list and save it to the required file 
kubectl get namespaces > /opt/course/1/namespaces
```

### Question 3: Task Weight 2%
```text
Team Neptune needs a Job template located at /opt/course/3/job.yaml. 
This Job should run image busybox:1.31.0 and execute sleep 2 && echo done. 
It should be in namespace neptune, run a total of 3 times and should execute 2 runs in parallel.

Start the Job and check its history. 
Each pod created by the Job should have the label id: awesome-job. 
The job should be named neb-new-job and the container neb-new-job-container.
```
#### Answer
```bash
kubectl create job neb-new-job -n neptune -o yaml --image=busybox:1.31.0 -- "sleep 2 && echo done"
--- //// Edited the file to include container name. 
Label can be added here itself OR can be added using """kubectl label jobs neptunejob id=awesome-job -n neptune"""

cat /opt/course/3/job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  labels:
    job-name: neb-new-job
    id: awesome-job
  name: neb-new-job
  namespace: neptune
spec:
  completions: 3
  parallelism: 2
  selector:
    matchLabels:
  template:
    metadata:
      creationTimestamp: null
      labels:
        job-name: neb-new-job
    spec:
      containers:
      - command:
        - sleep 2 && echo done
        image: busybox:1.31.0
        imagePullPolicy: IfNotPresent
        name: neb-new-job-container
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Never
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status: {}

---
Check the status of the job
kubectl describe jobs neb-new-job -n neptune
```

### Question 3: Weight 5%
```text
Team Mercury asked you to perform some operations using Helm, all in Namespace mercury:

- Delete release internal-issue-report-apiv1
- Upgrade release internal-issue-report-apiv2 to any newer version of chart bitnami/nginx available
- Install a new release internal-issue-report-apache of chart bitnami/apache. 
  The Deployment should have two replicas, set these via Helm-values during install
- There seems to be a broken release, stuck in pending-upgrade state. Find it and delete it
```

#### Answer
List the repo * Remove the release
```bash
helm list -n mercury
NAME                            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
internal-issue-report-apiv1     mercury         1               2021-09-16 12:11:34.76544812 +0000 UTC  deployed        nginx-9.5.0     1.21.1     
internal-issue-report-apiv2     mercury         1               2021-09-16 12:11:38.122758697 +0000 UTC deployed        nginx-9.5.0     1.21.1     
internal-issue-report-app       mercury         1               2021-09-16 12:11:41.417558951 +0000 UTC deployed        nginx-9.5.0     1.21.1  
helm uninstall internal-issue-report-apiv1 -n mercury
release "internal-issue-report-apiv1" uninstalled
```

Search for the lastest version of bitnami/nginx and then execute upgrade
```bash
helm search repo bitnami | egrep -i nginx
bitnami/nginx                                   9.5.16          1.21.4          Chart for the nginx server                        
bitnami/nginx-ingress-controller                9.0.9           1.1.0           Chart for the nginx Ingress controller 

helm repo upgrade
helm upgrade internal-issue-report-apiv2 bitnami/nginx --version=9.5.16 -n mercury 
Release "internal-issue-report-apiv2" has been upgraded. Happy Helming!
NAME: internal-issue-report-apiv2
LAST DEPLOYED: Mon Dec 13 13:11:51 2021
NAMESPACE: mercury
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
CHART NAME: nginx
CHART VERSION: 9.5.16
APP VERSION: 1.21.4

** Please be patient while the chart is being deployed **

NGINX can be accessed through the following DNS name from within your cluster:

    internal-issue-report-apiv2-nginx.mercury.svc.cluster.local (port 80)

To access NGINX from outside the cluster, follow the steps below:

1. Get the NGINX URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace mercury -w internal-issue-report-apiv2-nginx'

    export SERVICE_PORT=$(kubectl get --namespace mercury -o jsonpath="{.spec.ports[0].port}" services internal-issue-report-apiv2-nginx)
    export SERVICE_IP=$(kubectl get svc --namespace mercury internal-issue-report-apiv2-nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    echo "http://${SERVICE_IP}:${SERVICE_PORT}"
```

Install a new release and scale to two replicas
```bash
helm install internal-issue-report-apache bitnami/apache -n mercury
NAME: internal-issue-report-apache
LAST DEPLOYED: Mon Dec 13 13:28:37 2021
NAMESPACE: mercury
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: apache
CHART VERSION: 8.9.6
APP VERSION: 2.4.51

** Please be patient while the chart is being deployed **

1. Get the Apache URL by running:

** Please ensure an external IP is associated to the internal-issue-report-apache service before proceeding **
** Watch the status using: kubectl get svc --namespace mercury -w internal-issue-report-apache **

  export SERVICE_IP=$(kubectl get svc --namespace mercury internal-issue-report-apache --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
  echo URL            : http://$SERVICE_IP/


WARNING: You did not provide a custom web application. Apache will be deployed with a default page. Check the README section "Deploying your custom web application" in https://github.com/bitnami/charts/blob/master/bitnami/apache/README.md#deploying-your-custom-web-application.

kubectl scale deployments/internal-issue-report-apache --replicas=2 -n mercury 
deployment.apps/internal-issue-report-apache scaled
```

** Was unable to find the fourth "pending-state" problem anywhere **
















