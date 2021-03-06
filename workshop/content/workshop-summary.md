This rounds out this workshop on how to set up a container image which is portable to different container runtimes and is secure.

Important to understand is that although containers provide a way of isolating application processes from being able to access the underlying host, they should not be the only mechanism you rely on to protect yourself against malicious actors.

You should always design container images so as to be able to be run as a non `root` user if there is no need to have any special privileges. Better still, design the container image to run as an arbitrary user ID to accomodate container platforms that may force applications to run as different user IDs. Finally, if the container platform doesn't itself enforce it, always drop any capabilities from containers that you do not need, such as kernel level abilities to become the `root` user.
