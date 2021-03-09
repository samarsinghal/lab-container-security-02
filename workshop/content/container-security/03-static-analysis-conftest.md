# Conftest

Conftest helps you write tests against structured configuration data. Using Conftest you can
write tests for your Kubernetes configuration, Tekton pipeline definitions, Terraform code,
Serverless configs or any other config files.

Conftest uses the Rego language from [Open Policy Agent](https://www.openpolicyagent.org/) for writing
the assertions. 


## Open Policy Agent

The Open Policy Agent (OPA) is an open source, general-purpose policy engine that enables unified, context-aware policy enforcement across the entire stack.

## How does OPA work?

OPA gives you a high-level declarative language to author and enforce policies
across your stack.

With OPA, you define _rules_ that govern how your system should behave. These
rules exist to answer questions like:

* Can user X call operation Y on resource Z?
* What clusters should workload W be deployed to?
* What tags must be set on resource R before it's created?

You integrate services with OPA so that these kinds of policy decisions do not
have to be *hardcoded* in your service. Services integrate with OPA by
executing _queries_ when policy decisions are needed.

When you query OPA for a policy decision, OPA evaluates the rules and data
(which you give it) to produce an answer. The policy decision is sent back as
the result of the query.

For example, in a simple API authorization use case:

* You write rules that allow (or deny) access to your service APIs.
* Your service queries OPA when it receives API requests.
* OPA returns allow (or deny) decisions to your service.
* Your service _enforces_ the decisions by accepting or rejecting requests accordingly.

- Try [play.openpolicyagent.org](https://play.openpolicyagent.org) to experiment with OPA policies.


Change to the `~/conftest` sub directory.

```execute
clear
cd ~/conftest
```

Here's a quick example of a policy defined in rego language deployment.rego


```execute
cat policy/deployment.rego
```

```rego
# from https://www.conftest.dev
package main

deny[msg] {
  input.kind = "Deployment"
  not input.spec.template.spec.securityContext.runAsNonRoot = true
  msg = "Containers must not run as root"
}

deny[msg] {
  input.kind = "Deployment"
  not input.spec.selector.matchLabels.app
  msg = "Containers must provide app label for pod selectors"
}
```

Lets looks at a simple Kubernetes deployment in `deployment.yaml`

```execute
cat deployment.yaml
```

Now, We will run Conftest against the deployment.yaml

```execute
conftest test deployment.yaml
```
The output shows, out of two test one passed and one failed because container is running as root. As per policy defination container must not run as root

FAIL - deployment.yaml - main - Containers must not run as root

2 tests, 1 passed, 0 warnings, 1 failure, 0 exceptions


Lets fix the deployment and validate, check the updated-deployment.yaml with added security context 

securityContext 
    runAsNonRoot: true

```execute
cat updated-deployment.yaml
```

Now run Conftest against the updated-deployment.yaml

```execute
conftest test updated-deployment.yaml
```
output 

2 tests, 2 passed, 0 warnings, 0 failures, 0 exceptions

We see conftest OPA in action one thing to note Conftest isn't specific to Kubernetes. It will happily let you write tests for any configuration files in a variety of different formats. Ex - dockerfile.


