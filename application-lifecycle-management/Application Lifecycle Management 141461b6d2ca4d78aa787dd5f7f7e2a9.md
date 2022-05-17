# Application Lifecycle Management

### Rolling Updates and Rollbacks

- When you create a Deployment it triggers a Rollout which creates a new Revision, whenever the Deployment is updated this cycle happens again
- You can check the status of the Rollout with `kubectl rollout status $DEPLOYMENT_NAME`
- To see the Rollout history use the `kubectl rollout history $DEPLOYMENT_NAME` command
- There are 2 Deployment Strategies
    - Recreate: destroys all current versions of the Pods and recreates them with the new version
    - Rolling Update: the Pods are destroyed and created one by one, minimizing downtime of the application
- When a Deployment is upgraded, a new ReplicaSet is created and it populates the new ReplicaSet while simultaneously “downgrading” the old ReplicaSet
- To undo an upgrade use the `kubectl rollout undo $DEPLOYMENT_NAME` command, the Deployment will destroy the Pods in the new ReplicaSet and create the Pods in the old ReplicaSet
- In the Deployment you can specify how many Pods can be down at a time for an upgrade (only for the RollingUpdate strategy)
    
    ```yaml
    strategy:
      rollingUpdate:
        maxSurge: 25%
        maxUnavailable: 25%
      type: RollingUpdate
    ```
    

Links:

- [https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)

### Commands and Arguments

- Anything that is supposed to be appended to the `docker run` command, has to go into the `args` part of the Pod definition file under `containers`
- If you need to replace the entrypoint then you can use the `command` section of the Pod definition file, also under the `containers` section

Links:

- [https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/](https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/)

### Environment Variables

- Use the `env` section under `containers` to set environment variables as key-value pairs for the application
- You can set environment variable values using key-value pairs, ConfigMaps and Secrets
    
    ```yaml
    # Key-value format
    env:
      - name: APP_COLOR
        value: pink
    # ConfigMap format
    env:
      - name: APP_COLOR
        valueFrom:
          configMapKeyRef:
    # Secret format
    env:
      - name: APP_COLOR
        valueFrom:
          secretKeyRef:
    
    ```
    

Links:

- [https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/)

### ConfigMaps

- You can define key-value pairs inside ConfigMaps and then inject those ConfigMaps into your application
- Using the imperative approach, you can easily define your key-value pairs in a single command `kubectl create configmap $CONFIGMAP_NAME --from-literal=$KEY=$VALUE` or using a file `kubectl create configmap $CONFIGMAP_NAME --from-file=$FILE`
    - To specify multiple values in the command, repeat the `--from-literal` part as many times as needed
    - The best way (in my opinion) is to use the imperative approach and append `--dry-run=client -o yaml > $FILENAME.yaml` to the command to create a declarative yaml file of the ConfigMap
- Attach a ConfigMap to a Pod by adding the `envFrom` section to the `containers` part of the Pod definition
    
    ```yaml
    # Inject the whole ConfigMap to a Pod
    envFrom:
      - configMapRef:
          name: $CONFIGMAP_NAME
    # Inject environment variables to a Pod
    env:
      - name: $ENV_NAME
        valueFrom:
          configMapKeyRef:
            name: $CONFIGMAP_NAME
            key: $KEY_NAME
    # Inject ConfigMap as a Volume
    volumes:
      - name: $CONFIGMAP_VOLUME
        configMap:
          name: $CONFIGMAP_NAME
    ```
    

Links:

- [https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

### Secrets

- Secrets are used to store sensitive data, first the Secret object has to be created and the injected into a Pod
- The creation is handled exactly the same way as ConfigMaps so I won’t list it here
- When creating the Secret using the declarative approach, you have to add the hashed format of the data
- To encode data use the `echo -n $TEXT | base64` command
- To view Secret data use the `kubectl get secret $SECRET_NAME -o yaml` command
- To decode values use the `echo -n $TEXT | base64 --decode` command
    
    ```yaml
    # Inject the whole Secret to a Pod
    envFrom:
      - secretRef:
          name: $SECRET_NAME
    # Inject a single Secret data as an environment variable
    env:
      - name: $ENV_NAME
        valueFrom:
          secretKeyRef:
            name: $SECRET_NAME
            key: $KEY_NAME
    # Inject Secret as a Volume
    volumes:
      - name: $SECRET_VOLUME
        secret:
          secretName: $SECRET_NAME
    ```
    
- If you add a Secret as a Volume, then each secret will be created as a file inside the container
- A secret is only sent to a node if a pod on that node requires it.
- Kubelet stores the secret into a tmpfs so that the secret is not written to disk storage.
- Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.

Links:

- [https://kubernetes.io/docs/concepts/configuration/secret/](https://kubernetes.io/docs/concepts/configuration/secret/)

### Multi-Container Pods

- Multi-Container Pods share the same network and storage, so they can refer to eachother as localhost
- To make a Multi-Container Pod, just add another container to the `containers` array in the Pod definition file

### InitContainers

- If you want to run a workload that has to run to completion like a script for example, then InitContainers are a great way to solve this issue
- InitContainers are specified under the `initContainers` section under `spec` in the Pod definition file
- When a Pod is first created the InitContainer is run and they have to run to completion before the real container starts
- You can configure multiple InitContainers but they run one at a time in a sequential order
- The Pod is restarted if any of the InitContainers fail

Links: [https://kubernetes.io/docs/concepts/workloads/pods/init-containers/](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)