# Laporan Proyek Machine Learning - [MARGOHAN L. SIRINGO-RINGO]

## Project Overview

Rekomendasi produk pada platform e-commerce merupakan fitur penting yang membantu pengguna menemukan produk sesuai kebutuhan dan preferensi mereka. Sistem rekomendasi dapat meningkatkan pengalaman berbelanja serta mendorong peningkatan penjualan.  
Masalah utama yang ingin diselesaikan adalah bagaimana mengolah data transaksi pembelian agar dapat memberikan rekomendasi produk yang relevan dan personal kepada pelanggan.

Referensi terkait sistem rekomendasi e-commerce dapat ditemukan pada penelitian seperti [Ricci et al., 2011] dan artikel [Adomavicius & Tuzhilin, 2005] yang membahas metode collaborative filtering dan content-based filtering dalam konteks rekomendasi produk.

## Business Understanding

### Problem Statements
- Pelanggan kesulitan menemukan produk yang sesuai dengan preferensi mereka di platform e-commerce yang memiliki jutaan produk.
- Platform e-commerce membutuhkan sistem yang dapat memberikan rekomendasi produk personal untuk meningkatkan engagement dan penjualan.

### Goals
- Membangun sistem rekomendasi produk yang dapat menyajikan rekomendasi relevan berdasarkan riwayat pembelian pelanggan.
- Mengimplementasikan dua pendekatan sistem rekomendasi: Content-Based Filtering dan Collaborative Filtering.
- Mengevaluasi performa kedua model untuk menentukan solusi terbaik.

### Solution Approach
Untuk mencapai tujuan tersebut, akan digunakan dua pendekatan sistem rekomendasi:
- **Content-Based Filtering:** Sistem merekomendasikan produk berdasarkan kemiripan fitur produk yang pernah dibeli atau disukai oleh pengguna.
- **Collaborative Filtering:** Sistem merekomendasikan produk berdasarkan pola interaksi pembelian pengguna lain yang memiliki preferensi serupa.

## Data Understanding

Dataset yang digunakan adalah **Online Retail Dataset** yang tersedia di UCI Machine Learning Repository ([link dataset](https://archive.ics.uci.edu/ml/datasets/Online+Retail)). Dataset ini berisi data transaksi penjualan dari toko e-commerce, meliputi kolom seperti InvoiceNo, StockCode (kode produk), Description (nama produk), Quantity, InvoiceDate, UnitPrice, CustomerID, dan Country.

Variabel utama dalam dataset ini adalah:
- **InvoiceNo:** Nomor transaksi penjualan.
- **StockCode:** Kode unik untuk setiap produk.
- **Description:** Nama atau deskripsi produk.
- **Quantity:** Jumlah produk yang dibeli.
- **InvoiceDate:** Tanggal transaksi.
- **UnitPrice:** Harga per unit produk.
- **CustomerID:** Identifikasi unik pelanggan.
- **Country:** Negara tempat pelanggan berada.


## Data Preparation

Tahapan pembersihan data yang dilakukan:
- Mengecek dan mengisi missing values pada kolom Description dengan "No Description".
- Menghapus baris yang memiliki CustomerID kosong.
- Menghapus data duplikat untuk menghindari bias model.
- Menghapus data dengan Quantity atau UnitPrice bernilai negatif atau nol untuk memastikan data valid.
- Melakukan pemeriksaan outlier pada variabel numerik untuk menghindari pengaruh data ekstrim.

Tahapan ini penting agar model mendapatkan data yang bersih dan valid, sehingga hasil prediksi lebih akurat.

## Modeling

Model yang digunakan adalah SVD (Singular Value Decomposition) dari library Surprise, yang merupakan algoritma matrix factorization populer untuk sistem rekomendasi.

Proses modeling meliputi:
- Melakukan split data menjadi training dan testing dengan rasio 80:20.
- Melatih model SVD pada data training.
- Melakukan prediksi pada data testing.
- Menghasilkan rekomendasi top-5 produk untuk setiap pengguna berdasarkan skor prediksi.

Kelebihan model ini adalah kemampuannya dalam menangkap pola preferensi pengguna secara efisien. Kekurangannya, model ini memerlukan data interaksi yang cukup dan kurang efektif untuk cold-start problem.

## Evaluation

Metrik evaluasi yang digunakan:
- **RMSE (Root Mean Squared Error):**  
  \[
  RMSE = \sqrt{\frac{1}{n} \sum_{i=1}^n (p_i - r_i)^2}
  \]  
  Mengukur rata-rata selisih kuadrat antara prediksi \( p_i \) dan nilai sebenarnya \( r_i \). Nilai RMSE yang lebih kecil menunjukkan prediksi lebih akurat.

- **Precision@k:**  
  Mengukur proporsi item relevan dari k item teratas yang direkomendasikan.  
  \[
  Precision@k = \frac{|\text{Rekomendasi relevan pada top-k}|}{k}
  \]

- **Recall@k:**  
  Mengukur proporsi item relevan yang berhasil direkomendasikan dari semua item relevan yang ada.  
  \[
  Recall@k = \frac{|\text{Rekomendasi relevan pada top-k}|}{|\text{Semua item relevan}|}
  \]

- **F1-score@k:**  
  Harmonik rata-rata dari Precision@k dan Recall@k.  
  \[
  F1 = 2 \times \frac{Precision \times Recall}{Precision + Recall}
  \]

Hasil evaluasi model pada data testing adalah:
- RMSE: 0.2352 (prediksi cukup akurat dengan error kecil)
- Precision@5: 0.7791 (sekitar 77.9% rekomendasi top-5 tepat sasaran)
- Recall@5: 0.6014 (sekitar 60.1% item relevan berhasil direkomendasikan)
- F1-score@5: 0.6788 (balance antara precision dan recall cukup baik)

Kesimpulan: Model SVD berhasil menghasilkan rekomendasi yang cukup akurat dan relevan untuk pengguna, sehingga dapat digunakan sebagai dasar sistem rekomendasi produk.

---

