# 通过pod的metadata.annotations来修改容器的basesize

##### 使用例子.

```
$ cat busybox.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
  annotations:
    storageOptSize: "20G"
spec:
  containers:
  - command:
    - sleep
    - "36000"
    image: busybox
    resources:
      limits:
        cpu: "1"
        memory: 512Mi
      requests:
        cpu: "0.8"
        memory: 250Mi
    name: busybox

$ kubectl create -f busybox.yaml
```

