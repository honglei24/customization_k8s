# 通过pod的metadata.annotations来修改容器的IO速率
key只可以为BlkioDeviceReadIOps，BlkioDeviceWriteIOps，BlkioDeviceReadBps，BlkioDeviceWriteBps。
value格式为列表形式，以逗号分隔，形如："device1:rate1,device2:rate2"。

##### 使用例子.

```
$ cat busybox.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
  annotations:
    BlkioDeviceReadIOps: "/dev/sda:100"
    BlkioDeviceWriteBps: "/dev/sda:200"
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

