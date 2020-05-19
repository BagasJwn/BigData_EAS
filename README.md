# BigData_EAS

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

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1.PNG)</br>

Pada workflow ini, create env local akan dibuat sebagai spark, dimana dihubungkan ke tiap metanode, tiap metanode berisi workflow yang sama, dengan konfigurasi yang berbeda, workflow dari tiap node adalah sebagai berikut :

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2.PNG)

workflow diatas berisi pada setiap node, yang akan dijelaskan pada tahapan berikutnya, penjelasan disesuaikan berdasarkan data yang dianalisa, dikarenakan data yang di analisa berbeda.


## 3. Daily Minimum Temperature
Workflow yang akan dijalankan pada tugas ini adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/1.PNG)

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
![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/2.PNG)

Pada data preparation kita akan mempersiapkan data sets yang telah ditambahkan id dengan menggunakan database, data sudah berada pada folder /files/ yang disiapkan dalam bentuk spark nantinya.
Node yang dijalankan pertama kali adalah file manger, yaitu akan dilakukan load data yang berasa dari knime, kemudian membuat local env yang kemudian akan jalankan Meta Node ``Load Data``.

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/3.PNG)

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/4.PNG)

Meta node ``Load Data`` berisi 2 node, node ini akan melakukan pembuatan table pada hive serta melakukan load table yang telah dibuat, hasil table yang telah di buat adalah sebagai berikut.

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/5.PNG)

Daily Temp Minimum akan disimpan pada hive dengan nama table daily_temperature.

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/6.PNG)

Selanjutnya yang dilakukan adalah merubah table hive tadi menjadi spark, dengan menjalankan node ``Hive to Spark``, hasil dari table spark tersebut adalah

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/7.PNG)


### 3.4 Modeling
Selanjutnya adalah melakukan modeling untuk merubah isi table yang ada, yang nantinya akan dilakukan pemecahan data untuk dilakukan analisa, workflow yang dijalankan adalah

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/10.PNG)

Pada workflow ini kita akan menjalankan meta node ``Extract time`` node ini akan melakukan pemisahan data yang nantinya akan di lakukan analisa, isi dari node ``Extract time`` ini adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/8.PNG)

Pada node ini, pertama kita akan merubah date yang berbentuk string kedalam bentuk Date, node yang dijalankan adalah node ``Spark SQL Query``

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/9.PNG)

Pada query ini melakukan select data yang berada pada table, yang kemudian data date dilakukan perubahan dari string menjadi bentuk date, sehingga menghasilkan

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/11.PNG)

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

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/12.PNG)

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

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/13.PNG)

Semua node telah dijalankan, hasil column nantinya akan dilakukan analisa pada meta node ``Aggregation and time series``, meta node ini berisi sejumlah node seperti berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/14.PNG)

Pada node ini, akan dilakukan analisa dengan menghitung rata rata dari data yang dianalisa, saya akan menjelaskan beberapa node, dikarenakan node yang lain melakukan hal yang sama, hanya berbeda pada column yang di analisa.

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/15.PNG)

Pada node diatas dilakukan group berdasarkan ``total usage`` dan ``usage by year``, akan melakukan penghitungan berdasarkan temp seperti pada gambar

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/16.PNG)

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/17.PNG)

Sedangkan ``usage by month``

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/18.PNG)

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/19.PNG)

Selanjutnya pada ``avg by month`` dilakukan penghitungan rata-rata berdasarkan bulan, dengan group by tahun

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/20.PNG)

Hasil dari rata-rata adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/21.PNG)

Selanjutnya adalah melakukan column rename, pada column rata-rata, sesuai nama yang kita mau

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/22.PNG)

Selanjutnya dilakukan join pada data sebelumnya, sehingga menghasilkan

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/23.PNG)

Data yang di join adalah data yang berasal dari ``usage by month`` dan ``total usage``, selanjutnya adalah lakukan hal yang sama pada node lainnya, setelah melakukan analisa rata-rata lakukan join pada table tersebut, sehingga hasil akhir dari table yang telah selesai di proses adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/24.PNG)

Data hasil akhir didapat dari data yang telah dipisah sebelumnya yang kemudian dilakukan analisa berdasarkan kebutuhan, inti dari metanode ini adalah untuk mendapatkan analisa berdasarkan data yang telah didapatkan pada metanode sebelumnya.


### 3.5 Evaluation
Pada evaluation akan dijalankan workflow

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/25.PNG)

