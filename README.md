# Tugas EAS dengan Menggunakan 4 Datasets
Tugas EAS ini berisi analisa mengenai 4 datasets time series yang berbeda.

## Menu Cepat
1. [Kebutuhan](#1-kebutuhan)
2. [Pendahuluan](#2-pendahuluan)
3. [Daily Minimum Temperature](#3-daily-minimum-temperature)
	- [Business Understanding](#31-business-understanding)
	- [Data Understanding](#32-data-understanding)
	- [Data Preparation](#33-data-preparation)
	- [Modeling](#34-modeling)
	- [Evaluation](#35-evaluation)
	- [Deployment](#36-deployment)
4. [Electric Production](#4-electric-production)
	- [Business Understanding](#41-business-understanding)
	- [Data Understanding](#42-data-understanding)
	- [Data Preparation](#43-data-preparation)
	- [Modeling](#44-modeling)
	- [Evaluation](#45-evaluation)
	- [Deployment](#46-deployment)
5. [Monthly Beer](#5-monthly-beer)
	- [Business Understanding](#51-business-understanding)
	- [Data Understanding](#52-data-understanding)
	- [Data Preparation](#53-data-preparation)
	- [Modeling](#54-modeling)
	- [Evaluation](#55-evaluation)
	- [Deployment](#56-deployment)
6. [Sales Shampoo](#6-sales-shampoo)
	- [Business Understanding](#61-business-understanding)
	- [Data Understanding](#62-data-understanding)
	- [Data Preparation](#63-data-preparation)
	- [Modeling](#64-modeling)
	- [Evaluation](#65-evaluation)
	- [Deployment](#66-deployment)
7. [Referensi](#7-referensi)

## 1. Kebutuhan
- KNIME
- Time Series Datasets (https://www.kaggle.com/shenba/time-series-datasets)


## 2. Pendahuluan
Pada tugas eas ini, akan dilakukan analisa pada 4 datasets time series, antara lain :
- Daily minimum temperature
- Electric Production
- Monthly Beer
- Sales Shampoo

Dimana workflow yang dibuat adalah sebagai berikut :

!https://github.com/BagasJwn/BigData_EAS/blob/master/screenshoot/1.PNG

Pada workflow ini, create env local akan dibuat sebagai spark, dimana dihubungkan ke tiap metanode, tiap metanode berisi workflow yang sama, dengan konfigurasi yang berbeda, workflow dari tiap node adalah sebagai berikut :

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2.PNG)

workflow diatas berisi pada setiap node, yang akan dijelaskan pada tahapan berikutnya, penjelasan disesuaikan berdasarkan data yang dianalisa, dikarenakan data yang di analisa berbeda.


## 3. Daily Minimum Temperature
Workflow yang akan dijalankan pada tugas ini adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/1.PNG)

Workflow ini berisi 3 meta node diantara lain ``Load Data Node``, ``Extract date-time attributes``, ``Aggregation and time series``.


### 3.1 Business Understanding
Data test yang digunakan pada workflow ini adalah data Time Series Daily Minimum Temperature, sehingga kemungkinan proses yang dapat dilakukan pada data ini adalah melakukan analisa terhadap minimal temperature, analisa yang dilakukan adalah sebagai berikut :
- Analisa berdasarkan Tahun (Total Tahun)
- Analisa berdasarkan Bulan (Rata-rata tiap Bulan)
- Analisa berdasarkan Minggu (Rata-rata tiap Minggu)
- Analisa berdasarkan Hari dalam Seminggu (Rata-rata tiap Hari Senin-Minggu, Rata-rata tiap hari)
- Analisa berdasarkan Hari Libur dan Hari Kerja (Rata-rata tiap Hari Libur dan Kerja)


### 3.2 Data Understanding
Datasets ini berisi sejumlah data yang berisi penggunaan minimum temp.

Terdapat 3 kolom data dengan :
- id_daily sebuah integer yang menjadi id utama pada data ini.
- tempdate sebuah string yang berisi waktu.
- dailymintemp sebuah double yang berisi data mintemp.


### 3.3 Data Preparation
![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/2.PNG)

Pada data preparation kita akan mempersiapkan data sets yang telah ditambahkan id dengan menggunakan database, data sudah berada pada folder /files/ yang disiapkan dalam bentuk spark nantinya.
Node yang dijalankan pertama kali adalah file manger, yaitu akan dilakukan load data yang berasa dari knime, kemudian membuat local env yang kemudian akan jalankan Meta Node ``Load Data``.

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/3.PNG)

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/4.PNG)

Meta node ``Load Data`` berisi 2 node, node ini akan melakukan pembuatan table pada hive serta melakukan load table yang telah dibuat, hasil table yang telah di buat adalah sebagai berikut.

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/5.PNG)

Daily Temp Minimum akan disimpan pada hive dengan nama table daily_temperature.

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/6.PNG)

Selanjutnya yang dilakukan adalah merubah table hive tadi menjadi spark, dengan menjalankan node ``Hive to Spark``, hasil dari table spark tersebut adalah

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/7.PNG)


### 3.4 Modeling
Selanjutnya adalah melakukan modeling untuk merubah isi table yang ada, yang nantinya akan dilakukan pemecahan data untuk dilakukan analisa, workflow yang dijalankan adalah

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/10.PNG)

Pada workflow ini kita akan menjalankan meta node ``Extract time`` node ini akan melakukan pemisahan data yang nantinya akan di lakukan analisa, isi dari node ``Extract time`` ini adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/8.PNG)

Pada node ini, pertama kita akan merubah date yang berbentuk string kedalam bentuk Date, node yang dijalankan adalah node ``Spark SQL Query``

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/9.PNG)

Pada query ini melakukan select data yang berada pada table, yang kemudian data date dilakukan perubahan dari string menjadi bentuk date, sehingga menghasilkan

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/11.PNG)

Pada kolom newdate, hasil perubahan -1 hari dari hari aslinya, hal ini terjadi karena sewaktu perubahan dilakukan konversi dalam bentuk timestamp, tapi tidak akan terpengaruh dengan hasil yang bulan, tahun, dan harinya. Selanjutnya adalah melakukan ekstrasi waktu baru untuk mendapatkan tahun, bulan, minggu, hari, dengan menjalankan ``Spark SQL Query``, dengan query yang berisi

```
SELECT 
`id_dailymintemp`,
`temp`,
`tempdate`,
year(`newdate`) as year,
month(`newdate`) as month,
weekofyear(`newdate`) as week,
date_format(`newdate`, 'EEEE') as dayOfWeek

FROM #table# t1
```

Pada query ini melakukan select year, month, week, dayofweek yang diambil dari newdate (date yang telah diekstrasi pada node sebelumnya), sehingga menghasilkan

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/12.PNG)

Selanjutnya adalah melakukan select baru dengan menjalankan query

```
SELECT *, 
CASE 
WHEN dayOfWeek in ('Saturday','Sunday') 	THEN 'WE' 
									        ELSE 'BD' 
END as dayClassifier

from #table#
```

Query ini akan melakukan pembuatan column baru, dengan value nya diambil dari dayOfWeek sebelumnya dengan kondisi dimana Saturday dan Sunday akan di isi dengan WE sedangkan hari lain di isi dengan BD, sehingga menghasilkan

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/13.PNG)

Semua node telah dijalankan, hasil column nantinya akan dilakukan analisa pada meta node ``Aggregation and time series``, meta node ini berisi sejumlah node seperti berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/14.PNG)

Pada node ini, akan dilakukan analisa dengan menghitung rata rata dari data yang dianalisa, saya akan menjelaskan beberapa node, dikarenakan node yang lain melakukan hal yang sama, hanya berbeda pada column yang di analisa.

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/15.PNG)

Pada node diatas dilakukan group berdasarkan ``total usage`` dan ``usage by year``, akan melakukan penghitungan berdasarkan temp seperti pada gambar

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/16.PNG)

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/17.PNG)

Sedangkan ``usage by month``

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/18.PNG)

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/19.PNG)

Selanjutnya pada ``avg by month`` dilakukan penghitungan rata-rata berdasarkan bulan, dengan group by tahun

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/20.PNG)

Hasil dari rata-rata adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/21.PNG)

Selanjutnya adalah melakukan column rename, pada column rata-rata, sesuai nama yang kita mau

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/22.PNG)

Selanjutnya dilakukan join pada data sebelumnya, sehingga menghasilkan

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/23.PNG)

Data yang di join adalah data yang berasal dari ``usage by month`` dan ``total usage``, selanjutnya adalah lakukan hal yang sama pada node lainnya, setelah melakukan analisa rata-rata lakukan join pada table tersebut, sehingga hasil akhir dari table yang telah selesai di proses adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/24.PNG)

Data hasil akhir didapat dari data yang telah dipisah sebelumnya yang kemudian dilakukan analisa berdasarkan kebutuhan, inti dari metanode ini adalah untuk mendapatkan analisa berdasarkan data yang telah didapatkan pada metanode sebelumnya.


### 3.5 Evaluation
Pada evaluation akan dijalankan workflow

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/25.PNG)

Pada tahap ini dilakukan select semua data dari node sebelumnya.

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/26.PNG)

Hasilnya adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/27.PNG)

Selanjutnya adalah melakukan Plot K-Mean, PCA, dan memiliki hasil sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/28.PNG)
![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/29.PNG)


### 3.6 Deployment
Selanjutnya pada tahap deployment kita akan menjalankan workflow

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/30.PNG)

Pada tahap ini dilakukan perubahan data dari spark kembali mejadi hive serta menympan spark kedalam HDFS dalam bentuk parquet, hasil dari data tersebut adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot1/31.PNG)


## 4. Electric Production
Workflow yang akan dijalankan pada tugas ini adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/1.PNG)

Workflow ini berisi 3 meta node diantara lain ``Load Data Node``, ``Extract date-time attributes``, ``Aggregation and time series``.


### 4.1 Business Understanding
Data test yang digunakan pada workflow ini adalah data Time Series Electric Production, sehingga kemungkinan proses yang dapat dilakukan pada data ini adalah melakukan analisa terhadap electric production, analisa yang dilakukan adalah sebagai berikut :
- Analisa berdasarkan Tahun (Total Tahun)
- Analisa berdasarkan Bulan (Rata-rata tiap Bulan)
- Analisa berdasarkan Minggu (Rata-rata tiap Minggu)
- Analisa berdasarkan Hari dalam Seminggu (Rata-rata tiap Hari Senin-Minggu, Rata-rata tiap hari)
- Analisa berdasarkan Hari Libur dan Hari Kerja (Rata-rata tiap Hari Libur dan Kerja)


### 4.2 Data Understanding
Datasets ini berisi sejumlah data yang berisi electric production.

Terdapat 2 kolom data dengan :
- date sebuah string yang berisi waktu.
- IPG2211A2N sebuah double yang berisi data electric production.


### 4.3 Data Preparation
![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/2.PNG)

Pada data preparation kita akan mempersiapkan data sets yang telah ditambahkan id dengan menggunakan database, data sudah berada pada folder /files/ yang disiapkan dalam bentuk spark nantinya.
Node yang dijalankan pertama kali adalah file manger, yaitu akan dilakukan load data yang berasa dari knime, kemudian membuat local env yang kemudian akan jalankan Meta Node ``Load Data``.

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/3.PNG)

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/4.PNG)

