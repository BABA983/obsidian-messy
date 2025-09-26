```bash
# 创建文件夹，我放在了用户目录的 Portable 目录下
mkdir ~/Portable && cd ~/Portable
mkdir -p redis-cluster/{7000..7005}

# 复制几份配置文件供集群实例使用
for port in {7000..7005}; do
    cp /opt/homebrew/etc/redis.conf redis-cluster/$port 
done

# 为配置文件追加集群相关配置
for port in {7000..7005}; do
    cat >> redis-cluster/$port/redis.conf <<EOF 
port $port  
cluster-enabled yes
cluster-config-file nodes-$port.conf 
cluster-node-timeout 5000
appendonly yes 
EOF
done

# 启动实例
for port in {7000..7005}; do
    redis-server redis-cluster/$port/redis.conf --daemonize yes --pidfile /tmp/redis-cluster-$port.pid
done

# 初始化
redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 \
127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1 < <(yes "yes")

```
## 链接集群
```bash
redis-cli -c -p 7000
info cluster
```
## 启动集群
```bash
for port in {7000..7005}; do redis-server redis-cluster/$port/redis.conf --daemonize yes --pidfile /tmp/redis-cluster-$port.pid done
```
## 停止集群
```bash
# pick one
for port in {7000..7005}; do kill `cat /tmp/redis-cluster-$port.pid` done
for port in {7000..7005}; do redis-cli -p $port shutdown done
```
## Reference
https://blog.yasking.org/a/macos-install-redis-cluster.html