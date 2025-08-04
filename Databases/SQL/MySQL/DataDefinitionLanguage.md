# 📐 Data Definition Language (DDL) – MySQL

**Data Definition Language (DDL)** adalah sekumpulan perintah SQL untuk **mendefinisikan**, **mengubah**, dan **menghapus** struktur objek di dalam MySQL (database, schema, tabel, kolom, index, view, dsb.). DDL tidak langsung memanipulasi baris data—itu tugas DML (INSERT/UPDATE/DELETE).

---

## 1. CREATE DATABASE

```sql
CREATE DATABASE IF NOT EXISTS `database_name`
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
USE `database_name`;
```

* **database\_name**
  Nama database yang akan dibuat (jika belum ada).

* **CHARACTER SET**
  Set karakter (misal utf8mb4).

* **COLLATE**
  Pengaturan collation (misal utf8mb4\_unicode\_ci).

---

## 2. CREATE SCHEMA

Di MySQL, `CREATE SCHEMA` adalah alias untuk `CREATE DATABASE`.

```sql
CREATE SCHEMA IF NOT EXISTS `schema_name`
  DEFAULT CHARACTER SET utf8mb4
  DEFAULT COLLATE utf8mb4_unicode_ci;
USE `schema_name`;
```

* Sama seperti `CREATE DATABASE`.

---

## 3. CREATE TABLE

```sql
CREATE TABLE IF NOT EXISTS `table_name` (
  `column1` INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `column2` VARCHAR(100) NOT NULL,
  `column3` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  ...
  PRIMARY KEY (`column1`),
  UNIQUE KEY `uniq_column2` (`column2`),
  CONSTRAINT `fk_example`
    FOREIGN KEY (`column3`)
    REFERENCES `other_table` (`id`)
    ON DELETE CASCADE
    ON UPDATE CASCADE
) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4
  COLLATE=utf8mb4_unicode_ci;
```

### 3.1. Penjelasan Field

* **INT UNSIGNED NOT NULL AUTO\_INCREMENT**
  Tipe integer auto increment.

* **VARCHAR(n) NOT NULL**
  Tipe karakter dengan panjang maksimal n.

* **TIMESTAMP DEFAULT CURRENT\_TIMESTAMP**
  Waktu otomatis pada insert.

### 3.2. Constraint

* **PRIMARY KEY**
  Kunci utama tabel.

* **UNIQUE KEY**
  Indeks unik.

* **FOREIGN KEY**
  Kunci asing dengan aksi `ON DELETE` dan `ON UPDATE`.

---

## 4. ALTER TABLE

```sql
ALTER TABLE `table_name`
  ADD COLUMN `new_column` VARCHAR(50) NOT NULL,
  MODIFY COLUMN `column2` VARCHAR(150) NOT NULL,
  CHANGE COLUMN `old_name` `new_name` INT UNSIGNED NOT NULL,
  DROP COLUMN `column3`,
  ADD INDEX `idx_column2` (`column2`),
  DROP INDEX `idx_column2`;
```

---

## 5. DROP & TRUNCATE

* **DROP TABLE**

  ```sql
  DROP TABLE IF EXISTS `table_name`;
  ```

* **TRUNCATE TABLE**

  ```sql
  TRUNCATE TABLE `table_name`;
  ```

---

## 6. DESCRIBE, EXPLAIN & SHOW

* **DESCRIBE**
  Menampilkan struktur tabel.

  ```sql
  DESCRIBE `table_name`;
  ```

* **EXPLAIN**
  Menampilkan query plan.

  ```sql
  EXPLAIN SELECT * FROM `table_name` WHERE `column2` = 'value';
  ```

* **SHOW CREATE TABLE**
  Menampilkan DDL lengkap.

  ```sql
  SHOW CREATE TABLE `table_name`;
  ```

---

## 7. Objek DDL Lainnya di MySQL
## 7. Objek DDL Lainnya di MySQL

### 7.1 Index

```sql
-- Create
CREATE INDEX idx_name ON table_name(column);
-- Drop
DROP INDEX idx_name ON table_name;
```

### 7.2 View

```sql
-- Create
CREATE VIEW view_name AS
  SELECT ...;
-- Drop
DROP VIEW view_name;
```

### 7.3 Trigger

```sql
DELIMITER //
CREATE TRIGGER trigger_name
  BEFORE INSERT ON table_name
  FOR EACH ROW
BEGIN
  -- aksi trigger
END;//
DELIMITER ;
-- Drop
DROP TRIGGER trigger_name;
```

