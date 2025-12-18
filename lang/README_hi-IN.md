# 🐳 MySQL Master x Master रेप्लिकेशन

GTID (Global Transaction IDs) का उपयोग करते हुए Docker Compose के साथ द्विदिशात्मक MySQL रेप्लिकेशन।

## 🚀 त्वरित शुरुआत

### विकास
```bash
cd dev/
docker-compose up -d
./setup-replication.sh mysql-master-2
./check-replication.sh
```

### उत्पादन
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

## 📁 संरचना

```
├── dev/                    # विकास (स्थानीय)
│   ├── docker-compose.yml
│   ├── setup-replication.sh
│   └── check-replication.sh
│
└── prod/                   # उत्पादन (अलग सर्वर)
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

## 🔧 आवश्यक कॉन्फ़िगरेशन

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

## 🔐 डिफ़ॉल्ट क्रेडेंशियल्स

- **Root**: `root` / `teste123`
- **रेप्लिकेशन**: `replicador` / `teste123`
- **phpMyAdmin**: http://localhost:8085

⚠️ **उत्पादन में बदलें!**

## 📊 स्थिति जांचें

```bash
# प्रोजेक्ट स्क्रिप्ट का उपयोग करें
./check-replication.sh

# या मैन्युअल रूप से
docker exec mysql-master-1 mysql -uroot -pteste123 -e "SHOW SLAVE STATUS\G" | grep -E "(Slave_IO_Running|Slave_SQL_Running|Seconds_Behind_Master)"
```

**अपेक्षित स्थिति:**
```
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Seconds_Behind_Master: 0
```

## 🚨 समस्या निवारण

### रेप्लिकेशन कनेक्ट नहीं हो रहा
```bash
# नेटवर्क जांचें
ping 192.168.1.20

# लॉग देखें
docker logs mysql-master-1 | tail -50

# पुनः कॉन्फ़िगर करें
cd prod/server-1/exec
./setup-replication.sh 192.168.1.20
```

### रेप्लिकेशन रीसेट करें
```bash
docker exec mysql-master-1 mysql -uroot -pteste123 -e "
STOP SLAVE; 
RESET SLAVE ALL;
"
cd exec && ./setup-replication.sh 192.168.1.20
```

## 🔄 द्विदिशात्मक रेप्लिकेशन

```
Master 1 (ID: 1, विषम IDs)  ⟷  Master 2 (ID: 2, सम IDs)
      GTID-based replication
```

- ✅ Auto-increment PK टकराव को रोकता है
- ✅ GTID स्थिरता की गारंटी देता है
- ✅ सिंक्रनाइज़ेशन < 5 सेकंड

## 🔐 सुरक्षा (उत्पादन)

```bash
# फ़ायरवॉल
sudo ufw allow from 192.168.1.20 to any port 3306
sudo ufw deny 3306

# SSL (अनुशंसित)
# SSL कॉन्फ़िगरेशन के लिए .github/copilot-instructions.md देखें
```

## 💾 बैकअप

```bash
# बैकअप
docker exec mysql-master-1 mysqldump -uroot -pteste123 --all-databases > backup.sql

# रिस्टोर
docker exec -i mysql-master-1 mysql -uroot -pteste123 < backup.sql
```

## 🧪 लचीलापन परीक्षण

```bash
cd dev/
./test-failover-resilience.sh
```

## 📚 अतिरिक्त दस्तावेज़ीकरण

- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - सामान्य समस्याएं और समाधान

---

**संस्करण**: 1.0  
**अंतिम अपडेट**: 17 दिसंबर 2025
