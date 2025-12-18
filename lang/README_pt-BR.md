# 🐳 MySQL Master x Master Replication

Replicação bidirecional MySQL com Docker Compose usando GTID (Global Transaction IDs).

## 🚀 Quick Start

### Desenvolvimento
```bash
cd dev/
docker-compose up -d
./setup-replication.sh mysql-master-2
./check-replication.sh
```

### Produção
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

## 📁 Estrutura

```
├── dev/                    # Desenvolvimento (local)
│   ├── docker-compose.yml
│   ├── setup-replication.sh
│   └── check-replication.sh
│
└── prod/                   # Produção (servidores separados)
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

## 🔧 Configuração Essencial

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

## 🔐 Credenciais Padrão

- **Root**: `root` / `teste123`
- **Replicação**: `replicador` / `teste123`
- **phpMyAdmin**: http://localhost:8085

⚠️ **Altere em produção!**

## 📊 Verificar Status

```bash
# Usar script do projeto
./check-replication.sh

# Ou manualmente
docker exec mysql-master-1 mysql -uroot -pteste123 -e "SHOW SLAVE STATUS\G" | grep -E "(Slave_IO_Running|Slave_SQL_Running|Seconds_Behind_Master)"
```

**Status esperado:**
```
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Seconds_Behind_Master: 0
```

## 🚨 Troubleshooting

### Replicação não conecta
```bash
# Verificar rede
ping 192.168.1.20

# Ver logs
docker logs mysql-master-1 | tail -50

# Reconfigurar
cd prod/server-1/exec
./setup-replication.sh 192.168.1.20
```

### Resetar replicação
```bash
docker exec mysql-master-1 mysql -uroot -pteste123 -e "
STOP SLAVE; 
RESET SLAVE ALL;
"
cd exec && ./setup-replication.sh 192.168.1.20
```

## 🔄 Replicação Bidirecional

```
Master 1 (ID: 1, IDs ímpares)  ⟷  Master 2 (ID: 2, IDs pares)
      GTID-based replication
```

- ✅ Auto-increment previne conflitos de PK
- ✅ GTID garante consistência
- ✅ Sincronização < 5 segundos

## 🔐 Segurança (Produção)

```bash
# Firewall
sudo ufw allow from 192.168.1.20 to any port 3306
sudo ufw deny 3306

# SSL (recomendado)
# Ver .github/copilot-instructions.md para configuração SSL
```

## 💾 Backup

```bash
# Backup
docker exec mysql-master-1 mysqldump -uroot -pteste123 --all-databases > backup.sql

# Restore
docker exec -i mysql-master-1 mysql -uroot -pteste123 < backup.sql
```

## 🧪 Testar Resiliência

```bash
cd dev/
./test-failover-resilience.sh
```

## 📚 Documentação Adicional

- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Problemas comuns e soluções

---

**Versão**: 1.0  
**Última atualização**: 17 de dezembro de 2025
