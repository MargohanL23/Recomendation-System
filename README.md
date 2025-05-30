# Laporan Proyek Machine Learning - [Nama Anda]

## Project Overview

Proyek ini bertujuan untuk mengembangkan sistem rekomendasi yang efektif untuk perusahaan e-commerce dengan menganalisis dataset transaksi penjualan. Dalam lanskap e-commerce yang kompetitif, kemampuan untuk merekomendasikan produk yang relevan kepada pelanggan adalah kunci untuk meningkatkan penjualan, kepuasan pelanggan, dan retensi. Sistem rekomendasi yang baik dapat membantu pelanggan menemukan produk yang mereka butuhkan atau sukai dengan lebih mudah, mengurangi churn, dan meningkatkan nilai rata-rata pesanan.

Masalah utama yang ingin diselesaikan adalah bagaimana menyediakan rekomendasi produk yang personal dan relevan kepada setiap pelanggan, mengingat banyaknya variasi produk dan perilaku pembelian yang kompleks. Tanpa sistem rekomendasi, pelanggan mungkin kesulitan menavigasi katalog produk yang besar, yang dapat menyebabkan pengalaman belanja yang kurang optimal dan hilangnya peluang penjualan.

Proyek ini mengimplementasikan dua pendekatan utama dalam sistem rekomendasi:
1.  **Content-Based Filtering**: Merekomendasikan item berdasarkan kemiripan atribut item itu sendiri.
2.  **Collaborative Filtering**: Merekomendasikan item berdasarkan perilaku interaksi pengguna dengan item.

Dengan menggabungkan atau membandingkan kedua pendekatan ini, diharapkan dapat dibangun sistem rekomendasi yang mampu memberikan nilai tambah signifikan bagi bisnis.

## Business Understanding

### Problem Statements

1.  **Kesulitan Pelanggan Menemukan Produk Relevan**: Dengan ribuan produk yang tersedia, pelanggan seringkali kesulitan menemukan item yang benar-benar sesuai dengan minat atau kebutuhan mereka, yang dapat mengakibatkan pengalaman belanja yang kurang efisien dan potensi kehilangan penjualan.
2.  **Optimalisasi Penjualan dan Retensi Pelanggan**: Perusahaan perlu strategi untuk meningkatkan penjualan lintas produk (cross-selling) dan menjaga pelanggan tetap terlibat, namun belum memiliki mekanisme otomatis untuk mengidentifikasi produk yang paling mungkin menarik bagi pelanggan individu.

### Goals

1.  **Membangun Sistem Rekomendasi Berbasis Konten**: Mengembangkan model yang mampu merekomendasikan produk berdasarkan deskripsi produk, sehingga pelanggan dapat menemukan item serupa dengan yang sudah mereka minati.
2.  **Membangun Sistem Rekomendasi Kolaboratif**: Mengembangkan model yang mampu merekomendasikan produk berdasarkan perilaku pembelian pengguna lain, sehingga pelanggan dapat menemukan item yang disukai oleh orang-orang dengan selera serupa.
3.  **Mengevaluasi Performa Model**: Mengukur efektivitas kedua model rekomendasi menggunakan metrik yang relevan seperti RMSE, Precision@K, Recall@K, dan F1-score@K untuk memahami kekuatan dan kelemahan masing-masing pendekatan.

## Solution Approach

Kami akan menggunakan dua pendekatan sistem rekomendasi yang berbeda untuk mengatasi masalah ini:

### 1. Content-Based Filtering (CBF)

* **Algoritma**: TF-IDF (Term Frequency-Inverse Document Frequency) dan Cosine Similarity.
* **Cara Kerja**: Mengubah deskripsi produk menjadi representasi numerik (vektor TF-IDF) dan kemudian menghitung kemiripan antar produk menggunakan Cosine Similarity. Rekomendasi diberikan dengan menemukan produk yang paling mirip dengan produk yang diminati pelanggan.
* **Kelebihan**: Tidak memerlukan data interaksi pengguna-item yang banyak (cold-start problem untuk user baru tidak terlalu signifikan), mampu merekomendasikan item baru.
* **Kekurangan**: Terbatas pada fitur item yang tersedia, tidak dapat merekomendasikan item di luar fitur yang ada, dan cenderung merekomendasikan item yang sangat mirip dengan yang sudah diketahui pengguna (kurang dalam serendipity).

