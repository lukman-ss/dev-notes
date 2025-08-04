# Data Manipulation Language (DML) – MySQL

**Data Manipulation Language (DML)** di MySQL adalah sekumpulan perintah SQL untuk **mengambil**, **menambah**, **mengubah**, dan **menghapus** baris data dalam tabel. Berikut ringkasan lengkap sintaks MySQL DML:

---

## 1. SELECT

```sql
SELECT [DISTINCT] <columns>
FROM <table> [AS alias]
  [ {INNER|LEFT|RIGHT|CROSS} JOIN <table2> [AS alias2]
      ON <join_condition> ]
  [WHERE <condition>]
  [GROUP BY <columns>]
  [HAVING <condition>]
  [ORDER BY <columns> [ASC|DESC]]
  [LIMIT <offset>, <row_count>];
```

* **DISTINCT**: hilangkan duplikat.
* **JOIN**: gabungkan tabel.
* **GROUP BY** & **HAVING**: agregasi dan filter hasil agregasi.
* **LIMIT**: batasi jumlah baris (`LIMIT offset, count`).

### 1.1 Contoh

```sql
-- Ambil 5 user teratas berdasarkan tanggal daftar
SELECT id, username, created_at
FROM users
WHERE status = 'active'
ORDER BY created_at DESC
LIMIT 5;

-- Hitung total order per customer
SELECT customer_id, COUNT(*) AS total_orders
FROM orders
GROUP BY customer_id
HAVING total_orders > 10;

-- Gabung tabel users + profil
SELECT u.id, u.username, p.full_name
FROM users u
LEFT JOIN profiles p ON p.user_id = u.id
WHERE u.status = 'active';
```

---

## 2. INSERT

```sql
-- Single-row insert
INSERT INTO <table> (col1, col2, ...)
VALUES (val1, val2, ...);

-- Multi-row insert
INSERT INTO <table> (col1, col2)
VALUES
  (v1a, v2a),
  (v1b, v2b),
  ...;

-- Ignoring duplicate errors
INSERT IGNORE INTO <table> (col1, col2)
VALUES (v1, v2);
```

### 2.1 Contoh

```sql
INSERT INTO products (name, price, stock)
VALUES ('Widget', 9.99, 100);

INSERT IGNORE INTO users (id, email)
VALUES (1, 'a@example.com');
```

---

## 3. UPDATE

```sql
UPDATE <table> [AS alias]
SET col1 = expr1,
    col2 = expr2,
    ...
[WHERE <condition>];
```

* Tanpa `WHERE`: semua baris ter-update.

### 3.1 Multi-table update

```sql
UPDATE t1
JOIN t2 ON t1.id = t2.ref_id
SET t1.col = t2.value
WHERE t2.flag = 1;
```

### 3.2 Contoh

```sql
UPDATE users
SET status = 'inactive'
WHERE last_login < NOW() - INTERVAL 30 DAY;
```

---

## 4. DELETE

```sql
DELETE FROM <table>
[WHERE <condition>];
```

* Tanpa `WHERE`: hapus semua baris.

### 4.1 Multi-table delete

```sql
DELETE t1, t2
FROM t1
JOIN t2 ON t2.ref_id = t1.id
WHERE t2.flag = 0;
```

### 4.2 Contoh

```sql
DELETE FROM sessions
WHERE expires_at < NOW();
```

---

## 5. REPLACE

```sql
REPLACE INTO <table> (col1, col2, ...)
VALUES (val1, val2, ...);
```

* Jika `PRIMARY KEY`/`UNIQUE KEY` duplikat, baris lama dihapus dan baris baru dimasukkan.

---

## 6. UPSERT (ON DUPLICATE KEY UPDATE)

```sql
INSERT INTO <table> (id, col1, col2)
VALUES (1, v1, v2)
ON DUPLICATE KEY UPDATE
  col1 = VALUES(col1),
  col2 = VALUES(col2);
```

---

## 7. LOAD DATA INFILE

```sql
LOAD DATA INFILE '/path/to/file.csv'
INTO TABLE <table>
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 LINES
(col1, col2, col3);
```

---

## 8. TRANSACTIONS

```sql
-- Mulai transaksi
START TRANSACTION;

-- Perintah DML
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
INSERT INTO transactions(user_id, amount) VALUES (1, -100);

-- Simpan atau batalkan
COMMIT;   -- simpan
-- ROLLBACK; -- batalkan
```

* **SAVEPOINT name** dan **ROLLBACK TO name** untuk rollback parsial.
* **SET TRANSACTION ISOLATION LEVEL** untuk atur isolation.

---

## 9. LOCKING

```sql
-- Kunci baris untuk update
SELECT * FROM accounts
WHERE id = 1
FOR UPDATE;
```

---

## 10. Contoh Lengkap

```sql
START TRANSACTION;

-- 1. Kurangi stok
UPDATE products
SET stock = stock - 1
WHERE id = 42;

-- 2. Tambah order baru
INSERT INTO orders (product_id, user_id, quantity, total_price)
VALUES (42, 10, 1,
        (SELECT price FROM products WHERE id = 42));

-- 3. Simpan histori
INSERT INTO order_log (order_id, event, event_time)
VALUES (LAST_INSERT_ID(), 'CREATED', NOW());

COMMIT;
```

---

## 11. Referensi

* MySQL DML: [https://dev.mysql.com/doc/refman/8.0/en/dml.html](https://dev.mysql.com/doc/refman/8.0/en/dml.html)
* MySQL Transactions: [https://dev.mysql.com/doc/refman/8.0/en/commit.html](https://dev.mysql.com/doc/refman/8.0/en/commit.html)