Meta node ``Load Data`` berisi 2 node, node ini akan melakukan pembuatan table pada hive serta melakukan load table yang telah dibuat, hasil table yang telah di buat adalah sebagai berikut.

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/5.PNG)

Electric Production akan disimpan pada hive dengan nama table electric_production.

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/6.PNG)

Selanjutnya yang dilakukan adalah merubah table hive tadi menjadi spark, dengan menjalankan node ``Hive to Spark``, hasil dari table spark tersebut adalah

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/7.PNG)


### 4.4 Modeling
Selanjutnya adalah melakukan modeling untuk merubah isi table yang ada, yang nantinya akan dilakukan pemecahan data untuk dilakukan analisa, workflow yang dijalankan adalah

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/10.PNG)

Pada workflow ini kita akan menjalankan meta node ``Extract time`` node ini akan melakukan pemisahan data yang nantinya akan di lakukan analisa, isi dari node ``Extract time`` ini adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/8.PNG)

Pada node ini, pertama kita akan merubah date yang berbentuk string kedalam bentuk Date, node yang dijalankan adalah node ``Spark SQL Query``

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/9.PNG)

Pada query ini melakukan select data yang berada pada table, yang kemudian data date dilakukan perubahan dari string menjadi bentuk date, sehingga menghasilkan

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/11.PNG)