### 2. Collaborative Filtering (CF)

* **Algoritma**: Singular Value Decomposition (SVD).
* **Cara Kerja**: Membangun matriks interaksi pengguna-item (dalam kasus ini, berdasarkan Quantity pembelian) dan menggunakan SVD untuk menemukan faktor laten yang menjelaskan preferensi pengguna dan karakteristik item. Model kemudian memprediksi Quantity untuk item yang belum diinteraksi oleh pengguna.
* **Kelebihan**: Mampu menemukan pola kompleks dalam data interaksi, merekomendasikan item yang tidak terduga (serendipity), dan tidak memerlukan informasi konten item.
* **Kekurangan**: Mengalami cold-start problem (sulit merekomendasikan untuk pengguna atau item baru tanpa data interaksi), dan sparsity problem (matriks interaksi seringkali sangat jarang).

## Data Understanding

Dataset yang digunakan adalah Online Retail yang bersumber dari UCI Machine Learning Repository. Dataset ini berisi transaksi e-commerce dari perusahaan retail Inggris antara tahun 2010 dan 2011. Dataset ini memiliki 541.909 baris dan 8 kolom saat dimuat.

Berikut adalah ringkasan jumlah missing value pada tiap kolom:
* `Description`: 1.454 nilai kosong
* `CustomerID`: 135.080 nilai kosong

Variabel-variabel pada dataset Online Retail adalah sebagai berikut:
* `InvoiceNo`: Nomor faktur. Jika diawali dengan huruf 'C' maka itu adalah pembatalan transaksi.
* `StockCode`: Kode unik untuk setiap produk.
* `Description`: Deskripsi atau nama produk.
* `Quantity`: Jumlah produk yang dibeli dalam satu transaksi.
* `InvoiceDate`: Tanggal dan waktu transaksi.
* `UnitPrice`: Harga per unit produk (dalam Pound Sterling).
* `CustomerID`: Nomor unik pelanggan.
* `Country`: Negara tempat pelanggan berada.

### Insight Awal dari Analisis Deskriptif dan Missing Values:

* `Description` dan `CustomerID` memiliki missing values yang signifikan. Kolom `CustomerID` sangat krusial untuk Collaborative Filtering, sehingga baris dengan `CustomerID` kosong perlu ditangani (dihapus).
* `Quantity` dan `UnitPrice` menunjukkan adanya nilai negatif, yang kemungkinan besar mengindikasikan transaksi retur atau pembatalan. Terdapat juga nilai-nilai ekstrem (outlier) yang perlu dibersihkan untuk memastikan kualitas data.
* Distribusi `Quantity` dan `UnitPrice` sangat skewed ke kanan, menunjukkan mayoritas transaksi melibatkan jumlah kecil dan harga rendah, dengan beberapa transaksi besar sebagai outlier.

## Data Preparation

Pada tahap ini, data dibersihkan dan ditransformasi untuk memenuhi persyaratan model Content-Based Filtering dan Collaborative Filtering.

Langkah-langkah Data Preparation:
1.  **Pembersihan Missing Values**:
    * Baris dengan `CustomerID` dan `Description` yang kosong dihapus.
    * **Alasan**: `CustomerID` yang kosong berarti kita tidak tahu siapa pelanggan yang melakukan transaksi, sehingga data tersebut tidak dapat digunakan untuk personalisasi rekomendasi. `Description` yang kosong berarti kita tidak memiliki informasi konten tentang produk tersebut.
2.  **Penanganan Transaksi Tidak Valid**:
    * Baris dengan `InvoiceNo` yang mengandung 'C' (transaksi dibatalkan) dihapus.
    * Baris dengan `Quantity <= 0` dan `UnitPrice <= 0` juga dihapus.
    * **Alasan**: Transaksi yang dibatalkan atau memiliki kuantitas/harga nol/negatif tidak merepresentasikan pembelian yang valid dan dapat bias hasil rekomendasi. Kami hanya ingin menganalisis pembelian yang sebenarnya.
3.  **Pembuatan Fitur `TotalPrice`**:
    * Kolom baru `TotalPrice` dibuat dengan mengalikan `Quantity` dan `UnitPrice`.
    * **Alasan**: `TotalPrice` memberikan gambaran total nilai interaksi pelanggan dengan suatu produk, yang dapat menjadi "rating" yang lebih informatif untuk model Collaborative Filtering dibandingkan hanya `Quantity` saja.
