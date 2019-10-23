## Simple tekton task run

### Create TaskRun
Run the task by creating the `TaskRun` custom resource
```
kubectl create -f taskrun.yaml
```
Make a change and re-run the `kubectl create` command

### Watch logs for TaskRun
Stream the logs using `stern`
```
stern task -c step -n kabanero
```
