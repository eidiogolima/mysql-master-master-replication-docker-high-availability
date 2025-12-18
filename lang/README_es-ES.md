# 🐳 MySQL Master x Master Replicación

Replicación bidireccional MySQL con Docker Compose usando GTID (Global Transaction IDs).

## 🚀 Inicio Rápido

### Desarrollo
```bash
cd dev/
docker-compose up -d
./setup-replication.sh mysql-master-2
./check-replication.sh
```

### Producción
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

## 📁 Estructura

```
├── dev/                    # Desarrollo (local)
│   ├── docker-compose.yml
│   ├── setup-replication.sh
│   └── check-replication.sh
│
└── prod/                   # Producción (servidores separados)
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

## 🔧 Configuración Esencial

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

## 🔐 Credenciales Predeterminadas

- **Root**: `root` / `teste123`
- **Replicación**: `replicador` / `teste123`
- **phpMyAdmin**: http://localhost:8085

⚠️ **¡Cambiar en producción!**

## 📊 Verificar Estado

```bash
# Usar script del proyecto
./check-replication.sh

# O manualmente
docker exec mysql-master-1 mysql -uroot -pteste123 -e "SHOW SLAVE STATUS\G" | grep -E "(Slave_IO_Running|Slave_SQL_Running|Seconds_Behind_Master)"
```

**Estado esperado:**
```
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Seconds_Behind_Master: 0
```

## 🚨 Solución de Problemas

### La replicación no conecta
```bash
# Verificar red
ping 192.168.1.20

# Ver logs
docker logs mysql-master-1 | tail -50

# Reconfigurar
cd prod/server-1/exec
./setup-replication.sh 192.168.1.20
```

### Resetear replicación
```bash
docker exec mysql-master-1 mysql -uroot -pteste123 -e "
STOP SLAVE; 
RESET SLAVE ALL;
"
cd exec && ./setup-replication.sh 192.168.1.20
```

## 🔄 Replicación Bidireccional

```
Master 1 (ID: 1, IDs impares)  ⟷  Master 2 (ID: 2, IDs pares)
      Replicación basada en GTID
```

- ✅ Auto-increment previene conflictos de clave primaria
- ✅ GTID garantiza consistencia
- ✅ Sincronización < 5 segundos

## 🔐 Seguridad (Producción)

```bash
# Firewall
sudo ufw allow from 192.168.1.20 to any port 3306
sudo ufw deny 3306

# SSL (recomendado)
# Ver .github/copilot-instructions.md para configuración SSL
```

## 💾 Respaldo

```bash
# Backup
docker exec mysql-master-1 mysqldump -uroot -pteste123 --all-databases > backup.sql

# Restaurar
docker exec -i mysql-master-1 mysql -uroot -pteste123 < backup.sql
```

## 🧪 Probar Resiliencia

```bash
cd dev/
./test-failover-resilience.sh
```

## 📚 Documentación Adicional

- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Problemas comunes y soluciones

---

**Versión**: 1.0  
**Última actualización**: 17 de diciembre de 2025
