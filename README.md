## Week 5 Assignment - Banksim Artifical Data
- database : banksim_artificial
- user : cloud_user
- password : 'cloud_user'

- ## Week 1 Assignment - Banksim Data Cleaning
- database : banksim_artificial
- user : cloud_user
- password : 'cloud_user'


flow untuk konfigurasi di database
- buat database
    `CREATE DATABASE banksim_artificial`
- buat username dan password
	`CREATE USER cloud_user WITH ENCRYPTED PASSWORD 'cloud_user';`
- membuka akses dari database banksim_artificial ke cloud user
	`GRANT ALL PRIVILEGES ON DATABASE banksim_artificial TO cloud_user;`
- masuk ke database banksim_artificial sebagai superuser yaitu postgres, jangan lupa password superusernya
    `\c banksim_artificial postgres`   
    muncul notifikasi `You are now connected to database "banksim_artificial" as user "postgres"`
- buka semua akses secara skema public ke cloud user
    `GRANT ALL ON SCHEMA public TO cloud_user;`

masukan data ke database menggunakan python
- menghubungkan ke database menggunakan python dengan alamat host, port, nama database, username dan password
    `CONNECT_DB = "host='localhost' port='5432' dbname='banksim_artificial' user='cloud_user' password='cloud_user'"`
- membuat database dengan pre-defined function `create_new_table(create_table_query)`
- query table untuk bsnet140513_032310 untuk dimasukan ke parameter
    ''' CREATE TABLE bsNet140513 (
        Source VARCHAR(20),
        Target VARCHAR(20),
        Weight NUMERIC(10,2),
        typeTrans VARCHAR(50),
        fraud INTEGER ); '''
- query table untuk bs140513_032310 untuk dimasukan ke parameter
    ''' CREATE TABLE bs140513 (
        step INTEGER,
        customer VARCHAR(20),
        age VARCHAR(2),
        gender VARCHAR(1),
        zipcodeOri VARCHAR(10),
        merchant VARCHAR(20),
        zipMerchant VARCHAR(10),
        category VARCHAR(50),
        amount NUMERIC(10,2),
        fraud INTEGER ); '''
- memasukan data dengan pre-defined function `insert_data_table(data_csv, table_in_database)`
- query untuk memasukan table bsnet140513_032310
    - `insert_data_table('bs140513_032310.csv','bs140513')`
    - parameter 1 merupakan nama file csv yaitu `bs140513_032310.csv`
    - parameter 2 merupakan nama table yang tersimpan di database yaitu `bs140513`
- query untuk memasukan table bs140513_032310
    - `insert_data_table('bsNET140513_032310.csv', 'bsnet140513')`
    - parameter 1 merupakan nama file csv yaitu `bsnet140513_032310.csv`
    - parameter 2 merupakan nama table yang tersimpan di database yaitu `bsnet140513`
- membuat pre-defined function untuk fetch query dari database `fetch_query(sql_query)` untuk sekedar query dan melihat-lihat data dari database
- membuat pre-defined function untuk fetch query dan disimpan di dataframe dengan `pandas_db_server_fetch(sql_query)` untuk menampung data ke dalam database
- setelah membuat dataframe baru dari database, masing-masing menjalankan function string replace dengan regex r `str.replace(r"'","")` untuk menghapus kutip di dalam value pada kolom dengan
    - pada `bsnet140513` dilakukan pada kolom `source`, `target` dan `typetrans`
    - pada `bst140513` dilakukan pada kolom `customer`, `age`, `gender`, `zipcodeori`,`merchant`,`zipmerchant` dan `category`
- melakukan filtering data pada `bs140513` karena nilainya yang anomali
    - exclude Gender E and U , untuk memfilter gender yang berisi `E` dan `U` karena anomali
    - exclude Age U, untuk filter data yang `age`nya bukan `U` karena data `U` merupakan anomali
    - diakhiri dengan membuat table baru `bs140513` versi bersihnya
- kolom `age` diubah dari object ke integer karena seharusnya kolom angka menggunakan `astype(str).astype(int)`
- membuat table baru versi bersih `bsnet140513` dengan memilih rows yang customernya hanya terdapat di `bs140513` menggunakan function `isin()` namun menggunakan table berbeda dan valuena serupa
    `df_bsnet140513_clean = df_bsnet140513[df_bsnet140513['source'].isin(df_bs140513_clean['customer'])]`
- membuat kolom baru `fraud status`, untuk mengisi status sesuai `0` jadi `no` dan `1` jadi `yes`
    `df_bsnet140513_clean['fraudstatus'] = df_bsnet140513_clean['fraud'].map({0:'No', 1:'Yes'})`
- membuat table baru `bsnet140513` versi bersih dengan kolom `fraud` yang menjadi `yes` dan `no` 
    `df_bsnet140513_clean = df_bsnet140513_clean.loc[:,df_bsnet140513_clean.columns!='fraud']`
    `df_bsnet140513_clean.columns = ['source', 'target', 'weight', 'typetrans', 'fraud']`
- pada table `bs140513` versi bersihnya mengganti penamaan value dari 
    - `gender` dari `M` menjadi `Male` dan `F` menjadi `Female`
    - `fraud` dari `0` menjadi `no` dan `1` menjadi `yes`
    - `df_bs140513_clean.replace({'gender':{'M':'Male', 'F':'Female'}, 'fraud':{0:'No', 1:'Yes'}}, inplace=True)`
- export data ke csv, tanpa header dan index agar mudah di baca oleh postgresql ketika diimport ke cloud sql
    - `df_bs140513_clean.to_csv('bs140513_clean.csv',index=False, header=False)`
    - `df_bsnet140513_clean.to_csv('bsNET140513_clean.csv',index=False, header=False)`
- pada goocle cloud platform > cloud storage > bucket
    - upload 2 file csv yang sebelumnya di export dari python
    - yaitu `bs140513_clean.csv` dan `bsNET140513_clean.csv` 
- pada google cloud platform > sql > postgresql, masuk ke instance yang sebelumnya dibuat `posgre-firstdata-instance`
    - buat database yang sama pada sql > database yaitu `banksim_artificial`
    - kemudian masuk ke cloud shell dan masukan `gcloud sql connect posgre-firstdata-instance --user=postgres --quiet` untuk akses instance
    - masuk ke database dengan  `\c banksim_artificial`
- kemudian menjalankan query yang baru untuk table bsnet140513 yaitu 
    '''CREATE TABLE bsNet140513_clean (
        Source VARCHAR(20),
        Target VARCHAR(20),
        Weight NUMERIC(10,2),
        typeTrans VARCHAR(50),
        fraud VARCHAR(4));'''    
- kemudian menjalankan query yang baru untuk table bs140513 yaitu 
    ''' CREATE TABLE bs140513_clean (
        step INTEGER,
        customer VARCHAR(20),
        age INTEGER,
        gender VARCHAR(10),
        zipcodeOri VARCHAR(10),
        merchant VARCHAR(20),
        zipMerchant VARCHAR(10),
        category VARCHAR(50),
        amount NUMERIC(10,2),
        fraud VARCHAR(4) ); '''
- melakukan import data pada menu overview untuk memasukan tabel `bs140513_clean.csv` dan `bsNET140513_clean.csv` yang sebelumnya sudah tersimpan di bucket
- membuat Jobs di Dataflow untuk memasukan data `bs140513` menggunakan `template postgresql to bigquery`
- membuat Jobs di Dataflow untuk memasukan data `bsNet140513` menggunakan `template postgresql to bigquery`
- mengaktifkan pipeline
