# docker_security

1. AppArmor (on ubuntu/debian system)

        docker run --security-opt apparmor=profile_name <container_name>
2. SELinux (on REHL sysyem)

        docker run --security-opt label=type:container_t <container_name>

3. Seccomp (Secure Computing Mode)

        docker run --security-opt seccomp=/path/to/custom-profile.json <container_name>      

4. Linux Capabilities

        docker run --cap-drop=NET_ADMIN --cap-add=SYS_TIME <container-name>

5. Docker user Namespace

        vi /etc/docker/daemon.json
        "userns-remap": "default" 

6. Docker Content Trust (DCT)

        export DOCKER_CONTENT_TRUST=1

# Seccomp (Secure Computing Mode) in Docker

> Seccomp (Secure Computing Mode) is a Linux kernel feature that restricts a container's ability to make system calls (syscalls) to the host kernel. Docker uses Seccomp to limit the attack surface by blocking unnecessary or dangerous syscalls.

> https://docs.docker.com/engine/security/seccomp/

> https://kubernetes.io/docs/tutorials/security/seccomp/ # for k8s  

       docker info # check seccomp configuration
        vi seccomp-profile.json
```
{
  "defaultAction": "SCMP_ACT_ALLOW",
  "architectures": [
    "SCMP_ARCH_X86_64",
    "SCMP_ARCH_X86",
    "SCMP_ARCH_X32"
  ],
  "syscalls": [
    {
      "name": "chmod",
      "action": "SCMP_ACT_ERRNO",
      "args": []
    }
  ]
}

```
```
docker run -it --name my-container --security-opt seccomp=seccomp-profile.json alpine sh
        
cat /etc/os-release 

touch /tmp/file.sh
ls -lh /tmp
chmod 400 /tmp/file.txt # operation not permitted bcos we have already permition denied  chmod cmd

chown nobody /tmp/file.sh
ls -lh /tmp # owner change root to nobody
apk add strace # install strace cmd

strace chmod 400 /tmp/file.sh  # operation not permitted bcos we have already permition denied  chmod cmd


strace chown root /tmp/file.sh # no error 
  
```
# Linux Capabilities
> Linux capabilities provide granular control over the privileges available to processes, offering a more fine-grained approach than the traditional "root or non-root" binary model. Docker allows you to manage these capabilities for containers.

> https://dockerlabs.collabnix.com/advanced/security/capabilities/
```
docker  run -it  --name new-container alpine sh

apk add -U libcap # install capsh cmd

capsh --print
```
```
echo "this is a file" > /tmp/nwefile.txt

ls -lh /tmp/newfile.txt

chown nobody /tmp/newfile.txt
```
```
docker run -itd --cap-drop CHOWN  alpine
docker ps
docker exec -it <conatiner_name> sh

apk add -U libcap # install capsh cmd

capsh --print

echo "hi" > /tmp/file.txt
ls -lh /tmp/
chown nobody /tmp/file.txt #  Operation not permitted bcos we have already  denied capabilities chowm cmd
```
```
docker stop $(docker ps -aq)

```
```
docker run -itd --cap-drop ALL --cap-add chown alpine # drop all capabilities excpt chown

docker ps

docker exec -it <container_name> sh
apk add -U libcap # install capsh cmd

capsh --print # show only chown capability in cureent 

```