4.  **Label Encoding**:
    * `CustomerID` di-encode menjadi `CustomerIdx` (indeks numerik unik).
    * `Description` di-encode menjadi `ProductIdx` (indeks numerik unik).
    * **Alasan**: Algoritma machine learning, terutama untuk Collaborative Filtering, membutuhkan input dalam format numerik. Label Encoding menyediakan representasi numerik unik untuk setiap pelanggan dan produk.
5.  **Transformasi TF-IDF untuk Content-Based Filtering**:
    * `TfidfVectorizer` diterapkan pada kolom `Description` untuk membuat `tfidf_matrix`.
    * **Alasan**: TF-IDF mengubah teks deskripsi produk menjadi vektor numerik yang merepresentasikan pentingnya kata-kata dalam deskripsi. Ini memungkinkan perhitungan kemiripan antar produk berdasarkan konten tekstual mereka. Stop words bahasa Inggris dihapus untuk fokus pada kata kunci yang lebih informatif.
6.  **Pembentukan DataFrame Interaksi untuk Collaborative Filtering**:
    * Data diagregasi berdasarkan `CustomerIdx` dan `ProductIdx`, dengan menjumlahkan `Quantity` untuk setiap pasangan unik. Ini membentuk `interaction_df`.
    * **Alasan**: Collaborative Filtering membutuhkan matriks interaksi pengguna-item, di mana setiap entri menunjukkan interaksi (dalam hal ini, total `Quantity` pembelian) antara pengguna dan produk tertentu.
7.  **Pembagian Data (Train-Test Split)**:
    * Data interaksi dibagi menjadi trainset (80%) dan testset (20%) menggunakan `surprise.model_selection.train_test_split`.
    * **Alasan**: Pembagian data ini krusial untuk mengevaluasi performa model secara objektif. Model dilatih pada trainset dan dievaluasi pada testset yang belum pernah dilihat model, mensimulasikan skenario dunia nyata.

## Modeling

Tahapan ini membahas mengenai model sistem rekomendasi yang dibangun untuk menyelesaikan permasalahan, dengan menyajikan contoh top-N recommendation sebagai output.

### 4.1 Content-Based Filtering

* **Prinsip Kerja**: Model ini merekomendasikan item berdasarkan kemiripan atribut antar item. Jika seorang pengguna menyukai suatu produk, maka model akan merekomendasikan produk lain yang memiliki karakteristik serupa.
* **Algoritma**: Kami menggunakan kombinasi TF-IDF untuk representasi teks dan Cosine Similarity untuk mengukur kemiripan.
    * **TF-IDF (TfidfVectorizer)**: Mengubah deskripsi produk menjadi vektor numerik yang menangkap pentingnya kata-kata dalam setiap deskripsi relatif terhadap seluruh korpus produk.
    * **Cosine Similarity**: Mengukur sudut kosinus antara dua vektor TF-IDF. Nilai yang mendekati 1 menunjukkan kemiripan tinggi, sedangkan nilai yang mendekati 0 menunjukkan tidak ada kemiripan.
        $$ \text{similarity}(\mathbf{A}, \mathbf{B}) = \cos(\theta) = \frac{\mathbf{A} \cdot \mathbf{B}}{||\mathbf{A}|| \cdot ||\mathbf{B}||} $$
* **Implementasi**: Fungsi `get_recommendations` mengambil nama produk sebagai input, menemukan vektor TF-IDF-nya, menghitung Cosine Similarity dengan semua produk lain, mengurutkan hasilnya, dan mengembalikan top-N produk paling mirip (tidak termasuk produk itu sendiri).
* **Contoh Top-5 Rekomendasi (Content-Based) untuk produk 'WHITE HANGING HEART T-LIGHT HOLDER'**:
    ```
    [('RED HANGING HEART T-LIGHT HOLDER', 0.8270236730803583),
    ('PINK HANGING HEART T-LIGHT HOLDER', 0.810668632404704),
    ('CREAM HANGING HEART T-LIGHT HOLDER', 0.7668061814073109),
    ('HEART T-LIGHT HOLDER', 0.7627817380574107),
    ('HEART T-LIGHT HOLDER', 0.7627817380574107)]
    ```
    Output ini menunjukkan bahwa model berhasil merekomendasikan produk yang secara semantik sangat mirip, seperti variasi warna dari produk yang sama atau produk dengan tema "heart T-light holder".

