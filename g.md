# Question 16

run
```
kubctl config use-context k8s
```

## Switch namespace to `mars`

## Implementing the Adapter Pattern pod, Create a new Pod in a YAML file named `adapter.yaml`. The Pod declares two containers. The container `app` uses the image `busybox` and runs the command `while true; do echo "$(date) | $(du -sh ~)" >> /var/logs/diskspace.txt; sleep 5; done;`. The adapter container `transformer` uses the image `busybox` and runs the command `sleep 20; while true; do while read LINE; do echo "$LINE" | cut -f2 -d"|" >> $(date +%Y-%m-%d-%H-%M-%S)-transformed.txt; done < /var/logs/diskspace.txt; sleep 20; done;` to strip the log output off the date for later consumption my a monitoring tool. Be aware that the logic does not handle corner cases (e.g. automatically deleting old entries) and would look different in production systems. Before creating the Pod, define an `emptyDir` volume. Mount the volume in both containers with the path `/var/logs`.Create the Pod, log into the container `transformer`. The current directory should continuously write a new file every 20 seconds.

<details><summary>Show Solution</summary>
<p>

```bash
kubectl run adapter --image=busybox --restart=Never -o yaml --dry-run -- /bin/sh -c 'while true; do echo "$(date) | $(du -sh ~)" >> /var/logs/diskspace.txt; sleep 5; done;' > adapter.yaml
```
The final Pod YAML file should look something like this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  name: adapter
spec:
  volumes:
    - name: config-volume
      emptyDir: {}
  containers:
  - args:
    - /bin/sh
    - -c
    - 'while true; do echo "$(date) | $(du -sh ~)" >> /var/logs/diskspace.txt; sleep 5; done;'
    image: busybox
    name: app
    volumeMounts:
      - name: config-volume
        mountPath: /var/logs
    resources: {}
  - image: busybox
    name: transformer
    args:
    - /bin/sh
    - -c
    - 'sleep 20; while true; do while read LINE; do echo "$LINE" | cut -f2 -d"|" >> $(date +%Y-%m-%d-%H-%M-%S)-transformed.txt; done < /var/logs/diskspace.txt; sleep 20; done;'
    volumeMounts:
      - name: config-volume
        mountPath: /var/logs
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```


```bash
$ kubectl exec adapter --container=transformer -it -- /bin/sh
/ # ls -l
-rw-r--r--    1 root     root           205 May 12 20:43 2019-05-12-20-43-32-transformed.txt
-rw-r--r--    1 root     root           369 May 12 20:43 2019-05-12-20-43-52-transformed.txt
...
/ # cat 2019-05-12-20-43-52-transformed.txt
 4.0K	/root
 4.0K	/root
 4.0K	/root
 4.0K	/root
 4.0K	/root
 4.0K	/root
 4.0K	/root
 4.0K	/root
 4.0K	/root
 4.0K	/root
 4.0K	/root
 4.0K	/root
 4.0K	/root
 4.0K	/root
 4.0K	/root
 4.0K	/root
 4.0K	/root
/ # exit
```
This particular section needs additional detail as these concepts are not covered that well via the tasks provided at kubernetes.io. Actually, the best coverage (for sidecars) is in the concepts section under logging architecture.

    One or more containers running within a pod for enhancing the main container functionality (logger container, git synchronizer container); These are sidecar container

    One or more containers running within a pod for accessing external applications/servers (Redis cluster, memcache cluster); These are called ambassador container

    One or more containers running within a pod to allow access to application running within the container (Monitoring container); These are called as adapter containers-

https://github.com/DhruvBansal86/EFK

https://medium.com/@goldbrickdhruv/efk-elasticsearch-fluentd-kibana-on-aks-azure-kubernetes-service-158b770501a3

</p>
</details>