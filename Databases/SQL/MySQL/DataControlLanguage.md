# Data Control Language (DCL) – MySQL

**Data Control Language (DCL)** di MySQL berisi perintah untuk **mengelola hak akses** dan **otorisasi** pengguna database. Dengan DCL, administrator dapat membuat user, memberikan atau mencabut izin, serta memeriksa hak akses yang sudah diterapkan.

---

## 1. CREATE USER

Membuat akun user baru di MySQL.

```sql
CREATE USER 'username'@'host'
  IDENTIFIED BY 'password';
```

* **'username'@'host'**  : pola koneksi (misal '%', 'localhost').
* **IDENTIFIED BY**      : menetapkan kata sandi.

### Contoh

```sql
CREATE USER 'app_user'@'%' IDENTIFIED BY 'StrongP@ssw0rd';
```

---

## 2. DROP USER

Menghapus akun user yang sudah ada.

```sql
DROP USER IF EXISTS 'username'@'host';
```

### Contoh

```sql
DROP USER IF EXISTS 'old_user'@'localhost';
```

---

## 3. GRANT

Memberikan hak akses spesifik ke user.

```sql
GRANT priv_type [, priv_type...]
  ON db_name.tbl_name
  TO 'username'@'host'
  [WITH GRANT OPTION];
```

* **priv\_type**   : SELECT, INSERT, UPDATE, DELETE, ALL PRIVILEGES, dsb.
* **db\_name.tbl\_name** : `*.*` untuk seluruh database.
* **WITH GRANT OPTION** : memungkinkan user memberi izin ke user lain.

### Contoh

```sql
-- Hak baca/tulis pada semua tabel di database shop
GRANT SELECT, INSERT, UPDATE, DELETE
  ON shop.*
  TO 'app_user'@'%';

-- Semua hak + bisa grant ke user lain
GRANT ALL PRIVILEGES
  ON production.*
  TO 'admin_user'@'192.168.1.%'
  WITH GRANT OPTION;
```

---

## 4. REVOKE

Mencabut hak akses yang pernah diberikan.

```sql
REVOKE priv_type [, priv_type...]
  ON db_name.tbl_name
  FROM 'username'@'host';
```

### Contoh

```sql
-- Cabut hak INSERT dan DELETE dari app_user
REVOKE INSERT, DELETE
  ON shop.orders
  FROM 'app_user'@'%';
```

---

## 5. SHOW GRANTS

Menampilkan daftar izin yang dimiliki user.

```sql
SHOW GRANTS FOR 'username'@'host';
```

### Contoh

```sql
SHOW GRANTS FOR 'app_user'@'%';
-- Output: GRANT SELECT, INSERT ON `shop`.* TO 'app_user'@'%'
```

---

## 6. FLUSH PRIVILEGES

Memuat ulang tabel privilege setelah perubahan manual di file grant tables.

```sql
FLUSH PRIVILEGES;
```

### Kapan digunakan?

* Jika Anda melakukan perubahan langsung pada tabel mysql.user, mysql.db, dsb. tanpa menggunakan GRANT/REVOKE.

---

## 7. Contoh Skenario Lengkap

```sql
-- 1. Buat user baru
CREATE USER 'reporter'@'%' IDENTIFIED BY 'RptP@ss1';

-- 2. Beri izin hanya baca pada database reporting
GRANT SELECT
  ON reporting.*
  TO 'reporter'@'%';

-- 3. Cek izin reporter
SHOW GRANTS FOR 'reporter'@'%';

-- 4. Cabut izin jika sudah tidak diperlukan
REVOKE SELECT
  ON reporting.*
  FROM 'reporter'@'%';

-- 5. Hapus user setelah selesai
DROP USER 'reporter'@'%';
```


