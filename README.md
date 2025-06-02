# Laporan Proyek Machine Learning - [MARGOHAN L. SIRINGO-RINGO]

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

Saya akan menggunakan dua pendekatan sistem rekomendasi yang berbeda untuk mengatasi masalah ini:

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

Dataset yang digunakan adalah **Online Retail Dataset** yang tersedia di UCI Machine Learning Repository ([link dataset](https://archive.ics.uci.edu/ml/datasets/Online+Retail)). Dataset ini berisi data transaksi penjualan dari toko e-commerce, meliputi kolom seperti InvoiceNo, StockCode (kode produk), Description (nama produk), Quantity, InvoiceDate, UnitPrice, CustomerID, dan Country. Dataset ini berisi transaksi e-commerce dari perusahaan retail Inggris antara tahun 2010 dan 2011. Dataset ini memiliki 541.909 baris dan 8 kolom saat dimuat.

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

### 1. Pembersihan Missing Values
- Baris dengan `CustomerID` dan `Description` yang kosong dihapus.
- **Alasan**: `CustomerID` yang kosong berarti kita tidak tahu siapa pelanggan yang melakukan transaksi, sehingga data tersebut tidak dapat digunakan untuk personalisasi rekomendasi. `Description` yang kosong berarti kita tidak memiliki informasi konten tentang produk tersebut.

### 2. Penanganan Transaksi Tidak Valid
- Baris dengan `InvoiceNo` yang mengandung 'C' (transaksi dibatalkan) dihapus.
- Baris dengan `Quantity <= 0` dan `UnitPrice <= 0` juga dihapus.
- **Alasan**: Transaksi yang dibatalkan atau memiliki kuantitas/harga nol/negatif tidak merepresentasikan pembelian yang valid dan dapat bias hasil rekomendasi. Analisis hanya dilakukan pada transaksi pembelian yang sah.

### 3. Pembuatan Fitur `TotalPrice`
- Kolom baru `TotalPrice` dibuat dengan rumus:

  ```python
  df['TotalPrice'] = df['Quantity'] * df['UnitPrice']
- **Alasan**: TotalPrice memberikan gambaran total nilai interaksi pelanggan dengan suatu produk, yang dapat menjadi 'rating' yang lebih informatif untuk model Collaborative Filtering dibandingkan hanya Quantity saja.

### 4. Label Encoding:
- CustomerID di-encode menjadi CustomerIdx (indeks numerik unik).
- Description di-encode menjadi ProductIdx (indeks numerik unik).

- **Alasan**: Algoritma machine learning, terutama untuk Collaborative Filtering, membutuhkan input dalam format numerik. Label Encoding menyediakan representasi numerik unik untuk setiap pelanggan dan produk.

### 5. Transformasi TF-IDF untuk Content-Based Filtering:
- TfidfVectorizer diterapkan pada kolom Description untuk membuat tfidf_matrix.
- * **TF-IDF (TfidfVectorizer)**: Mengubah deskripsi produk menjadi vektor numerik yang menangkap pentingnya kata-kata dalam setiap deskripsi relatif terhadap seluruh korpus produk.

- **Alasan**: TF-IDF mengubah teks deskripsi produk menjadi vektor numerik yang merepresentasikan pentingnya kata-kata dalam deskripsi. Ini memungkinkan perhitungan kemiripan antar produk berdasarkan konten tekstual mereka. Stop words bahasa Inggris dihapus untuk fokus pada kata kunci yang lebih informatif.

### 6. Pembentukan DataFrame Interaksi untuk Collaborative Filtering:
- Data diagregasi berdasarkan CustomerIdx dan ProductIdx, dengan menjumlahkan Quantity untuk setiap pasangan unik. Ini membentuk interaction_df.

- **Alasan**: Collaborative Filtering membutuhkan matriks interaksi pengguna-item, di mana setiap entri menunjukkan interaksi (dalam hal ini, total Quantity pembelian) antara pengguna dan produk tertentu.

### 7. Pembagian Data (Train-Test Split):
- Data interaksi dibagi menjadi trainset (80%) dan testset (20%) menggunakan train_test_split dari sklearn.model_selection.

- **Alasan**: Pembagian data ini krusial untuk mengevaluasi performa model secara objektif. Model dilatih pada trainset dan dievaluasi pada testset yang belum pernah dilihat model, mensimulasikan skenario dunia nyata.
  
## Modeling

Tahapan ini membahas mengenai model sistem rekomendasi yang dibangun untuk menyelesaikan permasalahan, dengan menyajikan contoh top-N recommendation sebagai output.

### 4.1 Content-Based Filtering

* **Prinsip Kerja**: Model ini merekomendasikan item berdasarkan kemiripan atribut antar item. Jika seorang pengguna menyukai suatu produk, maka model akan merekomendasikan produk lain yang memiliki karakteristik serupa.
* **Algoritma**: Saya menggunakan kombinasi TF-IDF untuk representasi teks dan Cosine Similarity untuk mengukur kemiripan.
    * **Cosine Similarity**: Mengukur sudut kosinus antara dua vektor TF-IDF. Nilai yang mendekati 1 menunjukkan kemiripan tinggi, sedangkan nilai yang mendekati 0 menunjukkan tidak ada kemiripan.
        $$\text{similarity}(\mathbf{A}, \mathbf{B}) = \cos(\theta) = \frac{\mathbf{A} \cdot \mathbf{B}}{||\mathbf{A}|| \cdot ||\mathbf{B}||}$$
* **Implementasi**: Fungsi `get_recommendations` mengambil nama produk sebagai input, menemukan vektor TF-IDF-nya, menghitung Cosine Similarity dengan semua produk lain, mengurutkan hasilnya, dan mengembalikan top-N produk paling mirip (tidak termasuk produk itu sendiri).
* **Contoh Top-5 Rekomendasi (Content-Based) untuk produk 'WHITE HANGING HEART T-LIGHT HOLDER'**:
    ```
    [('RED HANGING HEART T-LIGHT HOLDER', np.float64(0.8270236730803583)),
     ('PINK HANGING HEART T-LIGHT HOLDER', np.float64(0.810668632404704)),
     ('CREAM HANGING HEART T-LIGHT HOLDER', np.float64(0.7668061814073109)),
     ('HEART T-LIGHT HOLDER   ', np.float64(0.7627817380574107)),
     ('HEART T-LIGHT HOLDER', np.float64(0.7627817380574107))]
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
* **Algoritma**: Saya menggunakan Singular Value Decomposition (SVD), sebuah algoritma faktorisasi matriks yang populer di sistem rekomendasi. SVD bekerja dengan mendekomposisi matriks interaksi pengguna-item menjadi tiga matriks yang lebih kecil, yang merepresentasikan fitur laten pengguna dan item. Dengan memfaktorisasi matriks, SVD dapat memprediksi rating (atau interaksi, dalam kasus ini adalah jumlah Quantity pembelian) yang belum diketahui antara pengguna dan item.
* **Implementasi**: Model SVD dilatih menggunakan library Surprise pada data interaksi pengguna-produk (`interaction_df`). Data ini sebelumnya telah dibagi menjadi trainset dan testset untuk melatih dan mengevaluasi model secara terpisah. Reader dikonfigurasi untuk memahami skala Quantity yang merupakan interaksi. Setelah model dilatih, prediksi interaksi dilakukan pada testset.
* **Contoh Top-5 Rekomendasi (Collaborative) untuk CustomerID '17850.0'**:
    ```
    Produk Rekomendasi (Description)       Prediksi Interaksi (Quantity)
    O ASSORTED COLOUR BIRD ORNAMENT                  80995
    1 POPPY'S PLAYHOUSE BEDROOM                      80995
    2 POPPY'S PLAYHOUSE KITCHEN                      80995
    3 FELTCRAFT PRINCESS CHARLOTTE DOLL              80995
    4 IVORY KNITTED MUG COSY                         80995
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

###**Content-Based Filtering**

Model Content-Based Filtering dievaluasi menggunakan metrik top-N rekomendasi dengan pendekatan berikut:

- **Precision@5**: Proporsi item yang direkomendasikan dan terbukti relevan berdasarkan riwayat pengguna.
- **Recall@5**: Proporsi item relevan yang berhasil direkomendasikan.
- **F1-Score@5**: Harmonik antara precision dan recall.

Evaluasi dilakukan dengan cara:
- Memilih 200 pengguna secara acak dari dataset.
- Menggunakan produk terakhir yang dibeli sebagai query rekomendasi.
- Mengukur apakah sistem dapat merekomendasikan produk-produk lain yang telah dibeli sebelumnya oleh pengguna tersebut.

### 📊 Hasil Evaluasi:

| Metrik                | Nilai     |
|------------------------|-----------|
| Precision@5           | 0.0472     |
| Recall@5              | 0.0038     |
| F1-Score@5            | 0.0071     |
| Gagal Rekomendasi     | 5 user     |

> Meskipun hasil evaluasi ini menunjukkan nilai metrik yang rendah, hal ini wajar terjadi pada pendekatan content-based yang hanya menggunakan informasi deskripsi teks, tanpa mempertimbangkan preferensi personal pengguna atau histori pembelian yang lebih dalam.


###**Collaborative Filtering (SVD)**

Untuk mengevaluasi model Collaborative Filtering (menggunakan algoritma SVD dari library Surprise), dilakukan dua jenis pengukuran:

1. RMSE (Root Mean Squared Error)
RMSE digunakan untuk mengukur seberapa jauh prediksi model terhadap interaksi (Quantity) dari nilai sebenarnya pada data uji.

Formula:

$$RMSE = \sqrt{\frac{1}{n} \sum_{i=1}^{n} (p_i - r_i)^2}$$

Nilai RMSE yang lebih rendah menunjukkan performa prediksi yang lebih baik.

2. Precision@5, Recall@5, dan F1-Score@5
Metrik ini digunakan untuk mengukur seberapa relevan hasil rekomendasi top-5 dari model dibandingkan dengan data historis pengguna.

- **Precision@5** mengukur proporsi item yang direkomendasikan dalam top-5 yang benar-benar relevan.
- **Recall@5** mengukur proporsi item relevan yang berhasil direkomendasikan dari semua item relevan.
- **F1-Score@5** adalah harmonic mean dari precision dan recall.

**Hasil Evaluasi:**

| Metrik                | Nilai     |
|------------------------|-----------|
| RMSE                  | 80976.7124 |
| Precision@5           | 1.0000     |
| Recall@5              | 0.3178     |
| F1-Score@5            | 0.4823     |

**Interpretasi:**
- Precision@5 = 1.0 berarti semua rekomendasi yang diberikan oleh model benar-benar pernah diinteraksikan oleh pengguna di data uji.
- Recall@5 sebesar 31.78% berarti sekitar sepertiga dari item relevan berhasil ditangkap oleh model.
- F1-Score berada di angka moderat, menunjukkan keseimbangan yang cukup baik antara presisi dan cakupan.


## Kesimpulan Evaluasi

Berdasarkan evaluasi terhadap kedua pendekatan sistem rekomendasi, berikut kesimpulan yang dapat diambil:

###Collaborative Filtering (SVD)
- Menunjukkan performa **sangat baik dalam Precision@5** (1.0000), artinya semua item yang direkomendasikan adalah relevan berdasarkan data historis pengguna.
- **F1-Score@5 mencapai 0.4823**, mencerminkan keseimbangan yang cukup antara ketepatan dan cakupan.
- Namun, **Recall@5 masih bisa ditingkatkan (0.3178)**, menunjukkan bahwa model belum sepenuhnya menangkap semua preferensi pengguna.

###Content-Based Filtering (TF-IDF + Cosine Similarity)
- Memberikan hasil **Precision@5 sebesar 0.0472**, menunjukkan bahwa sebagian kecil rekomendasi sesuai dengan minat historis pengguna.
- **Recall@5 sangat rendah (0.0038)**, artinya banyak item relevan yang tidak berhasil direkomendasikan.
- **F1-Score@5 hanya 0.0071**, mencerminkan keterbatasan pendekatan ini jika hanya mengandalkan deskripsi produk.

###Catatan
- **Collaborative Filtering** lebih unggul dalam memahami pola pembelian pengguna, terutama saat data interaksi tersedia dengan baik.
- **Content-Based Filtering** cocok sebagai pelengkap saat pengguna baru (cold-start problem) atau saat produk baru ditambahkan.
- Kombinasi kedua pendekatan (hybrid model) bisa menjadi solusi ideal untuk sistem rekomendasi yang lebih komprehensif.

**Kesimpulan Akhir:**  
Model **Collaborative Filtering (SVD)** menghasilkan performa yang lebih kuat dan akurat dalam memberikan rekomendasi dibandingkan model Content-Based Filtering pada dataset ini.

