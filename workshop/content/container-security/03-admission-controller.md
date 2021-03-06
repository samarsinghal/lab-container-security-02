# Kubernetes Admission Controller Webhook

Kubernetes admission controllers are plugins that govern and enforce how the cluster is used. They can be thought of as a gatekeeper that intercept (authenticated) API requests and may change the request object or deny the request altogether. The admission control process has two phases: the mutating phase is executed first, followed by the validating phase. Consequently, admission controllers can act as mutating or validating controllers or as a combination of both. 

## Verify

For this demo we deployed a Webhook Server on the cluster - lets verify the webhook-server pod is running

1. The `webhook-server` pod in the `webhook-demo` namespace should be running:
```
$ kubectl -n webhook-demo get pods
NAME                             READY     STATUS    RESTARTS   AGE
webhook-server-6f976f7bf-hssc9   1/1       Running   0          35m
```

If case webhook server is not running. 
Run `./deploy.sh`. This will create a CA, a certificate and private key for the webhook server,
and deploy the resources in the newly created `webhook-demo` namespace in your Kubernetes cluster.


2. A `MutatingWebhookConfiguration` named `demo-webhook` should exist:

A mutating admission controller webhook is defined by creating a MutatingWebhookConfiguration object in Kubernetes
This configuration defines a webhook webhook-server.webhook-demo.svc, and instructs the Kubernetes API server to consult the service webhook-server in namespace webhook-demo whenever a pod is created by making a HTTP POST request to the /mutate URL.

```
$ kubectl get mutatingwebhookconfigurations
NAME           AGE
demo-webhook   36m
```

## Admission Controller Webhook Demo

The logic of this demo webhook is fairly simple: it enforces more secure defaults for running
containers as non-root user. While it is still possible to run containers as root, the webhook
ensures that this is only possible if the setting `runAsNonRoot` is *explicitly* set to `false`
in the `securityContext` of the Pod. If no value is set for `runAsNonRoot`, a default of `true`
is applied, and the user ID defaults to `1234`.

Change to the `~/admission-controller` sub directory.

```execute
clear
cd ~/admission-controller
```


1. Deploy [a pod](examples/pod-with-defaults.yaml) that neither sets `runAsNonRoot` nor `runAsUser`:
```
$ kubectl create -f examples/pod-with-defaults.yaml
```
Verify that the pod has default values in its security context filled in:
```
$ kubectl get pod/pod-with-defaults -o yaml
...
  securityContext:
    runAsNonRoot: true
    runAsUser: 1234
...
```
Also, check the logs that the pod had in fact been running as a non-root user:
```
$ kubectl logs pod-with-defaults
I am running as user 1234
```

2. Deploy a pod that explicitly sets `runAsNonRoot` to `false`, allowing it to run as the
`root` user:
```
$ kubectl create -f examples/pod-with-override.yaml
$ kubectl get pod/pod-with-override -o yaml
...
  securityContext:
    runAsNonRoot: false
...
$ kubectl logs pod-with-override
I am running as user 0
```

3. Attempt to deploy a pod that has a conflicting setting: `runAsNonRoot` set to `true`, but `runAsUser` set to false.
The admission controller should block the creation of that pod.
```
$ kubectl create -f examples/pod-with-conflict.yaml 
Error from server (InternalError): error when creating "examples/pod-with-conflict.yaml": Internal error
occurred: admission webhook "webhook-server.webhook-demo.svc" denied the request: runAsNonRoot specified,
but runAsUser set to 0 (the root user)
```

## Build the Image from Sources (optional)

An image can be built by running `make`.
If you want to modify the webhook server for testing purposes, be sure to set and export
the shell environment variable `IMAGE` to an image tag for which you have push access. You can then
build and push the image by running `make push-image`. Also make sure to change the image tag
in `deployment/deployment.yaml.template`, and if necessary, add image pull secrets.
