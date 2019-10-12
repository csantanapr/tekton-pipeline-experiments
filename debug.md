
tkn resource list -n kabanero

oc create -f pipeline-debug.yaml -n kabanero
oc delete -f pipeline-debug.yaml -n kabanero

tkn pipeline list -n kabanero

tkn pipeline start csantana-nodejs-pipeline \
        -r git-source=csantana-nodejs-git \
        -r docker-image=csantana-nodejs-image \
        -s kabanero-operator \
        -n kabanero

tkn pipeline start csantana-nodejs-pipeline \
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

tkn pipelinerun list -n kabanero -l 1
tkn pipeline logs -f -n kabanero
tkn taskrun list -n kabanero -l 1



      command:
        [
          "/bin/bash",
          "-c",
          "cd /project && cp -a ${inputs.resources.git-source.path}/* $APPSODY_WATCH_DIR && $APPSODY_INSTALL && npm test; if [ $? -eq 0 ]; then npm test --prefix user-app; else exit 1; fi",
        ]


tkn pipelinerun describe custom-pipeline-nodejs-express-1569521168-r-80331 -n kabanero
Name:              custom-pipeline-nodejs-express-1569521168-r-80331
Namespace:         kabanero
Pipeline Ref:      nodejs-express-test-build-deploy-pipeline
Service Account:   kabanero-operator

Status
STARTED      DURATION     STATUS
3 days ago   17 minutes   Succeeded

Resources
NAME           RESOURCE REF
docker-image   custom-pipeline-nodejs-express-docker-image-1569521168
git-source     custom-pipeline-nodejs-express-git-source-1569521168

Params
NAME               VALUE
image-tag          3279cf5
image-name         docker-registry.default.svc:5000/hema-dev/appsody_sample_nodejs-express
release-name       appsody_sample_nodejs-express
repository-name    appsody_sample_nodejs-express
target-namespace   kabanero
docker-registry    docker-registry.default.svc:5000/hema-dev

Taskruns
NAME                                                              TASK NAME                        STARTED      DURATION     STATUS
custom-pipeline-nodejs-express-1569521168-r-80331-deploy--kh2n2   deploy-task                      3 days ago   19 seconds   Succeeded
custom-pipeline-nodejs-express-1569521168-r-80331-nodejs--wq2n5   nodejs-express-test-build-task   3 days ago   17 minutes   Succeeded

tkn pipelinerun describe -n kabanero csantana-nodejs-pipeline-run-g9xzv               
Name:              csantana-nodejs-pipeline-run-g9xzv
Namespace:         kabanero
Pipeline Ref:      csantana-nodejs-pipeline
Service Account:   kabanero-operator

Status
STARTED      DURATION   STATUS
2 days ago   1 second   Failed

Message
TaskRun csantana-nodejs-pipeline-run-g9xzv-nodejs-express-test-bu-kv7s6 has failed (Missing or invalid Task kabanero/csantana-nodejs-express-test-build-task: Pod "csantana-nodejs-pipeline-run-g9xzv-nodejs-express-test-bu-kv7s6-pod-71b05f" is invalid: [spec.containers[1].volumeMounts[0].name: Not found: "varlibcontainers", spec.containers[3].volumeMounts[0].name: Not found: "varlibcontainers"])

Resources
NAME           RESOURCE REF
git-source     csantana-nodejs-git
docker-image   csantana-nodejs-image

Params
No params

Taskruns
NAME                                                              TASK NAME                        STARTED      DURATION   STATUS
csantana-nodejs-pipeline-run-g9xzv-nodejs-express-test-bu-kv7s6   nodejs-express-test-build-task   2 days ago   ---        Failed(CouldntGetTask)