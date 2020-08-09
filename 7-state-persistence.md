# State Persistence (8%)
## Switch namespace to `saturn`
## Defining and Mounting a PersistentVolume

1. Create a Persistent Volume named `pv`, access mode `ReadWriteMany`, storage class name `shared`, 512MB of storage capacity and the host path `/data/config`.
2. Create a Persistent Volume Claim named `pvc` that requests the Persistent Volume in step 1. The claim should request 256MB. Ensure that the Persistent Volume Claim is properly bound after its creation.
3. Mount the Persistent Volume Claim from a new Pod named `app` with the path `/var/app/config`. The Pod uses the image `nginx`.
4. Check the events of the Pod after starting it to ensure that the Persistent Volume was mounted properly.

<details><summary>Show Solution</summary>
<p>

Create a YAML file for the Persistent Volume and create it with the command `kubectl create` command.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv
spec:
  capacity:
    storage: 512m
  accessModes:
    - ReadWriteMany
  storageClassName: shared
  hostPath:
    path: /data/config
```

You will see that the Persistent Volume has been created but and is available to be claimed.

```bash
$ kubectl get pv
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv     512m       RWX            Retain           Available           shared                  4s
```

Create a YAML file for the Persistent Volume Claim and create it with the command `kubectl create` command.

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 256m
  storageClassName: shared
```

You will see that the Persisten Volume Claim has been created and has been bound to the Persisten Volume.

```bash
$ kubectl get pvc
NAME   STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc    Bound    pv       512m       RWX            shared         2s

$ kubectl get pv
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM         STORAGECLASS   REASON   AGE
pv     512m       RWX            Retain           Bound    default/pvc   shared                  1m
```

Create a YAML file for the Pod and create it with the command `kubectl create` command.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: app
  name: app
spec:
  containers:
  - image: nginx
    name: app
    volumeMounts:
      - mountPath: "/var/app/config"
        name: configpvc
    resources: {}
  volumes:
    - name: configpvc
      persistentVolumeClaim:
        claimName: pvc
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

You can check the events of a Pod with the `kubectl describe` command. You should see an entry that indicates the successful mount.

```bash
$ kubectl describe pod app
...
Events:
  Type    Reason                 Age   From                         Message
  ----    ------                 ----  ----                         -------
  Normal  Scheduled              16s   default-scheduler            Successfully assigned app to docker-for-desktop
  Normal  SuccessfulMountVolume  16s   kubelet, docker-for-desktop  MountVolume.SetUp succeeded for volume "pv"
  Normal  SuccessfulMountVolume  16s   kubelet, docker-for-desktop  MountVolume.SetUp succeeded for volume "default-token-fsmmp"
  Normal  Pulling                15s   kubelet, docker-for-desktop  pulling image "nginx"
  Normal  Pulled                 14s   kubelet, docker-for-desktop  Successfully pulled image "nginx"
  Normal  Created                14s   kubelet, docker-for-desktop  Created container
  Normal  Started                13s   kubelet, docker-for-desktop  Started container
```

</p>
</details>

## Defining and Mounting a PersistentVolume More Challenged.

1. Create busybox pod with two containers, each one will have the image busybox and will run the 'sleep 3600' command. Make both containers mount an emptyDir at '/etc/foo'. Connect to the second busybox, write the first column of '/etc/passwd' file to '/etc/foo/passwd'. Connect to the first busybox and write '/etc/foo/passwd' file to standard output.

2. Create a PersistentVolume of 10Gi, called 'myvolume'. Make it have accessMode of 'ReadWriteOnce' and 'ReadWriteMany', storageClassName 'normal', mounted on hostPath '/etc/foo'. Save it on pv.yaml, add it to the cluster. Show the PersistentVolumes that exist on the cluster.

3. Create a PersistentVolumeClaim for this storage class, called mypvc, a request of 4Gi and an accessMode of ReadWriteOnce, with the storageClassName of normal, and save it on pvc.yaml. Create it on the cluster. Show the PersistentVolumeClaims of the cluster. Show the PersistentVolumes of the cluster

4. Create a busybox pod with command 'sleep 3600', save it on pod.yaml. Mount the PersistentVolumeClaim to '/etc/foo'. Connect to the 'busybox' pod, and copy the '/etc/passwd' file to '/etc/foo/passwd'.

5. Create a second pod which is identical with the one you just created (you can easily do it by changing the 'name' property on pod.yaml). Connect to it and verify that '/etc/foo' contains the 'passwd' file. Delete pods to cleanup

6. Create a busybox pod with 'sleep 3600' as arguments. Copy '/etc/passwd' from the pod to your local folder

<details><summary>Show Solution</summary>
<p>


</p>
</details>