Pada kolom newdate, hasil perubahan -1 hari dari hari aslinya, hal ini terjadi karena sewaktu perubahan dilakukan konversi dalam bentuk timestamp, tapi tidak akan terpengaruh dengan hasil yang bulan, tahun, dan harinya. Selanjutnya adalah melakukan ekstrasi waktu baru untuk mendapatkan tahun, bulan, minggu, hari, dengan menjalankan ``Spark SQL Query``, dengan query yang berisi

```
SELECT 
SELECT 
`DATE`,
`eprod`,
year(`newdate`) as year,
month(`newdate`) as month,
weekofyear(`newdate`) as week,
date_format(`newdate`, 'EEEE') as dayOfWeek

FROM #table# t1
```

Pada query ini melakukan select year, month, week, dayofweek yang diambil dari newdate (date yang telah diekstrasi pada node sebelumnya), sehingga menghasilkan

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/12.PNG)

Selanjutnya adalah melakukan select baru dengan menjalankan query

```
SELECT *, 
CASE 
WHEN dayOfWeek in ('Saturday','Sunday') 	THEN 'WE' 
									        ELSE 'BD' 
END as dayClassifier

from #table#
```

Query ini akan melakukan pembuatan column baru, dengan value nya diambil dari dayOfWeek sebelumnya dengan kondisi dimana Saturday dan Sunday akan di isi dengan WE sedangkan hari lain di isi dengan BD, sehingga menghasilkan

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/13.PNG)

