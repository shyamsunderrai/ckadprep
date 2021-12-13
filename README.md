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

