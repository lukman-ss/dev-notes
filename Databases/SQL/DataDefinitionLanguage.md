# 📐 Data Definition Language (DDL) – SQL

**Data Definition Language (DDL)** adalah sekumpulan perintah SQL untuk **mendefinisikan**, **mengubah**, dan **menghapus** struktur objek di dalam database (database itu sendiri, schema, tabel, kolom, index, view, dsb.). DDL tidak langsung memanipulasi baris data—itu tugas DML (INSERT/UPDATE/DELETE).

---

## 1. CREATE DATABASE

```sql
CREATE DATABASE database_name
  [ WITH 
      [ OWNER = db_owner ]
    | [ TEMPLATE = template_db ]
    | [ ENCODING = 'encoding' ]
    | [ LC_COLLATE = 'locale' ]
    | [ LC_CTYPE = 'locale' ]
    | [ TABLESPACE = tablespace_name ]
    | [ CONNECTION LIMIT = max_connections ]
  ];
````

* **database\_name**
  Nama unik database yang akan dibuat.

* **OWNER = db\_owner**
  User yang menjadi pemilik database.

* **TEMPLATE = template\_db**
  Salinan struktur & data awal (PostgreSQL).

* **ENCODING**, **LC\_COLLATE**, **LC\_CTYPE**
  Karakter set & aturan collation.

* **TABLESPACE**
  Lokasi fisik file database.

* **CONNECTION LIMIT**
  Batas koneksi simultan ke database.

---

## 2. CREATE SCHEMA

Schema adalah *namespace* dalam satu database.

```sql
CREATE SCHEMA schema_name
  [ AUTHORIZATION db_user ];
```

* **schema\_name**
  “Folder” virtual untuk objek (tabel, view, fungsi).

* **AUTHORIZATION db\_user**
  Pemilik schema.

---

## 3. CREATE TABLE

```sql
CREATE TABLE [ IF NOT EXISTS ] table_name
(
  -- definisi kolom
  column1_name data_type [ column_constraint ... ],
  column2_name data_type [ column_constraint ... ],
  ...
  -- definisi constraint tabel (opsional)
  [ table_constraint ... ]
)
[ PARTITION BY ... ]
[ WITH ( storage_parameter = value [, ...] ) ]
[ TABLESPACE tablespace_name ];
```

### 3.1. column\_definition

| Bagian                 | Fungsi                                                    |
| ---------------------- | --------------------------------------------------------- |
| **column\_name**       | Nama kolom (snake\_case)                                  |
| **data\_type**         | Tipe data (INT, BIGINT, VARCHAR(n), TEXT, DATE, BOOLEAN…) |
| **column\_constraint** | NOT NULL, UNIQUE, PRIMARY KEY, DEFAULT, CHECK, REFERENCES |

#### Contoh kolom

```sql
user_id     BIGSERIAL     PRIMARY KEY,
username    VARCHAR(50)   NOT NULL UNIQUE,
email       VARCHAR(100)  NOT NULL,
created_at  TIMESTAMP     NOT NULL DEFAULT CURRENT_TIMESTAMP,
status      VARCHAR(20)   NOT NULL DEFAULT 'active'
                 CHECK (status IN ('active','inactive','blocked'))
```

* **NOT NULL**
  Kolom wajib diisi.

* **UNIQUE**
  Nilai harus unik.

* **PRIMARY KEY**
  Gabungan NOT NULL + UNIQUE.

* **DEFAULT expr**
  Nilai default jika tidak di-INSERT.

* **CHECK (condition)**
  Validasi custom, misal daftar nilai atau rentang angka.

* **REFERENCES other\_table(column)**
  Foreign key—harus cocok dengan primary key di tabel lain.

### 3.2. table\_constraint

Didefinisikan setelah semua kolom:

```sql
PRIMARY KEY (col1 [, col2, ...]),
UNIQUE     (colA [, colB, ...]),
FOREIGN KEY (colX) REFERENCES other_table(pk_col) ON DELETE CASCADE,
CHECK      (expression)
```

* **PRIMARY KEY(...)**
  Bisa multi-kolom (composite key).

* **FOREIGN KEY(...) …**
  Atur aksi `ON DELETE` / `ON UPDATE` (CASCADE, SET NULL, RESTRICT).

---

## 4. ALTER TABLE

Mengubah struktur tabel yang sudah ada:

```sql
ALTER TABLE table_name
  ADD    COLUMN column_name data_type [ column_constraint ... ],
  DROP   COLUMN column_name                [ CASCADE | RESTRICT ],
  ALTER  COLUMN column_name SET DEFAULT expr,
  ALTER  COLUMN column_name DROP DEFAULT,
  RENAME COLUMN old_name TO new_name,
  ADD    table_constraint,
  DROP   CONSTRAINT constraint_name;