Semua node telah dijalankan, hasil column nantinya akan dilakukan analisa pada meta node ``Aggregation and time series``, meta node ini berisi sejumlah node seperti berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/14.PNG)

Pada node ini, akan dilakukan analisa dengan menghitung rata rata dari data yang dianalisa, saya akan menjelaskan beberapa node, dikarenakan node yang lain melakukan hal yang sama, hanya berbeda pada column yang di analisa.

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/15.PNG)

Pada node diatas dilakukan group berdasarkan ``total usage`` dan ``usage by year``, akan melakukan penghitungan berdasarkan temp seperti pada gambar

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/16.PNG)

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/17.PNG)

Sedangkan ``usage by month``

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/18.PNG)

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/19.PNG)

Selanjutnya pada ``avg by month`` dilakukan penghitungan rata-rata berdasarkan bulan, dengan group by tahun

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/20.PNG)

Hasil dari rata-rata adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/21.PNG)

Selanjutnya adalah melakukan column rename, pada column rata-rata, sesuai nama yang kita mau

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/22.PNG)

Selanjutnya dilakukan join pada data sebelumnya, sehingga menghasilkan

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/23.PNG)

Data yang di join adalah data yang berasal dari ``usage by month`` dan ``total usage``, selanjutnya adalah lakukan hal yang sama pada node lainnya, setelah melakukan analisa rata-rata lakukan join pada table tersebut, sehingga hasil akhir dari table yang telah selesai di proses adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/24.PNG)

Data hasil akhir didapat dari data yang telah dipisah sebelumnya yang kemudian dilakukan analisa berdasarkan kebutuhan, inti dari metanode ini adalah untuk mendapatkan analisa berdasarkan data yang telah didapatkan pada metanode sebelumnya.


### 4.5 Evaluation
Pada evaluation akan dijalankan workflow

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/25.PNG)

Pada tahap ini dilakukan select semua data dari node sebelumnya.

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/26.PNG)

Hasilnya adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/27.PNG)

Selanjutnya adalah melakukan Plot K-Mean, PCA, dan memiliki hasil sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/28.PNG)
![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/29.PNG)


### 4.6 Deployment
Selanjutnya pada tahap deployment kita akan menjalankan workflow

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/30.PNG)

Pada tahap ini dilakukan perubahan data dari spark kembali mejadi hive serta menympan spark kedalam HDFS dalam bentuk parquet, hasil dari data tersebut adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot2/31.PNG)


## 5. Monthly Beer
Workflow yang akan dijalankan pada tugas ini adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/1.PNG)

Workflow ini berisi 3 meta node diantara lain ``Load Data Node``, ``Extract date-time attributes``, ``Aggregation and time series``.


### 5.1 Business Understanding
Data test yang digunakan pada workflow ini adalah data Time Series Monthly Beer Production, sehingga kemungkinan proses yang dapat dilakukan pada data ini adalah melakukan analisa terhadap monthly beer production, analisa yang dilakukan adalah sebagai berikut :
- Analisa berdasarkan Tahun (Total Tahun)
- Analisa berdasarkan Bulan (Rata-rata tiap Bulan)


### 5.2 Data Understanding
Datasets ini berisi sejumlah data yang berisi monthly beer.

Terdapat 2 kolom data dengan :
- Month sebuah string yang berisi waktu.
- Monthly beer production sebuah double yang berisi data produksi beer per bulan.


### 5.3 Data Preparation
![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/2.PNG)

