## 起動方法

```bash
docker compose up -d --build
```

- BE:http://localhost:8080

- FE:http://localhost:3000

### 環境

- MYSQL_USER=app
- MYSQL_PASSWORD=app
- MYSQL_DATABASE=app
- MYSQL_ROOT_PASSWORD: root

## テスト実行方法

バックエンド（Go）で 1 本のユニットテストを用意しています。  
ページネーション用の小さな関数 `calcTotalPages` が正しく動作することを確認しています。

### 実行コマンド

```bash
cd backend
go test ./...
```
### テスト内容の概要
- `calcTotalPages` が総件数と1ページあたり件数から正しいページ数を計算できることを確認
- 端数切り上げのケース、0件の場合、1ページあたり件数が0以下の場合など複数パターンをチェック



## 使用した主な SQL

### 1. 入札作成（POST /bids）
```sql
INSERT INTO bids (car_id, amount, bidder, request_id)
VALUES (?, ?, ?, ?);
```

### 2. 入札一覧取得（GET /bids?item_id=）
```sql
SELECT id, car_id, amount, bidder, request_id, created_at
FROM bids
WHERE car_id = ?
ORDER BY created_at DESC;
```

### 3. 車両ごとの最高入札額（結合 + 集計）
```sql
SELECT
  c.id       AS car_id,
  c.model    AS model,
  MAX(b.amount) AS max_bid
FROM cars c
LEFT JOIN bids b ON c.id = b.car_id
GROUP BY c.id, c.model;
```
## EXPLAIN(2本）

<img width="1890" height="1062" alt="スクリーンショット 2025-11-21 003113" src="https://github.com/user-attachments/assets/9245a5c2-eed4-4210-83ff-0bb72f0a9204" />

- `入札一覧取得（上）`
- `車両ごとの最高入札額（下）`



## DDL(cars/bids)

```sql
CREATE TABLE IF NOT EXISTS cars (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  model VARCHAR(64) NOT NULL,
  price INT NOT NULL,
  year INT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);


CREATE TABLE IF NOT EXISTS bids (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  car_id BIGINT NOT NULL,
  amount INT NOT NULL,
  bidder VARCHAR(64) NOT NULL,
  request_id CHAR(36) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT fk_bids_car FOREIGN KEY (car_id) REFERENCES cars(id),
  UNIQUE KEY uk_request_id (request_id)
);
```


## インデックス

- `idx_cars_price`  
  価格フィルタや並び替えを高速化するため、`cars.price` にインデックスを付与。

- `idx_bids_car_created`  
  `GET /bids?item_id=` の  
  `WHERE car_id = ? ORDER BY created_at DESC` を高速にするため、  
  `bids(car_id, created_at)` に複合インデックスを付与。

## SSR / CSR の選択

Client Side Rendering（CSR）を採用。
車両一覧・詳細は SEO を特に必要としておらず、価格フィルタやページネーションなどクライアント側での状態更新が中心のため、API(Go) と連携するシンプルな SPA 構成にすることで実装・デバッグコストを下げた。







