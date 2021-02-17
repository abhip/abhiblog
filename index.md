## Welcome to my blog

I would be posting quick summary of things which i learned 

### How to get core dump for a process running inside docker container

The process to collect core dump for a process inside docker container seems to have some additional steps compared to taking core dump directly on the host.

1.  In general when we want to collect core dump, we will 
    - Set the maximum size of a core dump to unlimited by running ```ulimit -c unlimited ```
    - Set where core dumps will be written by running ```sysctl -w kernel.core_pattern=/tmp/core-%e.%p.%h.%t ``` Here core pattern will be written to tmp directory.
2.  Now if when we try to run above command to set where the core dump will be written inside a container, you may see the message **sysctl: setting key "kernel.core_pattern": Read-only file system** This is because we are not running the container in privileged mode. In order to run the container in privileged mode, one can use the following command 
```
docker run --ulimit core=-1 -dit --privileged --network host --name test haproxy:hatetesting
```
In above command, we pass **--privileged** flag to run the container in privileged mode and **--ulimit core=-1** to set maximum size of core dump to unlimited. If you now ```sysctl -w kernel.core_pattern=/tmp/core-%e.%p.%h.%t``` inside the container it will work without any issue. Important thing one should be aware of is that when you set the core pattern inside the container, it will also change on the host. So you may want to change back the core pattern location on the host after collecting the core dump for the process. Another note, enabling privileged mode comes with some security concerns, please refer to the documentation before enabling it. 

3.  If you want by default containers which are started by docker demon to have privileged flag true and ulimit set to unlimited then you can modify your docker demon config file to have these flags/options enabled. Refer to https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file
4.  Inside the container, you can test if the core dump are getting created by running ```kill -11 $pid ```

### Links
1.  Know more about core dumps - https://jvns.ca/blog/2018/04/28/debugging-a-segfault-on-linux/
2.  Discussion around collecting core dumps inside docker container - https://github.com/moby/moby/issues/11740
