# Deploy the Superhero Ratings App to ACS

## Review/Edit the YAML Config Files

* In VS Code (or vi), open the `helper_files` directory and review the yaml file: `heroes.yaml`
* Note the environment variables that direct each app to other services.
* Update the yaml file for the proper container image names. 
    * You will need to replace the `<login server>` with the ACR login server created in Lab 2. 
    * Repeat this **THREE** times in the heroes yaml file (for the web, api, and db images). Example: 

        ```
        spec:
        containers:
        - image: mycontainerregistry.azurecr.io/azureworkshop/rating-web:v1
            name:  heroes-web-cntnr
        ```

## Setup ACS with access to Azure Container Registry

There are a few ways that AKS clusters can access your private Azure Container Registry. Generally the service account that kubernetes utilizes will have rights based on its Azure credentials. In our lab config, we must create a secret to allow this access. 

```
# set these values to yours
ACR_SERVER=
ACR_USER=
ACR_PWD=

kubectl create secret docker-registry acr-secret --docker-server=$ACR_SERVER --docker-username=$ACR_USER --docker-password=$ACR_PWD --docker-email=superman@heroes.com
```

> Note: You can review the `heroes.yaml` and see where the `imagePullSecrets` is configured.

## Deploy the Applications to AKS

* Use the kubectl CLI to deploy each app

    ```
    cd ~/blackbelt-aks-hackfest/linux-container-workshop/helper_files

    kubectl apply -f heroes.yaml
    ```

## Validate

* Check to see if pods are running in your cluster

    ```
    kubectl get pods

    NAME                                 READY     STATUS    RESTARTS   AGE
    heroes-api-deploy-1140957751-2z16s   1/1       Running   0          2m
    heroes-db-deploy-2357291595-k7wjk    1/1       Running   0          3m
    heroes-web-1645635641-pfzf9          1/1       Running   0          2m
    ```
* Run script to import data into MongoDB

    ```
    # replace the pod name below with your db pod name
    # once you exec into pod, run the `import.sh` script

    kubectl exec -it heroes-db-deploy-2357291595-xb4xm bash
    root@heroes-db-deploy-2357291595-xb4xm:/# ./import.sh
    2018-01-16T21:38:44.819+0000	connected to: localhost
    2018-01-16T21:38:44.918+0000	imported 4 documents
    2018-01-16T21:38:44.927+0000	connected to: localhost
    2018-01-16T21:38:45.031+0000	imported 72 documents
    2018-01-16T21:38:45.040+0000	connected to: localhost
    2018-01-16T21:38:45.152+0000	imported 2 documents
    root@heroes-db-deploy-2357291595-xb4xm:/# exit

    # be sure to exit pod as shown above
    ```

* Check to see if services are deployed

    ```
    kubectl get service

    NAME         TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)          AGE
    api          LoadBalancer   10.0.20.156   52.176.104.50    3000:31416/TCP   5m
    kubernetes   ClusterIP      10.0.0.1      <none>           443/TCP          12m
    mongodb      ClusterIP      10.0.5.133    <none>           27017/TCP        5m
    web          LoadBalancer   10.0.54.206   52.165.235.114   8080:32404/TCP   5m
    ```

* Browse to the External IP for your web application (on port 8080) and try the app