
<br/>

# **An ansible note**

<br/>
<br/>

# PostgreSQL-11 高可用主从模式 基于[pgpool官方example](https://www.pgpool.net/docs/latest/en/html/example-cluster.html)与`ansible`工具实现自动化
# ![Cluster System Configuration](https://www.pgpool.net/docs/latest/en/html/cluster_40.gif)

<br/>

# 支持离线安装
**一、将postgresql-11 rpm包下载至down/postgres-11，目录自行创建**

**二、将pgpool-ii rpm包下载至down/pgpool-ii，目录自行创建**

<br/>

# 快速指南
- **拉取仓库**
``` bash
cd /root && git clone https://github.com/shensw4/ansible-pgpool.git
```

- **安装pip** 
``` bash
# 已安装的请跳过
curl -sSl https://bootstrap.pypa.io/get-pip.py | python -
```
- **安装ansible, sshpass**
``` bash
pip install ansible==2.8.7
# 国内用户建议用aliyun镜像加速
pip install ansible==2.8.7 -i https://mirrors.aliyun.com/pypi/simple/

# 如果已经对节点分发好SSH Key的请跳过
yum install -y sshpass
```

- **设置ansible配置文件变量**
``` bash
export ANSIBLE_CONFIG=/root/ansible/ansible_pgsql.cfg
```

- **分发SSH Key** 
``` bash
# 如果主备服务器密码一致，使用如下
ansible-playbook -k generate-key-use-password.yaml

# 如果主备节点密码不一致，则为每台节点手动生成
ssh-keygen -t rsa -b 2048
ssh-copy-id -i ~/.ssh/id_rsa.pub root@server1
ssh-copy-id -i ~/.ssh/id_rsa.pub root@server2
ssh-copy-id -i ~/.ssh/id_rsa.pub root@server3 # ...
```

- **编辑ansible inventory文件**
``` bash
# 修改pgsqlPrimary，pgsqlStandby 的IP地址，端口
# 修改pgsqlHA的CIDR，VIP，密码等
cp -a hosts-pgsql-ha.example hosts-pgsql-ha
vim hosts-pgsql-ha
```

- **开始部署**
``` bash
ansible-playbook postgresql-ha.yaml
```

- **查看节点状态**
``` bash
psql -h 'yourVIP' -p 'yourPgpoolPort' -U pgpool postgres -c "show pool_nodes"
```

- **恢复指定node**
``` bash
# 因为第一次部署，除主节点外其他节点都未启动，需要同步数据到其他节点
# 节点故障也可使用该命令恢复启动节点，恢复时间随数据容量增大

su - postgres
pcp_recovery_node -h 'yourVIP' -p 9898 -U pgpool -n 1
``` 