Pada tahap ini dilakukan select semua data dari node sebelumnya.

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/26.PNG)

Hasilnya adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/27.PNG)

Selanjutnya adalah melakukan Plot K-Mean, PCA, dan memiliki hasil sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/28.PNG)
![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/29.PNG)


### 3.6 Deployment
Selanjutnya pada tahap deployment kita akan menjalankan workflow

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/30.PNG)

Pada tahap ini dilakukan perubahan data dari spark kembali mejadi hive serta menympan spark kedalam HDFS dalam bentuk parquet, hasil dari data tersebut adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/1/31.PNG)


## 4. Electric Production
Workflow yang akan dijalankan pada tugas ini adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/1.PNG)

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
![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/2.PNG)

Pada data preparation kita akan mempersiapkan data sets yang telah ditambahkan id dengan menggunakan database, data sudah berada pada folder /files/ yang disiapkan dalam bentuk spark nantinya.
Node yang dijalankan pertama kali adalah file manger, yaitu akan dilakukan load data yang berasa dari knime, kemudian membuat local env yang kemudian akan jalankan Meta Node ``Load Data``.

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/3.PNG)

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/4.PNG)

Meta node ``Load Data`` berisi 2 node, node ini akan melakukan pembuatan table pada hive serta melakukan load table yang telah dibuat, hasil table yang telah di buat adalah sebagai berikut.

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/5.PNG)

Electric Production akan disimpan pada hive dengan nama table electric_production.

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/6.PNG)

Selanjutnya yang dilakukan adalah merubah table hive tadi menjadi spark, dengan menjalankan node ``Hive to Spark``, hasil dari table spark tersebut adalah

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/7.PNG)


### 4.4 Modeling
Selanjutnya adalah melakukan modeling untuk merubah isi table yang ada, yang nantinya akan dilakukan pemecahan data untuk dilakukan analisa, workflow yang dijalankan adalah

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/10.PNG)

Pada workflow ini kita akan menjalankan meta node ``Extract time`` node ini akan melakukan pemisahan data yang nantinya akan di lakukan analisa, isi dari node ``Extract time`` ini adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/8.PNG)

Pada node ini, pertama kita akan merubah date yang berbentuk string kedalam bentuk Date, node yang dijalankan adalah node ``Spark SQL Query``

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/9.PNG)

Pada query ini melakukan select data yang berada pada table, yang kemudian data date dilakukan perubahan dari string menjadi bentuk date, sehingga menghasilkan

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/11.PNG)

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

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/12.PNG)

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

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/13.PNG)

Semua node telah dijalankan, hasil column nantinya akan dilakukan analisa pada meta node ``Aggregation and time series``, meta node ini berisi sejumlah node seperti berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/14.PNG)

Pada node ini, akan dilakukan analisa dengan menghitung rata rata dari data yang dianalisa, saya akan menjelaskan beberapa node, dikarenakan node yang lain melakukan hal yang sama, hanya berbeda pada column yang di analisa.

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/15.PNG)

Pada node diatas dilakukan group berdasarkan ``total usage`` dan ``usage by year``, akan melakukan penghitungan berdasarkan temp seperti pada gambar

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/16.PNG)

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/17.PNG)

Sedangkan ``usage by month``

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/18.PNG)

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/19.PNG)

Selanjutnya pada ``avg by month`` dilakukan penghitungan rata-rata berdasarkan bulan, dengan group by tahun

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/20.PNG)

Hasil dari rata-rata adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/21.PNG)

Selanjutnya adalah melakukan column rename, pada column rata-rata, sesuai nama yang kita mau

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/22.PNG)

Selanjutnya dilakukan join pada data sebelumnya, sehingga menghasilkan

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/23.PNG)

Data yang di join adalah data yang berasal dari ``usage by month`` dan ``total usage``, selanjutnya adalah lakukan hal yang sama pada node lainnya, setelah melakukan analisa rata-rata lakukan join pada table tersebut, sehingga hasil akhir dari table yang telah selesai di proses adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/24.PNG)

Data hasil akhir didapat dari data yang telah dipisah sebelumnya yang kemudian dilakukan analisa berdasarkan kebutuhan, inti dari metanode ini adalah untuk mendapatkan analisa berdasarkan data yang telah didapatkan pada metanode sebelumnya.


### 4.5 Evaluation
Pada evaluation akan dijalankan workflow

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/25.PNG)

Pada tahap ini dilakukan select semua data dari node sebelumnya.

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/26.PNG)