* **Kelebihan Content-Based Filtering**:
    * **Transparansi**: Rekomendasi mudah dijelaskan (misalnya, "Anda mungkin menyukai ini karena mirip dengan X").
    * **Tidak Ada Cold-Start untuk Pengguna Baru**: Dapat memberikan rekomendasi kepada pengguna baru asalkan mereka memiliki preferensi awal terhadap satu atau lebih item.
    * **Rekomendasi Item Baru**: Mampu merekomendasikan item baru ke sistem segera setelah item tersebut memiliki deskripsi konten.
* **Kekurangan Content-Based Filtering**:
    * **Over-specialization**: Cenderung merekomendasikan item yang sangat mirip dengan yang sudah disukai pengguna, sehingga membatasi eksplorasi produk baru.
    * **Membutuhkan Data Konten yang Kaya**: Kualitas rekomendasi sangat bergantung pada kualitas dan kelengkapan deskripsi item.

### 4.2 Collaborative Filtering

* **Prinsip Kerja**: Model Collaborative Filtering merekomendasikan item berdasarkan preferensi pengguna lain yang memiliki selera serupa (user-based) atau item yang disukai oleh pengguna yang sama (item-based). Dalam proyek ini, kita menggunakan pendekatan yang mendasari matriks faktorisasi.
* **Algoritma**: Kami menggunakan Singular Value Decomposition (SVD), sebuah algoritma faktorisasi matriks yang populer di sistem rekomendasi. SVD bekerja dengan mendekomposisi matriks interaksi pengguna-item menjadi tiga matriks yang lebih kecil, yang merepresentasikan fitur laten pengguna dan item. Dengan memfaktorisasi matriks, SVD dapat memprediksi rating (atau interaksi, dalam kasus ini adalah jumlah Quantity pembelian) yang belum diketahui antara pengguna dan item.
* **Implementasi**: Model SVD dilatih menggunakan library Surprise pada data interaksi pengguna-produk (`interaction_df`). Data ini sebelumnya telah dibagi menjadi trainset dan testset untuk melatih dan mengevaluasi model secara terpisah. Reader dikonfigurasi untuk memahami skala Quantity yang merupakan interaksi. Setelah model dilatih, prediksi interaksi dilakukan pada testset.
* **Contoh Top-5 Rekomendasi (Collaborative) untuk CustomerID '17850.0'**:
    ```
    Produk Rekomendasi (Description)       Prediksi Interaksi (Quantity)
    O ROUND SNACK BOXES SET OF 4 FRUITS              109.810578
    1 JAM MAKING SET WITH JARS                       105.748366
    2 SET/20 RED RETROSPOT PAPER NAPKINS             105.158300
    3 REGENCY CAKESTAND 3 TIER                       102.812239
    4 LUNCH BAG RED RETROSPOT                        102.502932
    ```
    Output ini menunjukkan produk-produk yang diprediksi akan memiliki interaksi (Quantity) tertinggi dengan CustomerID 17850.0, berdasarkan pola pembelian dari pengguna lain yang serupa.

* **Kelebihan Collaborative Filtering**:
    * **Menemukan Pola Tersembunyi**: Mampu menemukan pola kompleks dalam data interaksi yang mungkin tidak terlihat dari fitur item saja.
    * **Serendipity**: Dapat merekomendasikan item yang tidak terduga atau tidak secara langsung terkait dengan preferensi eksplisit pengguna, tetapi disukai oleh pengguna lain dengan selera serupa.
    * **Tidak Membutuhkan Fitur Item**: Tidak memerlukan informasi konten item, cukup data interaksi.
* **Kekurangan Collaborative Filtering**:
    * **Cold-Start Problem**: Sulit memberikan rekomendasi untuk pengguna atau item baru yang belum memiliki data interaksi yang cukup.
    * **Sparsity Problem**: Kinerja dapat menurun jika matriks interaksi sangat jarang (banyak nilai kosong).
    * **Scalability**: Dapat menjadi mahal secara komputasi untuk dataset yang sangat besar.

## Evaluation

