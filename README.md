# 支持lxcfs

## 前提条件
#### 安装lxcfs
```
yum install lxcfs
```

#### 修改lxcfs.service文件
```
[Unit]
Description=FUSE filesystem for LXC
ConditionVirtualization=!container
Before=lxc.service
Documentation=man:lxcfs(1)

[Service]
ExecStart=/usr/bin/lxcfs /var/lib/lxc/lxcfs/
KillMode=process
Restart=always
#ExecStopPost=-/bin/fusermount -u /var/lib/lxc/lxcfs
Delegate=yes
ExecStartPost=/usr/local/bin/remount_lxcfs.sh

[Install]
WantedBy=multi-user.target
```

启动lxcfs
```
# mkdir -p /var/lib/lxc/lxcfs/
# systemctl daemon-reload
# systemctl restart lxcfs
# systemctl enable lxcfs
```

#### 修改/usr/local/bin/remount_lxcfs.sh
改脚本实现了lxcfs服务重启时候的重新挂载
```
#!/bin/sh

LXCFS="/var/lib/lxc/lxcfs"

containers=$(docker ps --format {{.Names}}|grep -v "k8s_POD")
for c in $containers;do
	echo "Remounting lxcfs for $c"
	PID=$(docker inspect $c --format '{{.State.Pid}}')
	for file in meminfo cpuinfo stat uptime swaps diskstats;do
                nsenter --target ${PID} --mount test -e $LXCFS/proc/$file
                [ $? -eq 0 ] || continue 
		nsenter --target ${PID} --mount -- mount -B "$LXCFS/proc/$file" "/proc/$file"
	done
done
```

## 使用方法：
定义pod的时候在pod注解里加入io.kubernetes.container.lxcfsEnable，示例如下
```
apiVersion: v1
kind: Pod
metadata:
  name: busybox 
  namespace: default
  annotations:
    io.kubernetes.container.lxcfsEnable: "True"
spec:
  hostNetwork: true
  nodeSelector:
    kubernetes.io/hostname: kube-node03
  containers:
  - command:
    - sleep
    - "36000"
    image: centos:latest 
    resources:
      limits:
        cpu: "1"
        memory: 512Mi
      requests:
        cpu: "1"
        memory: 512Mi
    name: busybox
```

## 实现原理
在pod注解里指定io.kubernetes.container.lxcfsEnable: "True"的时候，在容器启动的时候挂载了下列目录，实现了资源隔离的可见性。
```
-v /var/lib/lxc/:/var/lib/lxc/:shared  \
-v /var/lib/lxc/lxcfs/proc/uptime:/proc/uptime \
-v /var/lib/lxc/lxcfs/proc/swaps:/proc/swaps  \
-v /var/lib/lxc/lxcfs/proc/stat:/proc/stat  \
-v /var/lib/lxc/lxcfs/proc/diskstats:/proc/diskstats \
-v /var/lib/lxc/lxcfs/proc/meminfo:/proc/meminfo \
-v /var/lib/lxc/lxcfs/proc/cpuinfo:/proc/cpuinfo
```
参考：https://github.com/pouchcontainer/blog/issues/3