Hasilnya adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/27.PNG)

Selanjutnya adalah melakukan Plot K-Mean, PCA, dan memiliki hasil sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/28.PNG)
![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/29.PNG)


### 4.6 Deployment
Selanjutnya pada tahap deployment kita akan menjalankan workflow

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/30.PNG)

Pada tahap ini dilakukan perubahan data dari spark kembali mejadi hive serta menympan spark kedalam HDFS dalam bentuk parquet, hasil dari data tersebut adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/2/31.PNG)


## 5. Monthly Beer
Workflow yang akan dijalankan pada tugas ini adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/1.PNG)

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
![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/2.PNG)

Pada data preparation kita akan mempersiapkan data sets yang telah ditambahkan id dengan menggunakan database, data sudah berada pada folder /files/ yang disiapkan dalam bentuk spark nantinya.
Node yang dijalankan pertama kali adalah file manger, yaitu akan dilakukan load data yang berasa dari knime, kemudian membuat local env yang kemudian akan jalankan Meta Node ``Load Data``.

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/3.PNG)

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/4.PNG)

Meta node ``Load Data`` berisi 2 node, node ini akan melakukan pembuatan table pada hive serta melakukan load table yang telah dibuat, hasil table yang telah di buat adalah sebagai berikut.

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/5.PNG)

Monthly Beer akan disimpan pada hive dengan nama table monthly_beer.

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/6.PNG)

Selanjutnya yang dilakukan adalah merubah table hive tadi menjadi spark, dengan menjalankan node ``Hive to Spark``, hasil dari table spark tersebut adalah

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/7.PNG)


### 5.4 Modeling
Selanjutnya adalah melakukan modeling untuk merubah isi table yang ada, yang nantinya akan dilakukan pemecahan data untuk dilakukan analisa, workflow yang dijalankan adalah

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/10.PNG)

Pada workflow ini kita akan menjalankan meta node ``Extract time`` node ini akan melakukan pemisahan data yang nantinya akan di lakukan analisa, isi dari node ``Extract time`` ini adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/8.PNG)

Pada node ini, pertama kita akan merubah date yang berbentuk string kedalam bentuk Date, node yang dijalankan adalah node ``Spark SQL Query``

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/9.PNG)

Pada query ini melakukan select year, month yang diambil dari kolom month, sehingga menghasilkan

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/11.PNG)

Semua node telah dijalankan, hasil column nantinya akan dilakukan analisa pada meta node ``Aggregation and time series``, meta node ini berisi sejumlah node seperti berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/12.PNG)

Pada node ini, akan dilakukan analisa dengan menghitung rata rata dari data yang dianalis.

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/14.PNG)

Pada node diatas dilakukan group berdasarkan ``total usage`` dan ``usage by year``, akan melakukan penghitungan berdasarkan temp seperti pada gambar

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/15.PNG)

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/16.PNG)

Sedangkan ``usage by month``

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/17.PNG)

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/18.PNG)

Selanjutnya pada ``avg by month`` dilakukan penghitungan rata-rata berdasarkan bulan, dengan group by tahun

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/19.PNG)

Hasil dari rata-rata adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/20.PNG)

Selanjutnya adalah melakukan column rename, pada column rata-rata, sesuai nama yang kita mau

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/21.PNG)

Selanjutnya dilakukan join pada data sebelumnya, sehingga menghasilkan

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/22.PNG)

Data yang di join adalah data yang berasal dari ``usage by month`` dan ``total usage``, data didapat dari data yang telah dipisah sebelumnya yang kemudian dilakukan analisa berdasarkan kebutuhan, inti dari metanode ini adalah untuk mendapatkan analisa berdasarkan data yang telah didapatkan pada metanode sebelumnya.


### 5.5 Evaluation
Pada evaluation akan dijalankan workflow

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/23.PNG)

Pada tahap ini dilakukan select semua data dari node sebelumnya.

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/24.PNG)

Hasilnya adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/25.PNG)

Selanjutnya adalah melakukan Plot K-Mean, PCA, dan memiliki hasil sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/26.PNG)
![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/27.PNG)


### 5.6 Deployment
Selanjutnya pada tahap deployment kita akan menjalankan workflow

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/28.PNG)

Pada tahap ini dilakukan perubahan data dari spark kembali mejadi hive serta menympan spark kedalam HDFS dalam bentuk parquet, hasil dari data tersebut adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/3/29.PNG)


