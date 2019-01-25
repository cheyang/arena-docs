## 前提要求

- Linux 操作系统（CentOS 7）
- 安装有Docker
- kubeconfig文件放置在/root/.kube/config

## 安装步骤

0\. 如果之前有部署过arena, 请先备份一下

```
mv /usr/local/bin/arena /usr/local/bin/arena.old
mv /charts /charts_old
docker rm -f arena
```

1\. 登录Master节点，运行如下`docker run`命令可以将arena的命令行工具和相关组件安装到该节点

```
docker run -itd --name=arena \
                --network host \
                -v /:/host \
                -v /root/.kube/config:/root/.kube/config \
                -e useHostNetwork=true \
                -e KUBECONFIG=/root/.kube/config \
                registry.cn-shanghai.aliyuncs.com/acs/arena:0.1.0-6a0e67c
```

如果要使用监控可以

```
docker run -itd --name=arena \
                --network host \
                -v /:/host \
                -v /root/.kube/config:/root/.kube/config \
                -e useHostNetwork=true \
                -e usePrometheus=true \
                -e platform=ack \
                -e KUBECONFIG=/root/.kube/config \
                registry.cn-shanghai.aliyuncs.com/acs/arena:0.1.0-6a0e67c
```

2\. 检查安装结果, 可以看到退出值

```
# docker ps -a |grep arena
fc94a2a8c095        registry.cn-shanghai.aliyuncs.com/acs/arena:0.1.0-6a0e67c                "/usr/local/bin/ru..."   6 seconds ago       Exited (0) 4 seconds ago                       arena
```

3\. 也可以进一步查看日志

```
# docker logs arena
```

4\. 也可以查看基础组件是否安装成功

```
# kubectl get all -n arena-system
NAME                                            READY     STATUS    RESTARTS   AGE
pod/mpi-operator-6dc94b5975-f574h               1/1       Running   0          12m
pod/tf-job-dashboard-5879874577-q8tcg           1/1       Running   0          12m
pod/tf-job-operator-v1alpha2-7db9f7785f-db7sg   1/1       Running   0          12m

NAME                       TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
service/tf-job-dashboard   NodePort   172.19.6.69   <none>        80:30861/TCP   12m

NAME                                       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mpi-operator               1         1         1            1           12m
deployment.apps/tf-job-dashboard           1         1         1            1           12m
deployment.apps/tf-job-operator-v1alpha2   1         1         1            1           12m

NAME                                                  DESIRED   CURRENT   READY     AGE
replicaset.apps/mpi-operator-6dc94b5975               1         1         1         12m
replicaset.apps/tf-job-dashboard-5879874577           1         1         1         12m
replicaset.apps/tf-job-operator-v1alpha2-7db9f7785f   1         1         1         12m
```

5\. 配置自动补全功能

```
# yum install bash-completion -y
# echo "source <(arena completion bash)" >> ~/.bashrc
# source <(arena completion bash)
```
完成后输入arena后，双击`tab`可以提示命令

6\. 执行`arena top node`检查gpu能力已经添加成功

```
# arena top node
NAME                                IPADDRESS     ROLE    GPU(Total)  GPU(Allocated)
c1b2n3l6szhh8wnbli  192.168.0.46  <none>  1           0
c1b2n3l6szhh8wnblj  192.168.0.45  <none>  1           0
c1b2n3l6szhh8wnblk  192.168.0.44  <none>  1           0
c3r0lhzw300xnp7tsx  192.168.0.41  master  0           0
c9dgdgctmz8z24563y  192.168.0.43  master  0           0
cbs7a8sejzwjjgr2yi  192.168.0.42  master  0           0
-----------------------------------------------------------------------------------------
Allocated/Total GPUs In Cluster:
0/3 (0%)
```

