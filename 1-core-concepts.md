


# Core Concepts (13%)
## Creating namespaces
1. ckad-core
2. ckad-configuration
3. ckad-multi-container
4. ckad-observability
5. ckad-pod-design
6. ckad-service-network
7. ckad-state-persistent


## Creating a Pod and Inspecting it

1. Switch namespace to `ckad-core`.
2. In the namespace `ckad-core` create a new Pod named `mypod` with the image `nginx:2.3.5`. Expose the port 80. set
   environment of the pod TESTER=Michael Xue
3. Identify the issue with creating the container. Write down the root cause of issue in a file named `pod-error.txt`.
4. Change the image of the Pod to `nginx:1.15.12`. and use jsonfile to check if image has been updated.
5. List the Pod and ensure that the container is running.
6. Log into the container and run the `ls` command. Write down the output. Log out of the container.
7. Retrieve the IP address of the Pod `mypod`.
8. Run a temporary Pod using the image `busybox`, shell into it and run a `wget` command against the `nginx` Pod using port 80.
9. Render the logs of Pod `mypod`.

## edit a pod using replace

1. Create a pod mypod2 using image nginx, when the pod is up and running change the image from nginx to nginx:latest


<details><summary>Show Solution</summary>
<p>

First, create the namespace.

```bash
$ kubectl create namespace ckad-core
```

Next, create the Pod in the new namespace.

```bash
$ kubectl run mypod --image=nginx:2.3.5 --restart=Never --port=80 --env=TESTER="Michael Xue" --namespace=ckad-core
pod/mypod created
```

You will see that the image cannot be pulled as it doesn't exist with this tag.

```bash
$ kubectl get pod -n ckad-core
NAME    READY   STATUS             RESTARTS   AGE
mypod   0/1     ImagePullBackOff   0          1m
```

The list of events can give you a deeper insight. redirect it to text file, pay attention to "Upper case E for Error"

```bash
$ kubectl describe pod -n ckad-core | grep Error>
...
Events:
  Type     Reason                 Age                 From                         Message
  ----     ------                 ----                ----                         -------
  Normal   Scheduled              3m3s                default-scheduler            Successfully assigned mypod to docker-for-desktop
  Normal   SuccessfulMountVolume  3m2s                kubelet, docker-for-desktop  MountVolume.SetUp succeeded for volume "default-token-jbcl6"
  Normal   Pulling                84s (x4 over 3m2s)  kubelet, docker-for-desktop  pulling image "nginx:2.3.5"
  Warning  Failed                 83s (x4 over 3m1s)  kubelet, docker-for-desktop  Failed to pull image "nginx:2.3.5": rpc error: code = Unknown desc = Error response from daemon: manifest for nginx:2.3.5 not found
  Warning  Failed                 83s (x4 over 3m1s)  kubelet, docker-for-desktop  Error: ErrImagePull
  Normal   BackOff                69s (x6 over 3m)    kubelet, docker-for-desktop  Back-off pulling image "nginx:2.3.5"
  Warning  Failed                 69s (x6 over 3m)    kubelet, docker-for-desktop  Error: ImagePullBackOff
```

Go ahead and edit the existing Pod. Alternatively, you could also just use the `kubectl set image pod mypod mypod=nginx --namespace=ckad-core` command.

```bash
$ kubectl edit pod mypod --namespace=ckad-core
```

After setting an image that does exist, the Pod should render the status `Running`.

```
kubectl get po nginx -o jsonpath='{.spec.containers[].image}{"\n"}'
```

```bash
$ kubectl get pod -n ckad-core
NAME    READY   STATUS    RESTARTS   AGE
mypod   1/1     Running   0          14m
```

You can shell into the container and run the `ls` command.

```bash
$ kubectl exec mypod -it --namespace=ckad-core  -- /bin/sh
/ # ls
bin  boot  dev	etc  home  lib	lib64  media  mnt  opt	proc  root  run  sbin  srv  sys  tmp  usr  var
/ # exit
```

Retrieve the IP address of the Pod with the `-o wide` command line option.

```bash
$ kubectl get pods -o wide -n ckad-core
NAME    READY   STATUS    RESTARTS   AGE   IP               NODE
mypod   1/1     Running   0          12m   192.168.60.149   docker-for-desktop
```

Remember to use the `--rm` to create a temporary Pod.

```bash
$ kubectl run busybox --image=busybox --rm -it --restart=Never -n ckad-core -- /bin/sh
If you don't see a command prompt, try pressing enter.
/ # wget -O- 192.168.60.149:80
Connecting to 192.168.60.149:80 (192.168.60.149:80)
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
-                    100% |**********************************************************************|   612  0:00:00 ETA
/ # exit
```

The logs of the Pod should show a single line indicating our request.

```bash
$ kubectl logs mypod -n ckad-core
192.168.60.162 - - [17/May/2019:13:35:59 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
```

</p>
</details>


Migrating from imperative commands to imperative object configuration 
Migrating from imperative commands to imperative object configuration involves several manual steps.

Export the live object to a local object configuration file:

kubectl get <kind>/<name> -o yaml > <kind>_<name>.yaml
Manually remove the status field from the object configuration file.

For subsequent object management, use replace exclusively.

kubectl replace -f <kind>_<name>.yaml