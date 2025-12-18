# 🐳 MySQL Master x Master Réplication

Réplication bidirectionnelle MySQL avec Docker Compose utilisant GTID (Global Transaction IDs).

## 🚀 Démarrage Rapide

### Développement
```bash
cd dev/
docker-compose up -d
./setup-replication.sh mysql-master-2
./check-replication.sh
```

### Production
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

## 📁 Structure

```
├── dev/                    # Développement (local)
│   ├── docker-compose.yml
│   ├── setup-replication.sh
│   └── check-replication.sh
│
└── prod/                   # Production (serveurs séparés)
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

## 🔧 Configuration Essentielle

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

## 🔐 Identifiants par Défaut

- **Root**: `root` / `teste123`
- **Réplication**: `replicador` / `teste123`
- **phpMyAdmin**: http://localhost:8085

⚠️ **À modifier en production !**

## 📊 Vérifier le Statut

```bash
# Utiliser le script du projet
./check-replication.sh

# Ou manuellement
docker exec mysql-master-1 mysql -uroot -pteste123 -e "SHOW SLAVE STATUS\G" | grep -E "(Slave_IO_Running|Slave_SQL_Running|Seconds_Behind_Master)"
```

**Statut attendu:**
```
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Seconds_Behind_Master: 0
```

## 🚨 Dépannage

### La réplication ne se connecte pas
```bash
# Vérifier le réseau
ping 192.168.1.20

# Voir les logs
docker logs mysql-master-1 | tail -50

# Reconfigurer
cd prod/server-1/exec
./setup-replication.sh 192.168.1.20
```

### Réinitialiser la réplication
```bash
docker exec mysql-master-1 mysql -uroot -pteste123 -e "
STOP SLAVE; 
RESET SLAVE ALL;
"
cd exec && ./setup-replication.sh 192.168.1.20
```

## 🔄 Réplication Bidirectionnelle

```
Master 1 (ID: 1, IDs impairs)  ⟷  Master 2 (ID: 2, IDs pairs)
      Réplication basée sur GTID
```

- ✅ Auto-increment prévient les conflits de clé primaire
- ✅ GTID garantit la cohérence
- ✅ Synchronisation < 5 secondes

## 🔐 Sécurité (Production)

```bash
# Pare-feu
sudo ufw allow from 192.168.1.20 to any port 3306
sudo ufw deny 3306

# SSL (recommandé)
# Voir .github/copilot-instructions.md pour la configuration SSL
```

## 💾 Sauvegarde

```bash
# Backup
docker exec mysql-master-1 mysqldump -uroot -pteste123 --all-databases > backup.sql

# Restauration
docker exec -i mysql-master-1 mysql -uroot -pteste123 < backup.sql
```

## 🧪 Tester la Résilience

```bash
cd dev/
./test-failover-resilience.sh
```

## 📚 Documentation Supplémentaire

- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Problèmes courants et solutions

---

**Version**: 1.0  
**Dernière mise à jour**: 17 décembre 2025