```

---

## 5. DROP & TRUNCATE

* **DROP**: hapus objek (struktur + data).

  ```sql
  DROP TABLE IF EXISTS table_name CASCADE;
  ```

* **TRUNCATE**: hapus seluruh baris tapi pertahankan struktur.

  ```sql
  TRUNCATE TABLE table_name RESTART IDENTITY CASCADE;
  ```

---

## 6. EXPLAIN, DESCRIBE & SHOW

Meski bukan murni DDL, perintah ini penting untuk *inspeksi* dan *optimasi*.

* **EXPLAIN** / **EXPLAIN ANALYZE**
  Tampilkan *query plan*.

  ```sql
  EXPLAIN SELECT * FROM users WHERE status = 'active';
  EXPLAIN ANALYZE UPDATE users SET status = 'blocked' WHERE user_id = 1;
  ```

* **DESCRIBE** / **DESC** (MySQL)
  Tampilkan struktur tabel.

  ```sql
  DESCRIBE users;
  ```

* **\d**, **\d+** (PostgreSQL psql)
  Lihat struktur tabel/indeks.

  ```
  \d users
  \d+ users
  ```

* **SHOW CREATE TABLE** (MySQL)
  Tampilkan DDL lengkap.

  ```sql
  SHOW CREATE TABLE users;
  ```

---

## 7. DDL “Lainnya”

| Objek                    | Perintah Create                                      | Perintah Drop                     |
| ------------------------ | ---------------------------------------------------- | --------------------------------- |
| **Index**                | `CREATE INDEX idx_name ON tbl(col);`                 | `DROP INDEX idx_name;`            |
| **View**                 | `CREATE VIEW v_user AS SELECT ...;`                  | `DROP VIEW v_user;`               |
| **Sequence**             | `CREATE SEQUENCE seq_name;`                          | `DROP SEQUENCE seq_name;`         |
| **Function / Procedure** | `CREATE FUNCTION fn_name(args) RETURNS type AS ...;` | `DROP FUNCTION fn_name(args);`    |
| **Trigger**              | `CREATE TRIGGER trg_name BEFORE INSERT ON tbl ...;`  | `DROP TRIGGER trg_name ON tbl;`   |
| **Type / Domain**        | `CREATE TYPE status AS ENUM('a','b','c');`           | `DROP TYPE status;`               |
| **Tablespace**           | `CREATE TABLESPACE ts_name LOCATION 'path';`         | `DROP TABLESPACE ts_name;`        |
| **Materialized View**    | `CREATE MATERIALIZED VIEW mv_name AS SELECT ...;`    | `DROP MATERIALIZED VIEW mv_name;` |
| **Comment**              | `COMMENT ON TABLE tbl IS '...';`                     | —                                 |

---

## 8. Contoh Lengkap

```sql
-- 1. Buat database
CREATE DATABASE my_app
  WITH OWNER = admin
       ENCODING = 'UTF8'
       LC_COLLATE = 'en_US.UTF-8'
       LC_CTYPE   = 'en_US.UTF-8'
       TEMPLATE   = template0;

-- 2. Buat schema (opsional)
CREATE SCHEMA auth AUTHORIZATION admin;

-- 3. Buat tabel users
CREATE TABLE auth.users
(
  user_id     BIGSERIAL     PRIMARY KEY,
  username    VARCHAR(50)   NOT NULL UNIQUE,
  email       VARCHAR(100)  NOT NULL,
  password    TEXT          NOT NULL,
  created_at  TIMESTAMP     NOT NULL DEFAULT CURRENT_TIMESTAMP,
  status      VARCHAR(20)   NOT NULL DEFAULT 'active'
                   CHECK (status IN ('active','inactive','blocked'))
);

-- 4. Tambah kolom profile_id sebagai FK
ALTER TABLE auth.users
  ADD COLUMN profile_id INT,
  ADD CONSTRAINT fk_profile
    FOREIGN KEY (profile_id)
    REFERENCES auth.profiles(profile_id)
    ON DELETE SET NULL;

-- 5. Buat index untuk username
CREATE INDEX idx_users_username ON auth.users(username);

-- 6. Buat view user aktif
CREATE VIEW auth.v_active_users AS
SELECT user_id, username, email
FROM auth.users
WHERE status = 'active';

-- 7. Inspect query plan
EXPLAIN ANALYZE
SELECT * FROM auth.users WHERE status = 'active';

-- 8. Hapus tabel & objek jika perlu
DROP MATERIALIZED VIEW IF EXISTS auth.mv_stats;
DROP VIEW IF EXISTS auth.v_active_users;
DROP INDEX IF EXISTS idx_users_username;
DROP TABLE IF EXISTS auth.users CASCADE;
DROP SCHEMA IF EXISTS auth CASCADE;
DROP DATABASE IF EXISTS my_app;
```

