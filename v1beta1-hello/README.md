Example showing new Tekton resources

- Create a new namespace to experiment
    ```bash
    kubectl create ns tekton-experiments
    ```
- Set the environment variable to be use in subsequent commands
    ```bash
    export NAMESPACE=tekton-experiments
    ```

- Create the environment variables to be use, replace with real values and include the single quotes:
    ```bash
    export DOCKER_USERNAME='<DOCKER_USERNAME>'
    ```
    ```bash
    export DOCKER_PASSWORD='<DOCKER_PASSWORD>'
    ```
    ```bash
    export DOCKER_EMAIL='<DOCKER_EMAIL>'
    ```
- Run the following command to create a secret `regcred` in the namespace `NAMESPACE`
    ```bash
    kubectl create secret docker-registry regcred \
      --docker-server=https://index.docker.io/v1/ \
      --docker-username=${DOCKER_USERNAME} \
      --docker-password=${DOCKER_PASSWORD} \
      --docker-email=${DOCKER_EMAIL} \
      -n ${NAMESPACE}
    ```

- Install the `git-clone` task
    ```bash
    kubectl create -n ${NAMESPACE} -f https://raw.githubusercontent.com/tektoncd/catalog/v1beta1/git/git-clone.yaml
    ```

- Install tasks and pipelines
    ```bash
    kubectl apply -n ${NAMESPACE} -f ./
    ```

- Run the pipeline test
    ```bash
    tkn pipeline start \
     test \
     -w name=w,claimName=my-pvc \
     --showlog \
     -n ${NAMESPACE} 
    ```

- Run the pipeline build using the ServiceAccount `pipeline`
    ```bash
    tkn pipeline start \
     build \
     -w name=w,claimName=my-pvc \
     -p image-repository=${DOCKER_USERNAME}
     -s pipeline \
     --showlog \
     -n ${NAMESPACE} 
    ```
