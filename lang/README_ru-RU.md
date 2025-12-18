# 🐳 MySQL Master x Master Репликация

Двунаправленная репликация MySQL с Docker Compose использующая GTID (Global Transaction IDs).

## 🚀 Быстрый старт

### Разработка
```bash
cd dev/
docker-compose up -d
./setup-replication.sh mysql-master-2
./check-replication.sh
```

### Продакшн
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

## 📁 Структура

```
├── dev/                    # Разработка (локально)
│   ├── docker-compose.yml
│   ├── setup-replication.sh
│   └── check-replication.sh
│
└── prod/                   # Продакшн (отдельные серверы)
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

## 🔧 Основная конфигурация

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

## 🔐 Учетные данные по умолчанию

- **Root**: `root` / `teste123`
- **Репликация**: `replicador` / `teste123`
- **phpMyAdmin**: http://localhost:8085

⚠️ **Измените в продакшне!**

## 📊 Проверка статуса

```bash
# Использовать скрипт проекта
./check-replication.sh

# Или вручную
docker exec mysql-master-1 mysql -uroot -pteste123 -e "SHOW SLAVE STATUS\G" | grep -E "(Slave_IO_Running|Slave_SQL_Running|Seconds_Behind_Master)"
```

**Ожидаемый статус:**
```
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Seconds_Behind_Master: 0
```

## 🚨 Устранение неполадок

### Репликация не подключается
```bash
# Проверить сеть
ping 192.168.1.20

# Просмотреть логи
docker logs mysql-master-1 | tail -50

# Переконфигурировать
cd prod/server-1/exec
./setup-replication.sh 192.168.1.20
```

### Сброс репликации
```bash
docker exec mysql-master-1 mysql -uroot -pteste123 -e "
STOP SLAVE; 
RESET SLAVE ALL;
"
cd exec && ./setup-replication.sh 192.168.1.20
```

## 🔄 Двунаправленная репликация

```
Master 1 (ID: 1, нечетные ID)  ⟷  Master 2 (ID: 2, четные ID)
      Репликация на основе GTID
```

- ✅ Auto-increment предотвращает конфликты первичных ключей
- ✅ GTID гарантирует согласованность
- ✅ Синхронизация < 5 секунд

## 🔐 Безопасность (Продакшн)

```bash
# Файрвол
sudo ufw allow from 192.168.1.20 to any port 3306
sudo ufw deny 3306

# SSL (рекомендуется)
# См. .github/copilot-instructions.md для настройки SSL
```

## 💾 Резервное копирование

```bash
# Бэкап
docker exec mysql-master-1 mysqldump -uroot -pteste123 --all-databases > backup.sql

# Восстановление
docker exec -i mysql-master-1 mysql -uroot -pteste123 < backup.sql
```

## 🧪 Тест устойчивости

```bash
cd dev/
./test-failover-resilience.sh
```

## 📚 Дополнительная документация

- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Распространенные проблемы и решения

---

**Версия**: 1.0  
**Последнее обновление**: 17 декабря 2025
