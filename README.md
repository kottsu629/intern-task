#Car Auction App (Go + Next.js)#

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

### 3. 車両ごとの最高入札額（JOIN + 集計）
```sql
SELECT
  c.id       AS car_id,
  c.model    AS model,
  MAX(b.amount) AS max_bid
FROM cars c
LEFT JOIN bids b ON c.id = b.car_id
GROUP BY c.id, c.model;
```

## インデックス

- `idx_cars_price`  
  価格フィルタや並び替えを高速化するため、`cars.price` にインデックスを付与。

- `idx_bids_car_created`  
  `GET /bids?item_id=` の  
  `WHERE car_id = ? ORDER BY created_at DESC` を高速にするため、  
  `bids(car_id, created_at)` に複合インデックスを付与。




