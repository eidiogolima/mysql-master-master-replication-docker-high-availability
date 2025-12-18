# 🐳 MySQL Master x Master レプリケーション

GTID（グローバルトランザクションID）を使用したDocker ComposeによるMySQL双方向レプリケーション。

## 🚀 クイックスタート

### 開発環境
```bash
cd dev/
docker-compose up -d
./setup-replication.sh mysql-master-2
./check-replication.sh
```

### 本番環境
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

## 📁 ディレクトリ構造

```
├── dev/                    # 開発環境（ローカル）
│   ├── docker-compose.yml
│   ├── setup-replication.sh
│   └── check-replication.sh
│
└── prod/                   # 本番環境（分離サーバー）
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

## 🔧 必須設定

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

## 🔐 デフォルト認証情報

- **Root**: `root` / `teste123`
- **レプリケーション**: `replicador` / `teste123`
- **phpMyAdmin**: http://localhost:8085

⚠️ **本番環境では変更してください！**

## 📊 ステータス確認

```bash
# プロジェクトスクリプトを使用
./check-replication.sh

# または手動で確認
docker exec mysql-master-1 mysql -uroot -pteste123 -e "SHOW SLAVE STATUS\G" | grep -E "(Slave_IO_Running|Slave_SQL_Running|Seconds_Behind_Master)"
```

**期待されるステータス:**
```
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Seconds_Behind_Master: 0
```

## 🚨 トラブルシューティング

### レプリケーションが接続できない
```bash
# ネットワークを確認
ping 192.168.1.20

# ログを確認
docker logs mysql-master-1 | tail -50

# 再設定
cd prod/server-1/exec
./setup-replication.sh 192.168.1.20
```

### レプリケーションをリセット
```bash
docker exec mysql-master-1 mysql -uroot -pteste123 -e "
STOP SLAVE; 
RESET SLAVE ALL;
"
cd exec && ./setup-replication.sh 192.168.1.20
```

## 🔄 双方向レプリケーション

```
Master 1 (ID: 1, 奇数ID)  ⟷  Master 2 (ID: 2, 偶数ID)
      GTIDベースのレプリケーション
```

- ✅ 自動インクリメントがPK競合を防止
- ✅ GTIDが一貫性を保証
- ✅ 同期時間 < 5秒

## 🔐 セキュリティ（本番環境）

```bash
# ファイアウォール
sudo ufw allow from 192.168.1.20 to any port 3306
sudo ufw deny 3306

# SSL（推奨）
# SSL設定については .github/copilot-instructions.md を参照
```

## 💾 バックアップ

```bash
# バックアップ
docker exec mysql-master-1 mysqldump -uroot -pteste123 --all-databases > backup.sql

# リストア
docker exec -i mysql-master-1 mysql -uroot -pteste123 < backup.sql
```

## 🧪 回復力テスト

```bash
cd dev/
./test-failover-resilience.sh
```

## 📚 追加ドキュメント

- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - 一般的な問題と解決策

---

**バージョン**: 1.0  
**最終更新**: 2025年12月17日
