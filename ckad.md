## Command
set current ns

```
kubectl config view set-context --current --namespace ckad-prep
kubtctl config view --minify | grep names
```
NOTE :- -e here enables the interpretation of backslash escapes 
        -n : this option is used to omit echoing trailing newline .

#Assign Memmary Resrouces to containaters and POds
###my problem
```
apiVersion: v1
kind: Pod
metadata:
   name: memory-demo
   namespace: mem-example
spec:
   containers:
     - name: memory-demo-crt
       image: polinux/stress
       resources:
          limites:
             - memory: 200mi
          requests:
             - memory: 100mi
       command: ["stress"]
       arg: ["--vm","1","--vm=bytes","150M","--vm-hang","1"]
```
#correct one
 apiVersion: v1
kind: Pod
metadata:
   name: memory-demo
   namespace: mem-example
spec:
   containers:
     - name: memory-demo-crt
       image: polinux/stress
       resources:
          limits:
              memory: "200Mi"
          requests:
              memory: "100Mi"
       command: ["stress"]
       args: ["--vm","1","--vm=bytes","150M","--vm-hang","1"]
```




#＃＃create a pod redis -image mount /data/redis to emptydir, run the pod watch it. get into the pod
cd into dataredis create a file.list all running process. 

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis
  name: redis
spec:
  volumes:
    - name: redis-storage
      emptyDir: {}
  containers:
  - image: redis
    name: redis
    resources: {}
    volumeMounts:
      - name: redis-storage
        mountPath: /data/redis
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```
## To config a configmap with text. 
```
#you must use double quote to separate lines"
echo -e "var1=val1\n# this is a comment\n\nvar2=val2\n#anothercomment" > config.env

### apiVersion: v1
```
##what is wrong with this settings.
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: hello
  name: hello
spec:
  containers:
  - image: bonomat/nodejs-hello-world
    name: hello
    ports:
    - containerPort: 3000
      name: nodejs-port
    resources: {}
    readinessProbe:
        httpGet:
          urlPath: nodejs-port
        initDelaySeconds: 2
    liveneessProbe:
        httpGet:
          urlPath: nodejs-port
        periodSeconds: 8
        activeDeadlineSeconds: 5
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
~                  
```

## what is the difference between activeDeadlineSeconds adn initalDelaySeconds
activeDelaySeconds is for jobs
initialDelaySeconds is for probe
```
#apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-timeout
spec:
  backoffLimit: 5
  activeDeadlineSeconds: 100
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
###
```
### what is wrong with this command line
k annotate po frontend contact=John Doe,commit=2d3mg3
answer
k annotate po frontend contact="John Doe",commit="2d3m

## give me the surranding 4 lines around annotate in yaml
```
#C captailise

k get pod backend -o yaml | grep -C 4 annotat
```

## container port open
```
#pay attention to portS. then containerPort
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never

```
NAME      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
sunpod     ClusterIP   10.0.170.211   <none>        80/TCP          27h
nimabee   NodePort    10.0.226.87    <none>        487:32252/TCP   49m
```

### hard disk mb use small m
```
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
