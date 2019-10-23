# Tekton Debugging
Small repo with experiments for debugging and customizing tekton pipelines


## Prerequisites
CLIs to get
- `oc`
- `tkn` [Install Instructions](https://github.com/tektoncd/cli#installing-tkn)
- `kubens`

Use `tkn` to work with Tekton

## Check current resources

```
kubens kabanero
```
out:
```
Context "kabanero/c100-e-us-east-containers-cloud-ibm-com:31718/IAM#csantana@us.ibm.com" modified.
Active namespace is "kabanero".
```

```
tkn pipelines list
```
out:
```
java-microprofile-build-deploy-pipeline             5 weeks ago    csantana-microprofile-1569380497                      3 weeks ago    10 minutes   Succeeded
java-microprofile-test-build-deploy-pipeline        22 hours ago   custom-pipeline-microprofile-1571168702               22 hours ago   22 minutes   Succeeded
java-spring-boot2-build-deploy-pipeline             5 weeks ago    csantana-spring-1569468014                            2 weeks ago    16 minutes   Succeeded
java-spring-boot2-test-build-deploy-pipeline        2 weeks ago    custom-pipeline-java-spring-boot2-1569612446          2 weeks ago    5 minutes    Failed
nodejs-build-deploy-pipeline                        5 weeks ago    ---                                                   ---            ---          ---
nodejs-express-build-deploy-pipeline                5 weeks ago    csantanapr-webhook-1570886792                         4 days ago     13 minutes   Succeeded
nodejs-express-test-build-deploy-pipeline           4 days ago     nodejs-express-test-build-deploy-pipeline-run-tr8wj   4 days ago     12 minutes   Succeeded
nodejs-loopback-build-deploy-pipeline               5 weeks ago    ---                                                   ---            ---          ---
pipeline0                                           5 weeks ago    ---                                                   ---            ---          ---```
```

```
tkn tasks list
```
out:
```
collections-build-task               4 weeks ago
java-microprofile-build-task         5 weeks ago
java-microprofile-deploy-task        5 weeks ago
java-microprofile-test-task          22 hours ago
java-spring-boot2-build-task         5 weeks ago
java-spring-boot2-deploy-task        5 weeks ago
java-spring-boot2-test-task          22 hours ago
monitor-result-task                  5 weeks ago
nodejs-build-task                    5 weeks ago
nodejs-deploy-task                   5 weeks ago
nodejs-express-build-task            5 weeks ago
nodejs-express-deploy-task           5 weeks ago
nodejs-express-test-task             1 week ago
nodejs-loopback-build-task           5 weeks ago
nodejs-loopback-deploy-task          5 weeks ago
pipeline0-task                       5 weeks ago
```

# Node.js Test Task

Create the git and image resource
```
oc create -f nodejs-express/resources.yaml -n kabanero
```
output
```
pipelineresource.tekton.dev/csantana-nodejs-git created
pipelineresource.tekton.dev/csantana-nodejs-image created
```

Verify resources
```
tkn resource list | grep csantana-nodejs
```
output
```
csantana-nodejs-git         git     url: https://github.com/csantanapr/appsody-sample-nodejs-express
csantana-nodejs-image       image   url: docker-registry.default.svc:5000/csantana/appsody-sample-nodejs-express:tkn
```

Creat the test Task
```
oc create -f nodejs-express/test-task.yaml            
```
out:
```
task.tekton.dev/nodejs-express-test-task created
```

Create a pipeline that only runs the test task
```
oc create -f nodejs-express/test-pipeline.yaml 
```
out:
```
pipeline.tekton.dev/nodejs-express-test-pipeline created
```

Start a pipeline run
```
tkn pipeline start nodejs-express-test-pipeline \
        -r git-source=csantana-nodejs-git \
        -s kabanero-operator \
        -n kabanero
```
out:
```
Pipelinerun started: nodejs-express-test-pipeline-run-5cp78

In order to track the pipelinerun progress run:
tkn pipelinerun logs nodejs-express-test-pipeline-run-5cp78 -f -n kabanero
```

Verify the pipeline run
```
tkn pipelinerun list -n kabanero -l 1             
```
out:
```
NAME                                     STARTED          DURATION   STATUS    
nodejs-express-test-pipeline-run-5cp78   20 seconds ago   ---        Running 
```

Verify the task run
```
tkn taskrun list -n kabanero -l 1
```
out:
```
NAME                                                     STARTED          DURATION     STATUS      
nodejs-express-test-pipeline-run-5cp78-test-task-g6tt7   32 seconds ago   24 seconds   Succeeded  
```

Describe the pipeline run
```
tkn pipelinerun describe nodejs-express-test-pipeline-run-5cp78 
```
out:
```               
Name:              nodejs-express-test-pipeline-run-5cp78
Namespace:         kabanero
Pipeline Ref:      nodejs-express-test-pipeline
Service Account:   kabanero-operator

Status
STARTED          DURATION     STATUS
56 seconds ago   24 seconds   Succeeded

Resources
NAME         RESOURCE REF
git-source   csantana-nodejs-git

Params
No params

Taskruns
NAME                                                     TASK NAME   STARTED          DURATION     STATUS
nodejs-express-test-pipeline-run-5cp78-test-task-g6tt7   test-task   56 seconds ago   24 seconds   Succeeded
```

Describe the task run
```
tkn taskrun describe  nodejs-express-test-pipeline-run-5cp78-test-task-g6tt7 
```
out:
```                
Name:              nodejs-express-test-pipeline-run-5cp78-test-task-g6tt7
Namespace:         kabanero
Task Ref:          nodejs-express-test-task
Service Account:   kabanero-operator

Status
STARTED        DURATION     STATUS
1 minute ago   24 seconds   Succeeded

Input Resources
NAME         RESOURCE REF
git-source   csantana-nodejs-git

Output Resources
No resources

Params
No params

Steps
NAME
npm-test
git-source-csantana-nodejs-git-bkf9j
```

Get the logs for a task
```
tkn taskrun logs nodejs-express-test-pipeline-run-5cp78-test-task-g6tt7 -f
```

Get the logs for a pipeline
```
tkn pipelinerun logs nodejs-express-test-pipeline-run-5cp78 -f
```

Create a full pipeline test, build, deploy
```
oc create -f nodejs-express/test-build-deploy-pipeline.yaml
```
out:
```
pipeline.tekton.dev/nodejs-express-test-build-deploy-pipeline created
```

Run the pipeline using all the parameters used by the tekton git webhook.
In this case lets deploy the app to the `kabanero-samples` namespace
```
tkn pipeline start nodejs-express-test-build-deploy-pipeline \
        -r git-source=csantana-nodejs-git \
        -r docker-image=csantana-nodejs-image \
        -p image-tag=master \
        -p image-name=docker-registry.default.svc:5000/kabanero-samples/appsody-sample-nodejs-express \
        -p release-name=appsody-sample-nodejs-express \
        -p repository-name=appsody-sample-nodejs-express \
        -p target-namespace=kabanero \
        -p docker-registry=docker-registry.default.svc:5000/kabanero-samples \
        -s kabanero-operator \
        -n kabanero
```
out:
```
Pipelinerun started: nodejs-express-test-build-deploy-pipeline-run-6kxgj

In order to track the pipelinerun progress run:
tkn pipelinerun logs nodejs-express-test-build-deploy-pipeline-run-6kxgj -f -n kabanero
```

Get the logs
```
tkn pipelinerun logs nodejs-express-test-build-deploy-pipeline-run-6kxgj -f -n kabanero
```
out:
```
[test-task : git-source-csantana-nodejs-git-8t2zn] {"level":"info","ts":1571251643.5800052,"logger":"fallback-logger","caller":"git/git.go:102","msg":"Successfully cloned https://github.com/csantanapr/appsody-sample-nodejs-express @ master in path /workspace/git-source"}

[test-task : npm-test] App started on PORT 3000
[test-task : npm-test]   Node.js Express stack
[test-task : npm-test]     /metrics endpoint
[test-task : npm-test]       ✓ status (39ms)
[test-task : npm-test]       ✓ contains os_cpu_used_ratio
[test-task : npm-test]       ✓ contains process_cpu_used_ratio
[test-task : npm-test]     /ready endpoint
[test-task : npm-test]       ✓ status
[test-task : npm-test]     /live endpoint
[test-task : npm-test]       ✓ status
[test-task : npm-test]     /health endpoint
[test-task : npm-test]       ✓ status
[test-task : npm-test]     /blah endpoint
[test-task : npm-test]       ✓ status
[test-task : npm-test]   7 passing (171ms)
[test-task : npm-test] 
[test-task : npm-test] App started on PORT 3000
[test-task : npm-test]   Node.js Express Simple template
[test-task : npm-test]     / endpoint
[test-task : npm-test]       ✓ status (67ms)
[test-task : npm-test] 
[test-task : npm-test]   1 passing (176ms)
```

To clean use `tkn` 
```
tkn resource delete csantana-nodejs-git -f
tkn resource delete csantana-nodejs-image -f
tkn task delete nodejs-express-test-task -f
tkn pipeline delete nodejs-test-pipeline -f 
tkn pipeline delete nodejs-express-test-build-deploy-pipeline -f 
```
or use `oc`
```
oc delete -f nodejs-express/
```