### 7.4 Procedure

```sql
DELIMITER //
CREATE PROCEDURE proc_name(IN param1 INT)
BEGIN
  -- body procedure
END;//
DELIMITER ;
-- Drop
DROP PROCEDURE proc_name;
```

### 7.5 Function

```sql
DELIMITER //
CREATE FUNCTION func_name(param1 INT) RETURNS INT
BEGIN
  RETURN param1 * 2;
END;//
DELIMITER ;
-- Drop
DROP FUNCTION func_name;
```

### 7.6 Event

```sql
-- Create
CREATE EVENT event_name
ON SCHEDULE EVERY 1 DAY
DO
  -- aksi event;
-- Drop
DROP EVENT event_name;
```

### 7.7 Database (alias)

````sql
-- Create (alias)
CREATE DATABASE ...;
-- Drop
DROP DATABASE database_name;
```## 8. Contoh Lengkap

```sql
-- 1. Buat database
CREATE DATABASE IF NOT EXISTS `my_app`
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
USE `my_app`;

-- 2. Buat tabel users
CREATE TABLE `users` (
  `user_id` INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `username` VARCHAR(50) NOT NULL UNIQUE,
  `email` VARCHAR(100) NOT NULL,
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- 3. Alter tabel: tambahkan profile_id
ALTER TABLE `users`
  ADD COLUMN `profile_id` INT UNSIGNED,
  ADD CONSTRAINT `fk_profile`
    FOREIGN KEY (`profile_id`)
    REFERENCES `profiles` (`profile_id`)
    ON DELETE SET NULL
    ON UPDATE CASCADE;

-- 4. Buat view
CREATE VIEW `v_active_users` AS
SELECT `user_id`, `username`, `email`
FROM `users`
WHERE `status` = 'active';

-- 5. Buat trigger
DELIMITER //
CREATE TRIGGER `trg_users_bi`
  BEFORE INSERT ON `users`
  FOR EACH ROW
BEGIN
  SET NEW.`created_at` = IFNULL(NEW.`created_at`, CURRENT_TIMESTAMP);
END;//
DELIMITER ;

-- 6. Inspect & cleanup
DESCRIBE `users`;
EXPLAIN SELECT * FROM `users` WHERE `username` = 'admin';
DROP VIEW IF EXISTS `v_active_users`;
DROP TRIGGER IF EXISTS `trg_users_bi`;
DROP TABLE IF EXISTS `users`;
DROP DATABASE IF EXISTS `my_app`;
````

---

## 8. Contoh Lengkap

```sql
-- 1. Buat database
CREATE DATABASE IF NOT EXISTS `my_app`
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
USE `my_app`;

-- 2. Buat tabel users
CREATE TABLE `users` (
  `user_id` INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `username` VARCHAR(50) NOT NULL UNIQUE,
  `email` VARCHAR(100) NOT NULL,
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- 3. Alter tabel: tambahkan profile_id
ALTER TABLE `users`
  ADD COLUMN `profile_id` INT UNSIGNED,
  ADD CONSTRAINT `fk_profile`
    FOREIGN KEY (`profile_id`)
    REFERENCES `profiles` (`profile_id`)
    ON DELETE SET NULL
    ON UPDATE CASCADE;

-- 4. Buat view
CREATE VIEW `v_active_users` AS
SELECT `user_id`, `username`, `email`
FROM `users`
WHERE `status` = 'active';

-- 5. Buat trigger
DELIMITER //
CREATE TRIGGER `trg_users_bi`
  BEFORE INSERT ON `users`
  FOR EACH ROW
BEGIN
  SET NEW.`created_at` = IFNULL(NEW.`created_at`, CURRENT_TIMESTAMP);
END;//
DELIMITER ;

-- 6. Inspect & cleanup
DESCRIBE `users`;
EXPLAIN SELECT * FROM `users` WHERE `username` = 'admin';
DROP VIEW IF EXISTS `v_active_users`;
DROP TRIGGER IF EXISTS `trg_users_bi`;
DROP TABLE IF EXISTS `users`;
DROP DATABASE IF EXISTS `my_app`;
```

> File ini **spesifik** untuk MySQL (dengan InnoDB, utf8mb4). Sesuaikan lagi opsi dengan kebutuhanmu.
