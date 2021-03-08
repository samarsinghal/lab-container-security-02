
Immutable means that a container won't be modified during its life: no updates, no patches, no configuration changes. If you must update the application code or apply a patch, you build a new image and redeploy it. Immutability makes deployments safer and more repeatable. If you need to roll back, you simply redeploy the old image. This approach allows you to deploy the same container image in every one of your environments, making them as identical as possible.

To use the same container image across different environments, we recommend that you externalize the container configuration (listening port, runtime options, and so on). Containers are usually configured with environment variables or configuration files mounted on a specific path. In Kubernetes, you can use both Secrets and ConfigMaps to inject configurations in containers as environment variables or files.

If you need to update a configuration, deploy a new container (based on the same image), with the updated configuration.

Change to the `~/container-immutability` sub directory.

```execute
clear
cd ~/container-immutability
```

### Immutable container setup using securityContext - readOnlyRootFilesystem

Create Pod holiday with two containers c1 and c2 of image bash:5.1.0. 
Force container c2 of Pod holiday to run immutable: no files can be changed during runtime by adding SecurityContext on container c2

    securityContext:
      readOnlyRootFilesystem: true

```execute
cat holiday.yaml  
```

Create the Pod:

```execute
kubectl apply -f holiday.yaml
```

Now test creating test file on both the containers 

```execute
kubectl exec holiday -c c1 -- touch /tmp/test # works
```

```execute
kubectl exec holiday -c c2 -- touch /tmp/test # error
```

The output shows that we are able to create file on container c1 but same operation failed on container c2 with error "Read-only file system"

Cleanup 

```execute
kubectl delete pod holiday
```


### Define container Immutability using startupProbe


create a pod

```execute
kubectl create -f mutable.yaml
```

Get a bash to the running Container

```execute
kubectl exec -it mutable -- bash
```

Verify touch command is working

```execute
touch test 
```

Exit container

```execute
exit
```

Use startupProbe to remove touch from containers

    startupProbe:
      exec:
        command:
        - rm
        - /bin/touch

```execute
cat immutable-no-touch.yaml
```

create a pod

```execute
kubectl create -f immutable-no-touch.yaml
```

Verify that the Pod's Container is running. Get a bash to the running Container

```execute
kubectl exec -it immutable-no-touch -- bash
```

Verify touch command is not working

```execute
touch test 
exit
```

Exit container

```execute
exit
```

Now, Use startupProbe to remove bash from containers


create a pod

```execute
kubectl create -f immutable-no-bash.yaml
```

Verify that the Pod's Container is running. Get a bash to the running Container you should see "executable file not found" error

```execute
kubectl exec -it immutable-no-bash -- bash
```

Cleanup 
```execute
kubectl delete pods mutable immutable-no-bash immutable-no-touch
```

