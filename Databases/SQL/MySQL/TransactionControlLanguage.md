# Transaction Control Language (TCL) – MySQL

**Transaction Control Language (TCL)** di MySQL adalah sekumpulan perintah SQL untuk **mengelola transaksi**, menjamin konsistensi dan integritas data, serta memungkinkan rollback jika terjadi kesalahan.

---

## 1. START TRANSACTION / BEGIN

```sql
START TRANSACTION;
-- atau
BEGIN;
````

Memulai blok transaksi. Semua perintah DML berikutnya akan bersifat atomik hingga `COMMIT` atau `ROLLBACK`.

---

## 2. COMMIT

```sql
COMMIT;
```

Menyimpan semua perubahan yang dilakukan sejak `START TRANSACTION`. Setelah `COMMIT`, perubahan tidak dapat dibatalkan.

---

## 3. ROLLBACK

```sql
ROLLBACK;
```

Membatalkan seluruh perubahan dalam transaksi saat ini, mengembalikan database ke keadaan sebelum `START TRANSACTION`.

---

## 4. SAVEPOINT & ROLLBACK TO

```sql
SAVEPOINT sp_name;
-- beberapa perintah DML...
ROLLBACK TO SAVEPOINT sp_name;
```

* `SAVEPOINT` membuat titik simpan parsial di dalam transaksi.
* `ROLLBACK TO SAVEPOINT` membatalkan perubahan hanya hingga titik tersebut, bukan seluruh transaksi.

---

## 5. SET TRANSACTION ISOLATION LEVEL

```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
-- Pilihan lain:
-- READ UNCOMMITTED
-- READ COMMITTED
-- SERIALIZABLE
```

Menentukan level isolasi transaksi untuk mengontrol fenomena *dirty reads*, *non-repeatable reads*, dan *phantom reads*.

---

## 6. Contoh Lengkap

```sql
START TRANSACTION;

-- 1. Kurangi stok produk
UPDATE products
SET stock = stock - 1
WHERE id = 42;

-- 2. Tambah order baru
INSERT INTO orders (product_id, user_id, quantity, total_price)
VALUES (
  42,
  10,
  1,
  (SELECT price FROM products WHERE id = 42)
);

-- 3. Catat histori order
INSERT INTO order_log (order_id, event, event_time)
VALUES (
  LAST_INSERT_ID(),
  'CREATED',
  NOW()
);

COMMIT;
```

---

## 7. Referensi

* MySQL Transactions & COMMIT/ROLLBACK
  [https://dev.mysql.com/doc/refman/8.0/en/commit.html](https://dev.mysql.com/doc/refman/8.0/en/commit.html)
* MySQL SAVEPOINT & Partial Rollback
  [https://dev.mysql.com/doc/refman/8.0/en/savepoint.html](https://dev.mysql.com/doc/refman/8.0/en/savepoint.html)
* MySQL Transaction Isolation Levels
  [https://dev.mysql.com/doc/refman/8.0/en/set-transaction.html](https://dev.mysql.com/doc/refman/8.0/en/set-transaction.html)