Pada data preparation kita akan mempersiapkan data sets yang telah ditambahkan id dengan menggunakan database, data sudah berada pada folder /files/ yang disiapkan dalam bentuk spark nantinya.
Node yang dijalankan pertama kali adalah file manger, yaitu akan dilakukan load data yang berasa dari knime, kemudian membuat local env yang kemudian akan jalankan Meta Node ``Load Data``.

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/3.PNG)

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/4.PNG)

Meta node ``Load Data`` berisi 2 node, node ini akan melakukan pembuatan table pada hive serta melakukan load table yang telah dibuat, hasil table yang telah di buat adalah sebagai berikut.

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/5.PNG)

Monthly Beer akan disimpan pada hive dengan nama table monthly_beer.

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/6.PNG)

Selanjutnya yang dilakukan adalah merubah table hive tadi menjadi spark, dengan menjalankan node ``Hive to Spark``, hasil dari table spark tersebut adalah

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/7.PNG)


### 5.4 Modeling
Selanjutnya adalah melakukan modeling untuk merubah isi table yang ada, yang nantinya akan dilakukan pemecahan data untuk dilakukan analisa, workflow yang dijalankan adalah

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/10.PNG)

Pada workflow ini kita akan menjalankan meta node ``Extract time`` node ini akan melakukan pemisahan data yang nantinya akan di lakukan analisa, isi dari node ``Extract time`` ini adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/8.PNG)

Pada node ini, pertama kita akan merubah date yang berbentuk string kedalam bentuk Date, node yang dijalankan adalah node ``Spark SQL Query``

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/9.PNG)

Pada query ini melakukan select year, month yang diambil dari kolom month, sehingga menghasilkan

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/11.PNG)

Semua node telah dijalankan, hasil column nantinya akan dilakukan analisa pada meta node ``Aggregation and time series``, meta node ini berisi sejumlah node seperti berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/12.PNG)

Pada node ini, akan dilakukan analisa dengan menghitung rata rata dari data yang dianalis.

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/14.PNG)

Pada node diatas dilakukan group berdasarkan ``total usage`` dan ``usage by year``, akan melakukan penghitungan berdasarkan temp seperti pada gambar

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/15.PNG)

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/16.PNG)

Sedangkan ``usage by month``

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/17.PNG)

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/18.PNG)

Selanjutnya pada ``avg by month`` dilakukan penghitungan rata-rata berdasarkan bulan, dengan group by tahun

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/19.PNG)

Hasil dari rata-rata adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/20.PNG)

Selanjutnya adalah melakukan column rename, pada column rata-rata, sesuai nama yang kita mau

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/21.PNG)

Selanjutnya dilakukan join pada data sebelumnya, sehingga menghasilkan

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/22.PNG)

Data yang di join adalah data yang berasal dari ``usage by month`` dan ``total usage``, data didapat dari data yang telah dipisah sebelumnya yang kemudian dilakukan analisa berdasarkan kebutuhan, inti dari metanode ini adalah untuk mendapatkan analisa berdasarkan data yang telah didapatkan pada metanode sebelumnya.


### 5.5 Evaluation
Pada evaluation akan dijalankan workflow

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/23.PNG)

Pada tahap ini dilakukan select semua data dari node sebelumnya.

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/24.PNG)

Hasilnya adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/25.PNG)

Selanjutnya adalah melakukan Plot K-Mean, PCA, dan memiliki hasil sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/26.PNG)
![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/27.PNG)


### 5.6 Deployment
Selanjutnya pada tahap deployment kita akan menjalankan workflow

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/28.PNG)

Pada tahap ini dilakukan perubahan data dari spark kembali mejadi hive serta menympan spark kedalam HDFS dalam bentuk parquet, hasil dari data tersebut adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot3/29.PNG)


## 6. Sales Shampoo
Workflow yang akan dijalankan pada tugas ini adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/1.PNG)

Workflow ini berisi 3 meta node diantara lain ``Load Data Node``, ``Extract date-time attributes``, ``Aggregation and time series``.


### 6.1 Business Understanding
Data test yang digunakan pada workflow ini adalah data Time Series Sales Shampoo, sehingga kemungkinan proses yang dapat dilakukan pada data ini adalah melakukan analisa terhadap sales shampoo, analisa yang dilakukan adalah sebagai berikut :
- Analisa berdasarkan Bulan (Total Bulan)
- Analisa berdasarkan Hari (Rata-rata tiap Hari)


### 6.2 Data Understanding
Datasets ini berisi sejumlah data yang berisi sales shampoo.

