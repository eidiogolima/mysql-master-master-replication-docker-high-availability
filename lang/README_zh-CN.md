# 🐳 MySQL Master x Master 复制

使用 Docker Compose 和 GTID（全局事务ID）实现的 MySQL 双向复制。

## 🚀 快速开始

### 开发环境
```bash
cd dev/
docker-compose up -d
./setup-replication.sh mysql-master-2
./check-replication.sh
```

### 生产环境
```bash
# Server 1
cd prod/server-1/
docker-compose up -d
cd exec && ./setup-replication.sh 192.168.1.20

# Server 2  
cd prod/server-2/
docker-compose up -d
cd exec && ./setup-replication.sh 192.168.1.10
```

## 📁 目录结构

```
├── dev/                    # 开发环境（本地）
│   ├── docker-compose.yml
│   ├── setup-replication.sh
│   └── check-replication.sh
│
└── prod/                   # 生产环境（独立服务器）
    ├── server-1/           # Master 1 (192.168.1.10)
    │   ├── docker-compose.yml
    │   ├── mysql/my-config-1.cnf
    │   └── exec/setup-replication.sh
    │
    └── server-2/           # Master 2 (192.168.1.20)
        ├── docker-compose.yml
        ├── mysql/my-config-2.cnf
        └── exec/setup-replication.sh
```

## 🔧 核心配置

### my-config-1.cnf (Master 1)
```ini
[mysqld]
server-id = 1
log-bin = mysql-bin
auto-increment-increment = 2
auto-increment-offset = 1
gtid_mode = ON
enforce_gtid_consistency = ON
binlog_expire_logs_seconds = 604800
```

### my-config-2.cnf (Master 2)
```ini
[mysqld]
server-id = 2
log-bin = mysql-bin
auto-increment-increment = 2
auto-increment-offset = 2
gtid_mode = ON
enforce_gtid_consistency = ON
binlog_expire_logs_seconds = 604800
```

## 🔐 默认凭证

- **Root**: `root` / `teste123`
- **复制用户**: `replicador` / `teste123`
- **phpMyAdmin**: http://localhost:8085

⚠️ **生产环境请更改密码！**

## 📊 检查状态

```bash
# 使用项目脚本
./check-replication.sh

# 或手动检查
docker exec mysql-master-1 mysql -uroot -pteste123 -e "SHOW SLAVE STATUS\G" | grep -E "(Slave_IO_Running|Slave_SQL_Running|Seconds_Behind_Master)"
```

**预期状态:**
```
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Seconds_Behind_Master: 0
```

## 🚨 故障排除

### 复制无法连接
```bash
# 检查网络
ping 192.168.1.20

# 查看日志
docker logs mysql-master-1 | tail -50

# 重新配置
cd prod/server-1/exec
./setup-replication.sh 192.168.1.20
```

### 重置复制
```bash
docker exec mysql-master-1 mysql -uroot -pteste123 -e "
STOP SLAVE; 
RESET SLAVE ALL;
"
cd exec && ./setup-replication.sh 192.168.1.20
```

## 🔄 双向复制

```
Master 1 (ID: 1, 奇数IDs)  ⟷  Master 2 (ID: 2, 偶数IDs)
      基于 GTID 的复制
```

- ✅ 自动增量防止主键冲突
- ✅ GTID 保证一致性
- ✅ 同步延迟 < 5 秒

## 🔐 安全配置（生产环境）

```bash
# 防火墙
sudo ufw allow from 192.168.1.20 to any port 3306
sudo ufw deny 3306

# SSL（推荐）
# 参考 .github/copilot-instructions.md 配置 SSL
```

## 💾 备份

```bash
# 备份
docker exec mysql-master-1 mysqldump -uroot -pteste123 --all-databases > backup.sql

# 恢复
docker exec -i mysql-master-1 mysql -uroot -pteste123 < backup.sql
```

## 🧪 测试弹性

```bash
cd dev/
./test-failover-resilience.sh
```

## 📚 其他文档

- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - 常见问题和解决方案

---

**版本**: 1.0  
**最后更新**: 2025年12月17日
