### Seccomp and AppArmor

Runtime security provides active protection for your containers while they're running. The idea is to detect and/or prevent malicious activity from occuring inside the container. With secure computing (seccomp) you can prevent a containerized application from making certain syscalls to the underlying host operating system's kernel. While the Linux operating system has a few hundred system calls, the lion's share of them are not necessary for running containers. By restricting what syscalls can be made by a container, you can effectively decrease your application's attack surface. 

On occasion, an application will need to access a kernel resource that requires special privileges normally granted only to the root user. However, running the application as a user with root privileges is a bad solution as it provides the application with access to the entire system. Instead, the kernel provides a set of capabilities that can be granted to a process to allow it coarse-grained access to only the kernel resources it needs and nothing more.

Using kernel modules such as AppArmor, Kubernetes provides an easy interface to both run the containerized application as a non-root user in the process namespace and restrict the set of capabilities granted to the process.

Unlike SELinux, seccomp was not designed to isolate containers from each other, however, it will protect the host kernel from unauthorized syscalls. It works by intercepting syscalls and only allowing those that have been whitelisted to pass through. Docker has a default seccomp profile which is suitable for a majority of general purpose workloads. 

Change to the `~/seccomp` sub directory.

```execute
clear
cd ~/seccomp
```

There are a couple of ways to run Docker container with a Seccomp profile, either you can run a docker container with the default profile through the command line, or specify a specific custom profile in .json format, or you can specify your Seccomp profile in Daemon configuration file.

Default Seccomp Profile:

If you didn’t specify a Seccomp profile, then Docker uses built-in Seccomp and Kernel configuration. To check this, you can run the following command:

```execute
docker run -it --rm --name alpine-test alpine /bin/sh
```

Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
e6b0cf9c0882: Pull complete 
Digest: sha256:2171658620155679240babee0a7714f6509fae66898db422ad803b951257db78
Status: Downloaded newer image for alpine:latest
To see if Seccomp filter is loaded inside the Alpine container:

```execute
grep Seccomp /proc/$$/status
```

Seccomp:	2
Here you have grep the pseudo-filesystem to access the kernel data, which indicates that default Seccomp filter was applied inside the running container.

Custom Seccomp Profile:

There are scenarios where you want to override the default profile, which gives you extra control and capabilities to play with. In the following example, you are going to use your own custom profile.

You can download Docker’s official default profile "https://raw.githubusercontent.com/docker/labs/master/security/seccomp/seccomp-profiles/default.json" and make changes to it:

Once you have the .json file with you, it’s time to make changes to the file your-custom-profile.json. For this guide, we are simply going to restrict mkdir command inside the container. To do this, we remove all the references to the mkdir in the file, which could look something like this:

... 	
	{
			"name": "mkdir",
			"action": "SCMP_ACT_ALLOW",
			"args": []
	},
... 
we edit the profile, try to run the Docker container with seccomp-profile-no-mkdir.json:

```execute
docker run -it --rm --security-opt seccomp=/path/to/file/your-custom-profile.json --name alpine-custom-seccomp alpine /bin/sh
```

Verify mkdir operation on container

```execute
mkdir test
```

We should see mkdir: can't create directory 'test': Operation not permitted

Exit container 
```execute
exit
```

The newly created profile container doesn’t allow you to run the mkdir syscall.

This way, you can create your own custom profile and make changes to it as per your requirement, as there are a number of opreations which you can restrict like chown, chmod and so on..

Run Container without Seccomp:

For some reason, if you wish to run a container without Seccomp profile, then you can override this by using --security-opt flag with unconfined flag:

```execute
docker run -it --rm --security-opt seccomp=unconfined --name alpine-wo-seccomp alpine /bin/sh
```

To see if your docker container runs without Seccomp profile, use this:

```execute
grep Seccomp /proc/$$/status
```

Seccomp:	0
You will see Seccomp: 0, which means the container is running without the default Seccomp profile. Note that it is not recommended to do unless you know what you are doing, as a certain security loop-hole will be wide open and can be exploited.


```execute
exit
```

AppArmor is similar to seccomp, only it restricts an container's capabilities including accessing parts of the file system. It can be run in either enforcement or complain mode.

You can configure your container or Pod to use this profile by adding the following annotation to your container's or Pod's spec (pre-1.19):


annotations:
  seccomp.security.alpha.kubernetes.io/pod: "runtime/default"

1.19 and later:


securityContext:
  seccompProfile:
    type: RuntimeDefault

It's also possible to create your own profiles for things that require additional privileges.

Kubernetes does not currently provide any native mechanisms for loading AppArmor or seccomp profiles onto Nodes. They either have to be loaded manually or installed onto Nodes when they are bootstrapped. This has to be done prior to referencing them in your Pods because the scheduler is unaware of which nodes have profiles.

### Consider add/dropping Linux capabilities before writing seccomp policies

Capabilities involve various checks in kernel functions reachable by syscalls. If the check fails, the syscall typically returns an error. The check can be done either right at the beginning of a specific syscall, or deeper in the kernel in areas that might be reachable through multiple different syscalls (such as writing to a specific privileged file). Seccomp, on the other hand, is a syscall filter which is applied to all syscalls before they are run. A process can set up a filter which allows them to revoke their right to run certain syscalls, or specific arguments for certain syscalls.

Before using seccomp, consider whether adding/removing Linux capabilities gives you the control you need. See https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-capabilities-for-a-container for further information.