Terdapat 2 kolom data dengan :
- Month sebuah string yang berisi waktu.
- Sales of shampoo sebuah double yang berisi data penjualan shampoo.


### 6.3 Data Preparation
![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/2.PNG)

Pada data preparation kita akan mempersiapkan data sets yang telah ditambahkan id dengan menggunakan database, data sudah berada pada folder /files/ yang disiapkan dalam bentuk spark nantinya.
Node yang dijalankan pertama kali adalah file manger, yaitu akan dilakukan load data yang berasa dari knime, kemudian membuat local env yang kemudian akan jalankan Meta Node ``Load Data``.

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/3.PNG)

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/4.PNG)

Meta node ``Load Data`` berisi 2 node, node ini akan melakukan pembuatan table pada hive serta melakukan load table yang telah dibuat, hasil table yang telah di buat adalah sebagai berikut.

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/5.PNG)

Monthly Beer akan disimpan pada hive dengan nama table sales_shampoo.

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/6.PNG)

Selanjutnya yang dilakukan adalah merubah table hive tadi menjadi spark, dengan menjalankan node ``Hive to Spark``, hasil dari table spark tersebut adalah

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/7.PNG)


### 6.4 Modeling
Selanjutnya adalah melakukan modeling untuk merubah isi table yang ada, yang nantinya akan dilakukan pemecahan data untuk dilakukan analisa, workflow yang dijalankan adalah

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/10.PNG)

Pada workflow ini kita akan menjalankan meta node ``Extract time`` node ini akan melakukan pemisahan data yang nantinya akan di lakukan analisa, isi dari node ``Extract time`` ini adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/8.PNG)

Pada node ini, pertama kita akan merubah date yang berbentuk string kedalam bentuk Date, node yang dijalankan adalah node ``Spark SQL Query``

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/9.PNG)

Pada query ini melakukan select year, month yang diambil dari kolom month, sehingga menghasilkan

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/11.PNG)

Semua node telah dijalankan, hasil column nantinya akan dilakukan analisa pada meta node ``Aggregation and time series``, meta node ini berisi sejumlah node seperti berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/12.PNG)

Pada node ini, akan dilakukan analisa dengan menghitung rata rata dari data yang dianalis.

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/14.PNG)

Pada node diatas dilakukan group berdasarkan ``total usage`` dan ``usage by year``, akan melakukan penghitungan berdasarkan temp seperti pada gambar

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/15.PNG)

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/16.PNG)

Sedangkan ``usage by month``

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/17.PNG)

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/18.PNG)

Selanjutnya pada ``avg by month`` dilakukan penghitungan rata-rata berdasarkan day, dengan group by bulan

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/19.PNG)

Hasil dari rata-rata adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/20.PNG)

Selanjutnya adalah melakukan column rename, pada column rata-rata, sesuai nama yang kita mau

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/21.PNG)

Selanjutnya dilakukan join pada data sebelumnya, sehingga menghasilkan

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/22.PNG)

Data yang di join adalah data yang berasal dari ``usage by month`` dan ``total usage``, data didapat dari data yang telah dipisah sebelumnya yang kemudian dilakukan analisa berdasarkan kebutuhan, inti dari metanode ini adalah untuk mendapatkan analisa berdasarkan data yang telah didapatkan pada metanode sebelumnya.


### 6.5 Evaluation
Pada evaluation akan dijalankan workflow

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/23.PNG)

Pada tahap ini dilakukan select semua data dari node sebelumnya.

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/24.PNG)

Hasilnya adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/25.PNG)

Selanjutnya adalah melakukan Plot K-Mean, PCA, dan memiliki hasil sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/26.PNG)
![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/27.PNG)


### 6.6 Deployment
Selanjutnya pada tahap deployment kita akan menjalankan workflow

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/28.PNG)

Pada tahap ini dilakukan perubahan data dari spark kembali mejadi hive serta menympan spark kedalam HDFS dalam bentuk parquet, hasil dari data tersebut adalah sebagai berikut

![](/BagasJwn/BigData_EAS/tree/master/screenshoot4/29.PNG)


## 7. Referensi                                                                
https://www.knime.com/learning-hub                                                                    
https://github.com/ahmadkikok/bigdata_2019/tree/master/tugas_1_etl-menggunakan-knime                      
https://github.com/ahmadkikok/bigdata_2019/tree/master/tugas_7_iris_meter_spark                      
https://hub.knime.com/knime/spaces/Examples/latest/10_Big_Data/02_Spark_Executor/09_Big_Data_Irish_Meter_on_Spark_only