Pada bagian ini, kami mengevaluasi performa model Collaborative Filtering (SVD) menggunakan metrik RMSE, Precision@K, Recall@K, dan F1-score@K.

### RMSE (Root Mean Squared Error)

RMSE mengukur seberapa dekat prediksi model terhadap nilai sebenarnya dari interaksi (Quantity pembelian). Ini adalah metrik yang umum digunakan untuk masalah regresi.
RMSE = sqrt( (1/n) * Σᵢ (pᵢ - rᵢ)² )
Dimana:
* $p_i$ adalah nilai prediksi
* $r_i$ adalah nilai aktual
* $n$ adalah jumlah observasi

**Hasil RMSE**:
RMSE: 80976.7124

Nilai RMSE ini menunjukkan rata-rata "kesalahan" prediksi model dalam memprediksi jumlah Quantity suatu produk yang akan dibeli oleh seorang pelanggan. Angka sebesar ini mengindikasikan bahwa model memiliki rata-rata error prediksi yang cukup besar pada skala Quantity yang digunakan. Penting untuk menginterpretasikannya dalam konteks rentang nilai Quantity aktual dalam dataset Anda, yang memiliki variasi sangat besar.

### Precision@K

Precision@K mengukur proporsi item yang direkomendasikan dalam daftar top-K yang sebenarnya relevan bagi pengguna.
$$ \text{Precision@K} = \frac{\text{Jumlah item relevan di top-K}}{\text{Jumlah item di top-K}} $$

**Hasil Precision@5**:
Precision@5: 0.8438

Ini menunjukkan bahwa sekitar 84.38% dari rekomendasi yang diberikan oleh model untuk setiap pengguna (dalam 5 rekomendasi teratas) merupakan item yang benar-benar relevan (yaitu, memiliki Quantity pembelian $\ge 1$). Ini mengindikasikan model mampu memberikan rekomendasi yang cukup akurat dan tepat sasaran.

### Recall@K

Recall@K mengukur proporsi item relevan yang berhasil ditemukan oleh model dalam daftar top-K rekomendasi dari seluruh item relevan yang ada.
$$ \text{Recall@K} = \frac{\text{Jumlah item relevan di top-K}}{\text{Jumlah semua item relevan}} $$

**Hasil Recall@5**:
Recall@5: 0.6295

Ini mengartikan bahwa model berhasil menemukan sekitar 62.95% dari seluruh item yang relevan untuk setiap pengguna. Artinya, model sudah cukup baik dalam menangkap sebagian besar item yang diminati pengguna, walaupun masih ada ruang untuk peningkatan agar cakupan rekomendasi lebih luas.

### F1-score@K

F1-score@K adalah rata-rata harmonis dari Precision@K dan Recall@K. Ini memberikan ukuran keseimbangan antara presisi dan cakupan, yang berguna ketika ada trade-off antara kedua metrik tersebut.
$$ \text{F1-score@K} = 2 \times \frac{\text{Precision@K} \times \text{Recall@K}}{\text{Precision@K} + \text{Recall@K}} $$

**Hasil F1-score@5**:
F1-score@5: 0.7201

Nilai ini menunjukkan performa keseluruhan model yang cukup baik, dengan keseimbangan yang baik antara kualitas (presisi) dan kuantitas (cakupan) rekomendasi.

### Kesimpulan Evaluasi:

* RMSE sebesar 80976.7124 menunjukkan rata-rata kesalahan prediksi model SVD dalam memprediksi Quantity. Skala Quantity yang besar dalam dataset membuat RMSE terlihat besar, namun ini perlu diinterpretasikan secara kontekstual.
* Precision@5 sebesar 0.8438 menunjukkan bahwa sekitar 84.38% dari rekomendasi yang diberikan adalah relevan.
* Recall@5 sebesar 0.6295 menunjukkan bahwa model berhasil menangkap sekitar 62.95% dari total item relevan.
* F1-score@5 sebesar 0.7201 menunjukkan keseimbangan yang baik antara presisi dan cakupan, mengindikasikan performa model yang solid secara keseluruhan.

Secara keseluruhan, model menunjukkan performa yang cukup baik dalam memberikan rekomendasi yang relevan, dengan potensi untuk meningkatkan cakupan (Recall) agar pengguna mendapatkan lebih banyak rekomendasi relevan tanpa mengorbankan presisi secara signifikan.
