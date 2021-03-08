# Kubernetes Admission Controller Webhook

Kubernetes admission controllers are plugins that govern and enforce how the cluster is used. They can be thought of as a gatekeeper that intercept (authenticated) API requests and may change the request object or deny the request altogether. The admission control process has two phases: the mutating phase is executed first, followed by the validating phase. Consequently, admission controllers can act as mutating or validating controllers or as a combination of both. 

Change to the `~/admission-controller` sub directory.

```execute
clear
cd ~/admission-controller
```

## Verify

For this demo we deployed a Webhook Server on the cluster - lets verify the webhook-server pod is running

1. The `webhook-server` pod in the `webhook-demo` namespace should be running:

```execute
kubectl -n webhook-demo get pods
```
You should see output like below: 

NAME                             READY     STATUS    RESTARTS   AGE
webhook-server-6f976f7bf-hssc9   1/1       Running   0          35m

If case webhook server is not running. 
Run `./webhook/deploy.sh`. This will create a CA, a certificate and private key for the webhook server,
and deploy the resources in the newly created `webhook-demo` namespace in your Kubernetes cluster.


2. A `MutatingWebhookConfiguration` named `demo-webhook` should exist:

A mutating admission controller webhook is defined by creating a MutatingWebhookConfiguration object in Kubernetes
This configuration defines a webhook webhook-server.webhook-demo.svc, and instructs the Kubernetes API server to consult the service webhook-server in namespace webhook-demo whenever a pod is created by making a HTTP POST request to the /mutate URL.

```execute
kubectl get mutatingwebhookconfigurations
```

NAME                   WEBHOOKS   
cert-manager-webhook   1          
demo-webhook           1          

## Admission Controller Webhook Demo

The logic of this demo webhook is fairly simple: it enforces more secure defaults for running
containers as non-root user. While it is still possible to run containers as root, the webhook
ensures that this is only possible if the setting `runAsNonRoot` is *explicitly* set to `false`
in the `securityContext` of the Pod. If no value is set for `runAsNonRoot`, a default of `true`
is applied, and the user ID defaults to `1234`.


1. Deploy [a pod](examples/pod-with-defaults.yaml) that neither sets `runAsNonRoot` nor `runAsUser`:

A pod with no securityContext specified. Without the webhook, it would run as user root (0). 
The webhook mutates it to run as the non-root user with uid 1234.


```execute
kubectl create -f pod-with-defaults.yaml
```

Verify that the pod has default values in its security context filled in:

```execute
kubectl get pod/pod-with-defaults -o yaml
```
...
  securityContext:
    runAsNonRoot: true
    runAsUser: 1234
...

Also, check the logs that the pod had in fact been running as a non-root user:

```execute
kubectl logs pod-with-defaults
```

output: I am running as user 1234


2. Deploy a pod that explicitly sets `runAsNonRoot` to `false`, allowing it to run as the `root` user:

A pod with a securityContext explicitly allowing it to run as root. The effect of deploying this with and without the webhook is the same. 
The explicit setting however prevents the webhook from applying more secure defaults.


```execute
kubectl create -f pod-with-override.yaml
```

Verify that the pod has default values in its security context filled in:

```execute
kubectl get pod/pod-with-override -o yaml
```

...
  securityContext:
    runAsNonRoot: false
...

Now, check the logs that the pod had in fact been running as a non-root user:

```execute
kubectl logs pod-with-override
```

output: I am running as user 0


3. Attempt to deploy a pod that has a conflicting setting: `runAsNonRoot` set to `true`, but `runAsUser` set to false.

A pod with a conflicting securityContext setting: it has to run as a non-root user, but we explicitly request a user id of 0 (root).
Without the webhook, the pod could be created, but would be unable to launch due to an unenforceable security context leading to it being stuck in a `CreateContainerConfigError` status. 
With the webhook, the creation of the pod is outright rejected.


```execute
kubectl create -f pod-with-conflict.yaml 
```

output: The admission controller should block the creation of that pod.


Error from server (InternalError): error when creating "examples/pod-with-conflict.yaml": Internal error
occurred: admission webhook "webhook-server.webhook-demo.svc" denied the request: runAsNonRoot specified,
but runAsUser set to 0 (the root user)

Cleanup 
```execute
kubectl delete pods pod-with-defaults pod-with-override pod-with-conflict