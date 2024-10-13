# Tugas Eksplorasi ABD
```markdown
Nama  : Ervina Anggraini  
NRP   : 5026231042  
Kelas : Administrasi Basis Data - A  
Sistem Informasi ITS
```
---

### 1. **Persiapan Database**
![CDM Database](images/image.png)
link database : https://github.com/pthom/northwind_psql/tree/master

### 2. **Database Administration**
**Pengertian**:  
Database Administration melibatkan serangkaian tugas untuk mengelola, mengatur, dan memelihara basis data agar berjalan dengan efisien, aman, dan lancar. DBA bertanggung jawab atas performa database, backup dan recovery data, pengelolaan hak akses pengguna, serta memastikan skema database tetap terjaga. Tugas lain termasuk pengaturan indeks, manajemen partisi, pemantauan penggunaan sumber daya, serta menjamin keamanan dan integritas data.

**Contoh**:
- **Membuat indeks** pada tabel **customers** untuk mempercepat pencarian berdasarkan kolom `city`:
  - Tes kecepatan pencarian sebelum diberi index
    ```sql
    EXPLAIN ANALYZE
    SELECT * FROM customers
    WHERE city = 'New York';
    ```
  - Beri index di tabel customer berdasarkan column city
    ```sql
    CREATE INDEX idx_customers_city ON customers(city);
    ```
  - Tes kecepatan pencarian setelah diberi index
    ```sql
    EXPLAIN ANALYZE
    SELECT * FROM customers
    WHERE city = 'New York';
    ```

- **Melakukan Memory Management**

  Memory Management dalam PostgreSQL adalah proses mengoptimalkan penggunaan memori untuk meningkatkan performa basis data. Pengelolaan memori ini melibatkan pengaturan parameter-parameter penting seperti **shared_buffers**, **work_mem**, **maintenance_work_mem**, dan **effective_cache_size**. 
    1. **shared_buffers** :
      Jumlah memori yang dialokasikan PostgreSQL untuk cache halaman data (data pages). Semakin besar nilai ini, semakin banyak data yang bisa disimpan dalam memori tanpa harus mengakses disk. Nilai default biasanya cukup rendah, dan pada sistem dengan RAM yang besar, sebaiknya parameter ini ditingkatkan.

        ```sql
        SHOW shared_buffers;
        ALTER SYSTEM SET shared_buffers = '256MB';
        ```
    2. **work_mem** :
      Jumlah memori yang dialokasikan untuk operasi per query yang membutuhkan memori tambahan, seperti sorting dan hash join. Nilai ini berlaku per operasi, sehingga jika ada banyak operasi simultan, penggunaan memori bisa bertambah secara signifikan.
    
        ```sql
        SHOW work_mem;
        ALTER SYSTEM SET work_mem = '16MB';
        ```

    3. **maintenance_work_mem** :
      Jumlah memori yang dialokasikan untuk operasi maintenance seperti vacuuming, creating indexes, dan lainnya. Menetapkan nilai yang lebih tinggi untuk parameter ini dapat mempercepat operasi maintenance.
      
        ```sql
        SHOW maintenance_work_mem;
        ALTER SYSTEM SET maintenance_work_mem = '128MB';
        ```

    4. **effective_cache_size** :
      Parameter ini tidak secara langsung mengalokasikan memori, tetapi memberi PostgreSQL perkiraan berapa banyak memori sistem yang dapat digunakan untuk caching oleh OS. Ini membantu PostgreSQL dalam membuat keputusan terkait perencanaan query. Semakin besar nilainya, semakin besar PostgreSQL akan menganggap cache yang tersedia, dan query akan dioptimalkan untuk performa yang lebih baik.
      
        ```sql
        SHOW effective_cache_size;
        ALTER SYSTEM SET effective_cache_size = '8GB';
        ```
    5. **Full Code**
        ```sql
        SHOW shared_buffers;
        SHOW work_mem;
        SHOW maintenance_work_mem;
        SHOW effective_cache_size;

        ALTER SYSTEM SET shared_buffers = '256MB';
        ALTER SYSTEM SET work_mem = '16MB';
        ALTER SYSTEM SET maintenance_work_mem = '128MB';
        ALTER SYSTEM SET effective_cache_size = '8GB';

        SELECT pg_reload_conf();
        ```
- **Melakukan Backup dan Restore Database**
    
    Backup dan Restore adalah bagian penting dari pengelolaan database untuk melindungi data dari kegagalan sistem, kerusakan, atau kehilangan data. PostgreSQL menyediakan beberapa metode untuk melakukan backup dan restore, tergantung pada skenario yang dihadapi.

    Untuk Melakukan Backup Database, di CLI ketikkan :
    ```bash
    \! pg_dump -U postgres -d postgres -t customers -t customer_customer_demo -t customer_demographics -Fc -f "[BACKUP_FILE_LOCATION]"
    ```

    Untuk Melakukan Restore Database, di CLI ketikkan :
    ```bash
    createdb -h localhost -U postgres new_database
    pg_restore -U postgres -d new_database "[BACKUP_FILE_LOCATION]"
    ```

