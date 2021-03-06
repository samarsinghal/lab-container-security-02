# Kubesec

Security risk analysis for Kubernetes resources
Open Source 
Opinionated ! Fixed set of rules (Security Best Practices)

## Live demo

[Visit Kubesec.io](https://kubesec.io)

This uses ControlPlane's hosted API at [v2.kubesec.io/scan](https://v2.kubesec.io/scan)

There are many ways to run it Kubesec is available as a:

- [Docker container image](https://hub.docker.com/r/kubesec/kubesec/tags) at `docker.io/kubesec/kubesec:v2`
- Linux/MacOS/Win binary (get the [latest release](https://github.com/controlplaneio/kubesec/releases))
- [Kubernetes Admission Controller](https://github.com/controlplaneio/kubesec-webhook)
- [Kubectl plugin](https://github.com/controlplaneio/kubectl-kubesec)


Change to the `~/kubesec` sub directory.

```execute
clear
cd ~/kubesec
```

```execute
cat deployment.yaml
```


#### Docker usage:

Run kubesec in Docker:

```execute
$ docker run -i kubesec/kubesec:latest scan /dev/stdin < deployment.yaml
```

## Kubesec-as-a-Service

Kubesec is also available via HTTPS at [v2.kubesec.io/scan](https://v2.kubesec.io/scan)

#### Command line usage:

```execute
$ curl -sSX POST --data-binary @"deployment.yaml" https://v2.kubesec.io/scan
```

## Example output

Kubesec returns a JSON array, and can scan multiple YAML documents in a single input file.

```json
[
  {
    "object": "Deployment/nginx.default",
    "valid": true,
    "fileName": "STDIN",
    "message": "Passed with a score of 0 points",
    "score": 0,
    "scoring": {
      "advise": [
        {
          "id": "ApparmorAny",
          "selector": ".metadata .annotations .\"container.apparmor.security.beta.kubernetes.io/nginx\"",
          "reason": "Well defined AppArmor policies may provide greater protection from unknown threats. WARNING: NOT PRODUCTION READY",
          "points": 3
        },
        {
          "id": "ServiceAccountName",
          "selector": ".spec .serviceAccountName",
          "reason": "Service accounts restrict Kubernetes API access and should be configured with least privilege",
          "points": 3
        },
        {
          "id": "SeccompAny",
          "selector": ".metadata .annotations .\"container.seccomp.security.alpha.kubernetes.io/pod\"",
          "reason": "Seccomp profiles set minimum privilege and secure against unknown threats",
          "points": 1
        },
        {
          "id": "LimitsCPU",
          "selector": "containers[] .resources .limits .cpu",
          "reason": "Enforcing CPU limits prevents DOS via resource exhaustion",
          "points": 1
        },
        {
          "id": "RequestsMemory",
          "selector": "containers[] .resources .limits .memory",
          "reason": "Enforcing memory limits prevents DOS via resource exhaustion",
          "points": 1
        },
        {
          "id": "RequestsCPU",
          "selector": "containers[] .resources .requests .cpu",
          "reason": "Enforcing CPU requests aids a fair balancing of resources across the cluster",
          "points": 1
        },
        {
          "id": "RequestsMemory",
          "selector": "containers[] .resources .requests .memory",
          "reason": "Enforcing memory requests aids a fair balancing of resources across the cluster",
          "points": 1
        },
        {
          "id": "CapDropAny",
          "selector": "containers[] .securityContext .capabilities .drop",
          "reason": "Reducing kernel capabilities available to a container limits its attack surface",
          "points": 1
        },
        {
          "id": "CapDropAll",
          "selector": "containers[] .securityContext .capabilities .drop | index(\"ALL\")",
          "reason": "Drop all capabilities and add only those required to reduce syscall attack surface",
          "points": 1
        },
        {
          "id": "ReadOnlyRootFilesystem",
          "selector": "containers[] .securityContext .readOnlyRootFilesystem == true",
          "reason": "An immutable root filesystem can prevent malicious binaries being added to PATH and increase attack cost",
          "points": 1
        },
        {
          "id": "RunAsNonRoot",
          "selector": "containers[] .securityContext .runAsNonRoot == true",
          "reason": "Force the running image to run as a non-root user to ensure least privilege",
          "points": 1
        },
        {
          "id": "RunAsUser",
          "selector": "containers[] .securityContext .runAsUser -gt 10000",
          "reason": "Run as a high-UID user to avoid conflicts with the host's user table",
          "points": 1
        }
      ]
    }
  }
]
```

Check the returned non-zero status code is the score is not greater than 10. 

```execute
$ docker run -i kubesec/kubesec:latest scan /dev/stdin < deployment.yaml | grep score
```

Let make some changes to increase the score. 

As advised we added the following to the deployment file 

    securityContext:
        readOnlyRootFilesystem: true
        runAsNonRoot: true
        runAsUser: 20000

    resources:
        requests:
        memory: "64Mi"
        cpu: "250m"
        limits:
        memory: "128Mi"
        cpu: "500m"
        
    serviceAccountName: notDefault

Check the updated deployment file. 

```execute t2
cat updated_deployment.yaml
```

Run kubesec on updated deployment file:

```execute
$ docker run -i kubesec/kubesec:latest scan /dev/stdin < updated_deployment.yaml
```

check the new score 

```execute
$ docker run -i kubesec/kubesec:latest scan /dev/stdin < updated_deployment.yaml | grep score
```