# MHEALTH-Unsupervised-Clustering
Projek Statistical Machine Learning Unsupervised Clustering

# MHEALTH-Unsupervised-Clustering

<div align="center">

![R](https://img.shields.io/badge/R-4.5.1-276DC3?style=for-the-badge&logo=r&logoColor=white)
![R Markdown](https://img.shields.io/badge/R%20Markdown-Analysis-blue?style=for-the-badge)
![Machine Learning](https://img.shields.io/badge/Unsupervised%20Learning-Clustering-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)

</div>

# Analisis Clustering Aktivitas Fisik Berdasarkan Data Sensor MHEALTH

*MHEALTH Dataset : UCI Machine Learning Repository*

Proyek ini melakukan analisis **unsupervised learning** dan **clustering** pada dataset **MHEALTH (Mobile Health)**. Dataset ini berisi data sensor tubuh dari beberapa perangkat wearable yang ditempatkan pada tubuh subjek saat melakukan berbagai aktivitas fisik.

Karena dataset MHEALTH memiliki label aktivitas asli (`activity_label`), proses clustering tetap dilakukan secara **unsupervised**, sedangkan label aktivitas hanya digunakan pada tahap **verifikasi atau validasi eksternal**. Analisis ini mencakup preprocessing data sensor time-series, ekstraksi fitur berbasis window, standarisasi, reduksi dimensi menggunakan PCA, penerapan beberapa algoritma clustering, serta validasi hasil cluster.

---

## Daftar Isi

- [Ringkasan Proyek](#ringkasan-proyek)
- [Dataset](#dataset)
- [Variabel](#variabel)
- [Label Aktivitas](#label-aktivitas)
- [Alur Analisis](#alur-analisis)
- [Preprocessing Data Besar](#preprocessing-data-besar)
- [Algoritma Clustering](#algoritma-clustering)
- [Validasi Cluster](#validasi-cluster)
- [Struktur File](#struktur-file)
- [Cara Menjalankan](#cara-menjalankan)
- [Catatan Data Besar](#catatan-data-besar)
- [Sitasi Dataset](#sitasi-dataset)
- [Tentang](#tentang)

---

## Ringkasan Proyek

| Komponen | Keterangan |
|---|---|
| Dataset | MHEALTH / Mobile Health Dataset |
| Sumber | UCI Machine Learning Repository |
| Jenis data | Multivariate Time-Series |
| Jumlah subjek | 10 subjek |
| Jumlah aktivitas | 12 aktivitas fisik + 1 null class |
| Sampling rate | 50 Hz |
| Jumlah baris setelah penggabungan | 1.215.745 baris |
| Jumlah kolom setelah penggabungan | 25 kolom |
| Variabel sensor | 23 variabel sensor |
| Kolom tambahan | `subject_id`, `activity_label` |
| Metode preprocessing utama | Windowing, feature extraction, standardisasi, PCA |
| Metode clustering | K-Means, GMM, Hierarchical, PAM, DBSCAN |
| Penentuan cluster | Elbow Method, Silhouette Method, dan referensi 12 aktivitas |
| Validasi | Internal dan eksternal |
| Metode final | Ditentukan berdasarkan evaluasi hasil clustering |

---

## Dataset

Dataset **MHEALTH (Mobile Health)** dikembangkan untuk benchmark analisis perilaku manusia berbasis **multimodal body sensing**. Data dikumpulkan dari sensor wearable yang dipasang pada beberapa bagian tubuh subjek.

Dataset tersedia di:

> https://archive.ics.uci.edu/dataset/319/mhealth+dataset

Sensor ditempatkan pada:

| Lokasi Sensor | Jenis Sinyal |
|---|---|
| Dada / chest | Accelerometer + ECG 2 lead |
| Pergelangan tangan kanan / right lower arm | Accelerometer, gyroscope, magnetometer |
| Pergelangan kaki kiri / left ankle | Accelerometer, gyroscope, magnetometer |

Pada proyek ini, file `.log` dari seluruh subjek telah digabung menjadi satu file CSV agar lebih mudah dianalisis di RStudio.

Nama file data utama:

```text
mhealth_combined_10_subjects.csv
```

---

## Variabel

Dataset hasil penggabungan memiliki 25 kolom, terdiri dari:

| Kelompok Variabel | Kolom | Keterangan |
|---|---|---|
| Identitas subjek | `subject_id` | Penanda subjek 1 sampai 10 |
| Chest accelerometer | `chest_acc_X`, `chest_acc_Y`, `chest_acc_Z` | Percepatan pada sensor dada |
| ECG | `ECG_1`, `ECG_2` | Sinyal elektrokardiogram dari sensor dada |
| Left ankle accelerometer | `left_ankle_acc_X`, `left_ankle_acc_Y`, `left_ankle_acc_Z` | Percepatan pada pergelangan kaki kiri |
| Left ankle gyroscope | `left_ankle_gyro_X`, `left_ankle_gyro_Y`, `left_ankle_gyro_Z` | Kecepatan sudut pada pergelangan kaki kiri |
| Left ankle magnetometer | `left_ankle_mag_X`, `left_ankle_mag_Y`, `left_ankle_mag_Z` | Orientasi medan magnet pada pergelangan kaki kiri |
| Right arm accelerometer | `right_arm_acc_X`, `right_arm_acc_Y`, `right_arm_acc_Z` | Percepatan pada lengan kanan |
| Right arm gyroscope | `right_arm_gyro_X`, `right_arm_gyro_Y`, `right_arm_gyro_Z` | Kecepatan sudut pada lengan kanan |
| Right arm magnetometer | `right_arm_mag_X`, `right_arm_mag_Y`, `right_arm_mag_Z` | Orientasi medan magnet pada lengan kanan |
| Label aktivitas | `activity_label` | Label aktivitas asli, hanya digunakan untuk validasi |

Variabel `activity_label` **tidak digunakan sebagai input clustering**, tetapi digunakan untuk mengevaluasi apakah cluster yang terbentuk mendekati aktivitas asli.

---

## Label Aktivitas

| Label | Aktivitas |
|---|---|
| 0 | Null class / tidak termasuk aktivitas utama |
| 1 | Standing still |
| 2 | Sitting and relaxing |
| 3 | Lying down |
| 4 | Walking |
| 5 | Climbing stairs |
| 6 | Waist bends forward |
| 7 | Frontal elevation of arms |
| 8 | Knees bending / crouching |
| 9 | Cycling |
| 10 | Jogging |
| 11 | Running |
| 12 | Jump front and back |

Dalam analisis utama, label `0` dapat dihapus agar clustering lebih fokus pada 12 aktivitas utama.

---

## Alur Analisis

1. Load library yang dibutuhkan
2. Membaca dataset gabungan MHEALTH
3. Mendeskripsikan struktur dataset
4. Memeriksa jumlah baris, jumlah kolom, missing value, dan distribusi label aktivitas
5. Menyeleksi variabel sensor sebagai input clustering
6. Melakukan eksplorasi data awal melalui statistik deskriptif, histogram, dan boxplot
7. Melakukan **windowing** pada data time-series sensor
8. Melakukan **feature extraction** pada setiap window
9. Melakukan standarisasi data menggunakan z-score
10. Melakukan reduksi dimensi menggunakan PCA
11. Menghitung jarak Euclidean pada data hasil PCA atau data sampel
12. Menentukan jumlah cluster optimal dengan Elbow Method dan Silhouette Method
13. Menerapkan beberapa metode clustering
14. Melakukan validasi internal dan eksternal
15. Melakukan visualisasi akhir dan interpretasi profil cluster

---

## Preprocessing Data Besar

Dataset MHEALTH memiliki lebih dari satu juta baris, sehingga tidak efisien jika setiap baris sensor mentah langsung digunakan untuk clustering.

Agar proses analisis lebih optimal, data diproses dengan pendekatan:

### 1. Windowing

Data sensor dibagi menjadi beberapa potongan waktu atau *window*. Pada proyek ini digunakan window 2 detik.

Karena sampling rate MHEALTH adalah 50 Hz, maka:

```text
1 detik  = 50 baris data
2 detik  = 100 baris data
```

Dengan windowing, satu unit observasi clustering bukan lagi satu baris sensor mentah, melainkan satu ringkasan pola sensor dalam interval waktu tertentu.

### 2. Feature Extraction

Pada setiap window, dihitung beberapa fitur statistik dari setiap sensor:

| Fitur | Keterangan |
|---|---|
| Mean | Rata-rata sinyal dalam window |
| Standard deviation | Variasi sinyal dalam window |
| Minimum | Nilai minimum dalam window |
| Median | Nilai tengah sinyal |
| Maximum | Nilai maksimum dalam window |

Dengan 23 variabel sensor dan 5 fitur statistik, terbentuk:

```text
23 sensor × 5 fitur = 115 fitur hasil ekstraksi
```

### 3. Standardisasi

Semua fitur distandarisasi menggunakan z-score agar perbedaan skala antar sensor tidak mendominasi proses clustering.

### 4. PCA

PCA digunakan untuk mereduksi dimensi dan mempercepat proses clustering, terutama karena fitur hasil ekstraksi cukup banyak.

---

## Algoritma Clustering

| Algoritma | Jenis | Keterangan | Fungsi R |
|---|---|---|---|
| K-Means | Partitioning | Mengelompokkan data berdasarkan centroid | `kmeans()` |
| Gaussian Mixture Model | Model-based | Menggunakan pendekatan probabilistik dan distribusi Gaussian | `Mclust()` |
| Hierarchical Average Linkage | Distance-based | Membentuk dendrogram berdasarkan jarak antar objek | `hclust()`, `cutree()` |
| PAM | Partitioning berbasis medoid | Lebih robust terhadap outlier dibanding K-Means | `pam()` |
| DBSCAN | Density-based | Mengelompokkan berdasarkan kepadatan dan dapat mendeteksi noise | `dbscan()` |

Catatan penting:  
Hierarchical clustering dan PAM tidak dijalankan pada seluruh data mentah karena perhitungan jarak penuh membutuhkan memori sangat besar. Oleh karena itu, metode tersebut diterapkan pada data hasil ekstraksi fitur atau sampel data yang lebih kecil.

---

## Validasi Cluster

Validasi dilakukan untuk menilai kualitas hasil clustering. Karena proses clustering bersifat unsupervised, validasi dibagi menjadi dua pendekatan utama.

### Internal Criteria

Validasi internal menggunakan informasi dari struktur data itu sendiri, tanpa memakai label asli.

| Metrik | Arah Interpretasi | Keterangan |
|---|---|---|
| Silhouette Width | Lebih besar lebih baik | Mengukur kekompakan dan keterpisahan cluster |
| Davies-Bouldin Index | Lebih kecil lebih baik | Mengukur rasio kedekatan antar cluster |
| Calinski-Harabasz Index | Lebih besar lebih baik | Mengukur rasio separasi antar cluster terhadap variasi dalam cluster |

### External Criteria

Validasi eksternal menggunakan label aktivitas asli sebagai pembanding. Label ini tidak digunakan saat clustering, hanya untuk mengevaluasi hasil.

| Metrik | Keterangan |
|---|---|
| Adjusted Rand Index / Rand Index | Mengukur kesesuaian pasangan objek antara cluster dan label asli |
| Purity | Mengukur dominasi label asli dalam setiap cluster |
| Contingency Table | Menunjukkan distribusi label aktivitas pada setiap cluster |

Interpretasi umum:

| Nilai Evaluasi | Makna |
|---|---|
| Semakin tinggi Silhouette | Cluster semakin kompak dan terpisah |
| Semakin tinggi Purity | Setiap cluster semakin homogen terhadap aktivitas tertentu |
| Semakin tinggi ARI / Rand Index | Hasil clustering semakin konsisten dengan label aktivitas asli |

---

## Struktur File

Struktur repository yang disarankan:

```text
MHEALTH-Unsupervised-Clustering/
│
├── README.md
├── mhealth_unsupervised_clustering_optimized_with_interpretation.Rmd
├── mhealth_combined_10_subjects.csv
│
├── outputs/
│   └── hasil_laporan.html
│
└── notes/
    └── README_data.txt
```

Jika file CSV terlalu besar untuk diunggah langsung ke GitHub, gunakan struktur berikut:

```text
MHEALTH-Unsupervised-Clustering/
│
├── README.md
├── mhealth_unsupervised_clustering_optimized_with_interpretation.Rmd
│
└── notes/
    └── README_data.txt
```

Lalu simpan file CSV secara lokal di folder yang sama dengan file `.Rmd` sebelum menjalankan analisis.

---

## Cara Menjalankan

### 1. Clone repository

```bash
git clone https://github.com/USERNAME/MHEALTH-Unsupervised-Clustering.git
cd MHEALTH-Unsupervised-Clustering
```

Ganti `USERNAME` dengan username GitHub Anda.

---

### 2. Siapkan dataset

Pastikan file berikut tersedia:

```text
mhealth_combined_10_subjects.csv
```

Letakkan file tersebut pada folder yang sama dengan file `.Rmd`.

Jika dataset belum tersedia, unduh dataset MHEALTH dari UCI Machine Learning Repository, lalu gabungkan file `.log` seluruh subjek menjadi satu CSV sesuai struktur kolom pada README dataset.

---

### 3. Install package R yang dibutuhkan

Jalankan kode berikut di RStudio:

```r
install.packages(c(
  "data.table",
  "dplyr",
  "tidyr",
  "ggplot2",
  "knitr",
  "rmarkdown",
  "cluster",
  "factoextra",
  "mclust",
  "dbscan",
  "fpc",
  "clusterSim",
  "clusterCrit",
  "clue",
  "reshape2",
  "RColorBrewer",
  "scales"
))
```

---

### 4. Render laporan

Jalankan:

```r
rmarkdown::render("mhealth_unsupervised_clustering_optimized_with_interpretation.Rmd")
```

Atau buka file `.Rmd` di RStudio, lalu klik:

```text
Knit → Knit to HTML
```

---

## Catatan Data Besar

File `mhealth_combined_10_subjects.csv` memiliki ukuran besar karena berisi lebih dari satu juta baris data sensor. Jika GitHub menolak upload file CSV, terdapat beberapa solusi:

### Opsi 1 — Jangan upload CSV ke repository

Tambahkan file CSV ke `.gitignore`:

```text
*.csv
*.RData
*.Rhistory
.Rproj.user/
```

Kemudian tulis instruksi pada README agar pengguna mengunduh dataset secara manual.

### Opsi 2 — Gunakan Git LFS

Jika tetap ingin mengunggah dataset besar, gunakan Git Large File Storage:

```bash
git lfs install
git lfs track "*.csv"
git add .gitattributes
git add mhealth_combined_10_subjects.csv
git commit -m "Add dataset using Git LFS"
git push origin main
```

### Opsi 3 — Gunakan GitHub Release

Dataset dapat diunggah sebagai file tambahan pada halaman **Release**, sedangkan repository utama hanya berisi script dan dokumentasi.

---

## Sitasi Dataset

Jika menggunakan dataset ini, sitasi yang disarankan:

> Banos, O., Garcia, R., & Saez, A. (2014). MHEALTH [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C5TW22

---

## Hasil yang Diharapkan

Analisis ini menghasilkan:

- Dataset hasil windowing dan ekstraksi fitur
- Visualisasi distribusi aktivitas
- Statistik deskriptif sensor
- Plot PCA
- Elbow Method dan Silhouette Method
- Hasil clustering dari K-Means, GMM, Hierarchical, PAM, dan DBSCAN
- Validasi internal dan eksternal
- Profiling cluster berdasarkan aktivitas dan subjek
- Interpretasi akhir terhadap pola aktivitas fisik

---

## Tentang

Proyek ini dibuat untuk tugas **Statistical Machine Learning** dengan fokus pada penerapan **unsupervised learning** dan **clustering** pada data sensor time-series.

Analisis dilakukan dengan menyesuaikan karakteristik dataset MHEALTH yang berukuran besar, sehingga preprocessing dilakukan melalui windowing, feature extraction, standardisasi, dan PCA sebelum clustering.

**Penulis:** Razan Nur Muhammad Ihsan  
**NIM:** 3338240032  
**Project:** MHEALTH-Unsupervised-Clustering