- **Mengelola hak akses pengguna** dengan memberikan izin kepada user untuk mengakses data:
  - admin_northwind: bisa melakukan CRUD di tabel order dan employee
    ```sql
    CREATE USER admin_northwind WITH PASSWORD 'password123';
    CREATE ROLE admin_role;
    GRANT SELECT, INSERT, UPDATE, DELETE ON orders TO admin_role;
    GRANT SELECT, INSERT, UPDATE, DELETE ON employees TO admin_role;
    GRANT admin_role TO admin_northwind;
    ALTER USER admin_northwind SET ROLE admin_role;
    ```
  - Sales_northwind: select tabel product, insert dan update table supplier
    ```sql
    CREATE USER sales_northwind WITH PASSWORD 'password123';
    CREATE ROLE sales_role;
    GRANT SELECT ON products TO sales_role;
    GRANT INSERT, UPDATE ON suppliers TO sales_role;
    GRANT sales_role TO sales_northwind;
    ALTER USER sales_northwind SET ROLE sales_role;
    ```
  - Hr_northwind: CRUD di table employee.
    ```sql
    CREATE USER hr_northwind WITH PASSWORD 'password123';
    CREATE ROLE hr_role;
    GRANT SELECT, INSERT, UPDATE, DELETE ON employees TO hr_role;
    GRANT hr_role TO hr_northwind;
    ALTER USER hr_northwind SET ROLE hr_role;
    ```
  - Testing User Permission
    ```sql
    -- 1. admin_northwind 
    -- Test CRUD operations on orders table
    -- Create
    INSERT INTO orders (order_id, customer_id, order_date) VALUES (7777, 'ALFKI', CURRENT_DATE);
    -- Read
    SELECT * FROM orders WHERE order_id = 10280;
    -- Update
    UPDATE orders SET customer_id = 'ANATR' WHERE order_id = 10280;
    -- Delete
    DELETE FROM orders WHERE order_id = 7777;

    -- Test CRUD operations on employees table
    -- Create
    INSERT INTO employees (employee_id, first_name, last_name) VALUES (9999, 'Mario', 'Balotelli');
    -- Read
    SELECT * FROM employees WHERE employee_id = 9999;
    -- Update
    UPDATE employees SET first_name = 'John' WHERE employee_id = 9999;
    -- Delete
    DELETE FROM employees WHERE employee_id = 9999;

    -- Attempt to perform operations on tables that are not permitted
    -- Try to SELECT from products table
    SELECT * FROM products;
    -- Try to INSERT into suppliers table
    INSERT INTO suppliers (supplier_id, company_name, contact_name) VALUES (9999, 'PT Mencari Cinta Sejati', 'Heru');

    -- 2. sales_northwind
    -- Test SELECT operation on products table
    SELECT * FROM products;
    -- Test INSERT and UPDATE operations on suppliers table
    -- Create
    INSERT INTO suppliers (supplier_id, company_name, contact_name) VALUES (9988, 'PT Ahahahaha', 'Mas Kulin');
    -- Update
    UPDATE suppliers SET contact_name = 'Mba Nana';
    -- (Attempting DELETE should fail)
    DELETE FROM suppliers WHERE supplier_id = 9988;

    -- 3. hr_northwind
    -- Test CRUD operations on employees table
    -- Create
    INSERT INTO employees (employee_id, first_name, last_name) VALUES (9998, 'Sultan', 'Razzaqi');
    -- Read
    SELECT * FROM employees WHERE employee_id = 9998;
    -- Update
    UPDATE employees SET first_name = 'Luki' WHERE employee_id = 9998;
    -- Delete
    DELETE FROM employees WHERE employee_id = 9998;
    -- (Attempting SELECT should fail)
    SELECT * FROM orders;
    ```
---

### 3. **Database Transaction**
**Pengertian**:  
Database Transaction adalah unit kerja yang dilakukan dalam satu rangkaian tindakan, di mana semua perintah SQL di dalamnya harus dijalankan sepenuhnya atau dibatalkan seluruhnya jika terjadi kesalahan. Transaksi diatur oleh empat prinsip yang dikenal dengan istilah **ACID**:
- **Atomicity**: Transaksi harus dieksekusi secara keseluruhan atau tidak sama sekali. Jika ada bagian dari transaksi yang gagal, semua perubahan yang telah dilakukan akan dibatalkan.
- **Consistency**: Transaksi harus membawa basis data dari satu keadaan valid ke keadaan valid lainnya. Konsistensi ini dipastikan dengan aturan-aturan seperti constraint atau foreign key.
- **Isolation**: Transaksi harus terisolasi satu sama lain sehingga hasil dari suatu transaksi tidak dapat dipengaruhi oleh transaksi lain yang berjalan bersamaan.
- **Durability**: Setelah transaksi di-commit, hasil dari transaksi tersebut harus permanen, bahkan jika terjadi kegagalan sistem.

**Contoh**:  
Menambahkan pesanan baru beserta detailnya ke dalam tabel **orders** dan **order_details** menggunakan transaksi:
```sql
BEGIN;

INSERT INTO orders (order_id, customer_id, employee_id, order_date)
VALUES (10001, 'CUST1', 1, '2024-10-13');

INSERT INTO order_details (order_id, product_id, unit_price, quantity)
VALUES (10001, 2, 20.00, 5);

COMMIT;
```
Jika ada kesalahan selama proses, transaksi dapat di-*rollback* agar perubahan dibatalkan.

---

### 4. **Concurrency Control**
**Pengertian**:  
Concurrency Control adalah mekanisme yang memastikan bahwa beberapa transaksi yang berjalan secara bersamaan tidak mengganggu satu sama lain, terutama ketika transaksi mengakses atau mengubah data yang sama. Tujuan dari concurrency control adalah menjaga **konsistensi** dan **integritas** data meskipun banyak pengguna atau aplikasi mengakses data secara bersamaan.

**Contoh**:  
Mengunci data pelanggan dengan **FOR UPDATE** agar transaksi lain tidak dapat mengubah data yang sama:
```sql
BEGIN;

SELECT * FROM customers WHERE customer_id = 'CUST1' FOR UPDATE;

UPDATE customers SET city = 'New York' WHERE customer_id = 'CUST1';

COMMIT;
```
Ini menghindari konflik ketika dua transaksi mencoba memperbarui baris yang sama secara bersamaan.