## 6. Sales Shampoo
Workflow yang akan dijalankan pada tugas ini adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/1.PNG)

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
![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/2.PNG)

Pada data preparation kita akan mempersiapkan data sets yang telah ditambahkan id dengan menggunakan database, data sudah berada pada folder /files/ yang disiapkan dalam bentuk spark nantinya.
Node yang dijalankan pertama kali adalah file manger, yaitu akan dilakukan load data yang berasa dari knime, kemudian membuat local env yang kemudian akan jalankan Meta Node ``Load Data``.

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/3.PNG)

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/4.PNG)

Meta node ``Load Data`` berisi 2 node, node ini akan melakukan pembuatan table pada hive serta melakukan load table yang telah dibuat, hasil table yang telah di buat adalah sebagai berikut.

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/5.PNG)

Monthly Beer akan disimpan pada hive dengan nama table sales_shampoo.

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/6.PNG)

Selanjutnya yang dilakukan adalah merubah table hive tadi menjadi spark, dengan menjalankan node ``Hive to Spark``, hasil dari table spark tersebut adalah

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/7.PNG)


### 6.4 Modeling
Selanjutnya adalah melakukan modeling untuk merubah isi table yang ada, yang nantinya akan dilakukan pemecahan data untuk dilakukan analisa, workflow yang dijalankan adalah

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/10.PNG)

Pada workflow ini kita akan menjalankan meta node ``Extract time`` node ini akan melakukan pemisahan data yang nantinya akan di lakukan analisa, isi dari node ``Extract time`` ini adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/8.PNG)

Pada node ini, pertama kita akan merubah date yang berbentuk string kedalam bentuk Date, node yang dijalankan adalah node ``Spark SQL Query``

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/9.PNG)

Pada query ini melakukan select year, month yang diambil dari kolom month, sehingga menghasilkan

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/11.PNG)

Semua node telah dijalankan, hasil column nantinya akan dilakukan analisa pada meta node ``Aggregation and time series``, meta node ini berisi sejumlah node seperti berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/12.PNG)

Pada node ini, akan dilakukan analisa dengan menghitung rata rata dari data yang dianalis.

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/14.PNG)

Pada node diatas dilakukan group berdasarkan ``total usage`` dan ``usage by year``, akan melakukan penghitungan berdasarkan temp seperti pada gambar

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/15.PNG)

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/16.PNG)

Sedangkan ``usage by month``

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/17.PNG)

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/18.PNG)

Selanjutnya pada ``avg by month`` dilakukan penghitungan rata-rata berdasarkan day, dengan group by bulan

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/19.PNG)

Hasil dari rata-rata adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/20.PNG)

Selanjutnya adalah melakukan column rename, pada column rata-rata, sesuai nama yang kita mau

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/21.PNG)

Selanjutnya dilakukan join pada data sebelumnya, sehingga menghasilkan

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/22.PNG)

Data yang di join adalah data yang berasal dari ``usage by month`` dan ``total usage``, data didapat dari data yang telah dipisah sebelumnya yang kemudian dilakukan analisa berdasarkan kebutuhan, inti dari metanode ini adalah untuk mendapatkan analisa berdasarkan data yang telah didapatkan pada metanode sebelumnya.


### 6.5 Evaluation
Pada evaluation akan dijalankan workflow

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/23.PNG)

Pada tahap ini dilakukan select semua data dari node sebelumnya.

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/24.PNG)

Hasilnya adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/25.PNG)

Selanjutnya adalah melakukan Plot K-Mean, PCA, dan memiliki hasil sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/26.PNG)
![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/27.PNG)


### 6.6 Deployment
Selanjutnya pada tahap deployment kita akan menjalankan workflow

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/28.PNG)

Pada tahap ini dilakukan perubahan data dari spark kembali mejadi hive serta menympan spark kedalam HDFS dalam bentuk parquet, hasil dari data tersebut adalah sebagai berikut

![](github.com/BagasJwn/BigData_EAS/tree/master/screenshoot/4/29.PNG)


## 7. Referensi                                                                
https://www.knime.com/learning-hub                                                                    
https://github.com/ahmadkikok/bigdata_2019/tree/master/tugas_1_etl-menggunakan-knime                      
https://github.com/ahmadkikok/bigdata_2019/tree/master/tugas_7_iris_meter_spark                      
https://hub.knime.com/knime/spaces/Examples/latest/10_Big_Data/02_Spark_Executor/09_Big_Data_Irish_Meter_on_Spark_only
