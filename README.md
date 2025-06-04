# Laporan Proyek Machine Learning - Eva Meivina Dwiana

## Domain Proyek

Industri buku dan literasi menghadapi tantangan dalam membantu pembaca menemukan buku yang sesuai dengan minat mereka, khususnya di era digital dengan banyaknya pilihan yang tersedia. Sistem rekomendasi menjadi solusi penting dalam meningkatkan pengalaman pengguna dan keterlibatan pembaca.

Salah satu pendekatan efektif dalam menyelesaikan permasalahan ini adalah membangun sistem rekomendasi berbasis pembelajaran mesin menggunakan data historis pengguna dan metadata buku. Dataset "Goodbooks-10k" dari Kaggle memberikan data yang ideal untuk membangun model semacam ini karena mencakup informasi rating dan metadata dari lebih dari 10.000 buku dan lebih dari 50.000 pengguna.

**Referensi:**
1. S. D. Lubis, H. H. M. Sitepu, dan D. R. Irawan, "Sistem Rekomendasi Buku Menggunakan Content-Based Filtering Pada Aplikasi Perpustakaan," *Jurnal Coscitech*, vol. 10, no. 2, 2023. [Link PDF](https://ejurnal.umri.ac.id/index.php/coscitech/article/view/5131/2462)
2. Zając, Z. "goodbooks-10k." Kaggle. [https://www.kaggle.com/datasets/zygmunt/goodbooks-10k](https://www.kaggle.com/datasets/zygmunt/goodbooks-10k)



## Business Understanding

### Problem Statement
Bagaimana membantu pengguna dalam menemukan buku yang sesuai dengan preferensi mereka, tanpa harus mencarinya secara manual?

### Goals
Mengembangkan sistem rekomendasi buku menggunakan metode Content-Based Filtering yang dapat memberikan rekomendasi buku yang relevan berdasarkan deskripsi konten buku.

### Solution Statement
Menerapkan metode Content-Based Filtering dengan menggunakan teknik **TF-IDF vectorization** untuk menganalisis kemiripan antar buku berdasarkan deskripsi.


## Data Understanding

Dataset yang digunakan dalam proyek ini adalah **Goodbooks-10k**, yaitu kumpulan data tentang buku, penilaian pengguna, dan tag buku yang dikumpulkan dari Goodreads. Dataset ini dapat diunduh dari [Goodbooks-10k di Kaggle](https://www.kaggle.com/zygmunt/goodbooks-10k).

Dataset ini terdiri dari lima file utama yang saling terhubung:

1. **books.csv**  
   Berisi data detail tentang buku-buku, seperti ID, judul, penulis, tahun terbit, jumlah rating, dan lain-lain. Dataset ini memiliki 10.000 baris (buku).

2. **book_tags.csv**  
   Berisi hubungan antara buku dan tag yang diberikan, serta jumlah kemunculan tag tersebut pada buku tertentu.

3. **ratings.csv**  
   Data penilaian pengguna terhadap buku tertentu, mencakup `user_id`, `book_id`, dan nilai rating (skala 1–5). Dataset ini memiliki hampir satu juta baris penilaian.

4. **tags.csv**  
   Berisi daftar semua tag yang ada beserta ID-nya.

5. **to_read.csv**  
   Daftar buku yang direncanakan untuk dibaca oleh pengguna tertentu (`user_id` dan `book_id`).

### Statistik Ringkas Dataset

| Dataset           | Jumlah Data          | Keterangan                        |
|-------------------|----------------------|---------------------------------|
| books             | 10.000 buku          | Data buku                       |
| book_tags         | 10.000 buku          | Tag yang terkait pada buku      |
| ratings           | 981.756 penilaian    | Rating pengguna                 |
| pengguna (ratings) | 53.424 pengguna unik | Jumlah pengguna pemberi rating |
| tags              | 34.252 tag           | Daftar tag yang tersedia        |
| to_read           | 912.705 daftar baca  | Buku yang akan dibaca pengguna  |

### Penjelasan Variabel pada Dataset

#### books.csv
- **id**: ID unik untuk setiap baris data buku.
- **book_id**: ID unik buku.
- **best_book_id**: ID buku terbaik (kadang mirip book_id).
- **work_id**: ID untuk karya terkait buku.
- **books_count**: Jumlah buku yang terkait (seri atau edisi).
- **isbn**: ISBN versi lama (dapat berisi nilai kosong).
- **isbn13**: ISBN versi 13 digit (ada nilai kosong).
- **authors**: Nama penulis buku.
- **original_publication_year**: Tahun terbit buku.
- **original_title**: Judul asli buku (ada beberapa nilai kosong).
- **title**: Judul buku.
- **language_code**: Kode bahasa buku (ada beberapa nilai kosong).
- **average_rating**: Rating rata-rata buku.
- **ratings_count**: Jumlah total rating yang diterima.
- **work_ratings_count**: Jumlah rating dari seluruh karya terkait.
- **work_text_reviews_count**: Jumlah review teks.
- **ratings_1** sampai **ratings_5**: Jumlah rating per skor (1-5).
- **image_url** dan **small_image_url**: URL gambar sampul buku.

#### book_tags.csv
- **goodreads_book_id**: ID buku yang diberikan tag.
- **tag_id**: ID tag.
- **count**: Banyaknya tag yang diberikan pada buku tersebut.

#### ratings.csv
- **book_id**: ID buku.
- **user_id**: ID pengguna yang memberikan rating.
- **rating**: Nilai rating yang diberikan (1 sampai 5).

#### tags.csv
- **tag_id**: ID tag.
- **tag_name**: Nama tag.

#### to_read.csv
- **user_id**: ID pengguna.
- **book_id**: ID buku yang ingin dibaca.

### Missing Values

Dari pemeriksaan nilai kosong pada dataset `books.csv`, ditemukan bahwa beberapa kolom seperti `isbn`, `isbn13`, `original_title`, dan `language_code` memiliki data kosong yang perlu diperhatikan dalam pengolahan selanjutnya. Dataset lain relatif bersih tanpa nilai kosong.


## Data Preparation

Pada tahap ini, dilakukan beberapa proses penting untuk menyiapkan data buku dan rating agar dapat digunakan dalam model rekomendasi.

1. **Pengecekan Missing Values**  
   Dataset `books` masih memiliki missing values pada kolom `isbn`, `isbn13`, `original_publication_year`, `original_title`, dan `language_code`. Sedangkan dataset `ratings` sudah bersih dari missing values.

2. **Imputasi dan Penghapusan Kolom**  
   - Missing values pada kolom `original_publication_year` diisi dengan rata-rata tahun publikasi.  
   - Kolom `isbn` dan `isbn13` dihapus karena banyak missing dan kurang relevan.  
   - Duplicate data dihapus pada kedua dataset untuk menghindari duplikasi data yang dapat mengganggu hasil.

3. **Penanganan Missing Data Kategorikal**  
   Missing values pada kolom `language_code` diisi dengan modus (nilai yang paling sering muncul) karena merupakan data kategorikal.

4. **Encoding Fitur Kategorikal**  
   Kolom `authors` dan `language_code` diubah menjadi bentuk numerik menggunakan *Label Encoding* agar dapat diproses model.

5. **Normalisasi Fitur Numerik**  
   Fitur numerik seperti `average_rating`, `ratings_count`, dan `original_publication_year` dinormalisasi dengan *MinMaxScaler* agar berada dalam rentang 0 hingga 1, sehingga fitur memiliki skala yang seragam.

6. **Ekstraksi Fitur Teks**  
   Judul buku (`title`) diolah menggunakan *TF-IDF Vectorizer* dengan maksimal 500 fitur dan penghilangan stop words bahasa Inggris untuk menangkap informasi penting dari teks.

7. **Penggabungan Fitur**  
   Fitur numerik yang sudah diskalakan dan fitur TF-IDF digabungkan menjadi satu matriks fitur konten (`feature_matrix`) dengan dimensi (10000, 505).

8. **Perhitungan Similarity Matrix**  
   Matriks kesamaan antar buku dihitung menggunakan cosine similarity pada `feature_matrix`. Matriks similarity ini berbentuk sparse matrix berukuran (10000, 10000) untuk efisiensi memori dan akan digunakan dalam rekomendasi berbasis konten.


## Modeling

Pada tahap pemodelan ini, digunakan metode **Content-Based Filtering** untuk merekomendasikan buku berdasarkan kemiripan konten dari fitur buku yang tersedia. Proses ini menggunakan matriks similarity yang dihitung dari fitur konten buku, seperti judul, penulis, rating, dan tahun terbit yang sudah diproses pada tahap Data Preparation.

### Algoritma yang Digunakan

- **Content-Based Filtering dengan Cosine Similarity**  
  Matriks similarity dihitung dengan metode cosine similarity pada fitur buku yang sudah diolah (termasuk TF-IDF dari judul dan fitur numerik yang sudah diskalakan).

### Fungsi Rekomendasi Buku

- Fungsi `get_book_recommendations` menerima judul buku sebagai input dan mengembalikan daftar rekomendasi buku yang paling mirip berdasarkan similarity score.
- Fungsi ini juga menampilkan informasi tambahan seperti penulis, rating rata-rata, jumlah rating, dan tahun terbit buku rekomendasi.
- Rekomendasi dibatasi pada jumlah `n` buku teratas dengan skor similarity tertinggi (default 5 buku).

### Evaluasi Model

- Dilakukan evaluasi menggunakan metrik **Precision@K**, dengan `k=5` buku rekomendasi.
- Precision diukur dengan membandingkan apakah penulis buku rekomendasi sama dengan penulis buku yang dicari (karena dalam content-based filtering, buku dengan penulis sama biasanya relevan).
- Evaluasi dilakukan pada sample buku sebanyak 30 data secara acak.
- Hasil evaluasi menunjukkan nilai rata-rata precision@5 yang memberikan gambaran performa sistem rekomendasi.

### Kelebihan dan Kekurangan Content-Based Filtering

| Kelebihan                                             | Kekurangan                                                |
|------------------------------------------------------|-----------------------------------------------------------|
| - Mudah dipahami dan diimplementasikan                | - Tidak dapat merekomendasikan buku yang sangat berbeda    |
| - Rekomendasi personal sesuai konten buku yang dipilih| - Rentan terhadap masalah *cold-start* pada buku baru      |
| - Tidak tergantung pada data rating pengguna yang besar| - Kemampuan eksplorasi terbatas karena hanya berdasarkan konten|

### Proses Improvement

- Untuk meningkatkan performa, bisa dilakukan hyperparameter tuning pada TF-IDF vectorizer seperti jumlah fitur maksimum, penggunaan n-gram, dan penghilangan stopwords.
- Selain itu, normalisasi fitur numerik juga dapat disesuaikan agar memberikan bobot yang optimal pada fitur tertentu.
- Namun, dalam implementasi ini, model sederhana sudah cukup sebagai baseline dan fokus utama adalah pada integrasi fitur konten dan evaluasi precision.

## Evaluation

Pada proyek rekomendasi buku berbasis content-based filtering ini, metrik evaluasi yang digunakan adalah **Precision@K** dengan K=5. Metrik ini dipilih karena sesuai dengan konteks problem statement, yaitu sistem rekomendasi yang bertujuan memberikan daftar rekomendasi buku yang relevan dan tepat berdasarkan buku yang dicari pengguna.

### Penjelasan Metrik Precision@K

- **Precision@K** mengukur proporsi item yang direkomendasikan dalam top-K hasil yang relevan dengan kebutuhan pengguna.  
- Dalam konteks ini, "relevan" diartikan sebagai buku-buku yang memiliki penulis (authors) yang sama dengan buku input (query), karena sistem merekomendasikan buku berdasarkan kemiripan konten (content-based filtering).
- Formula Precision@K:

\[
Precision@K = \frac{\text{Jumlah rekomendasi relevan dalam top-K}}{K}
\]

- Nilai Precision@K berkisar dari 0 hingga 1, dimana 1 berarti semua rekomendasi dalam top-K relevan, dan 0 berarti tidak ada rekomendasi relevan sama sekali.

### Hasil Evaluasi

Berdasarkan evaluasi dengan sampel 30 buku dan K=5, didapatkan hasil:

- **Average Precision@5: 0.2333**  
  Artinya, secara rata-rata dari 5 rekomendasi yang diberikan, sekitar 23,33% atau sekitar 1 hingga 2 buku dari 5 rekomendasi memiliki penulis yang sama dengan buku query.

- **Range Precision: 0.0 sampai 1.0**  
  Ini menunjukkan variasi rekomendasi, ada beberapa buku yang mendapat rekomendasi sangat relevan (precision 1.0), dan ada juga yang kurang relevan (precision 0.0).

- Evaluasi ini dilakukan pada 30 sampel buku yang diambil secara acak dari dataset.

### Interpretasi Hasil

- Nilai Precision@5 sebesar 0.2333 menunjukkan bahwa model content-based filtering memberikan rekomendasi yang cukup baik namun masih bisa ditingkatkan.  
- Karena model ini menggunakan kemiripan berbasis konten (seperti fitur teks atau metadata buku), hasil yang ada sudah menggambarkan bahwa model mampu menemukan buku dengan kemiripan, tapi belum sepenuhnya kuat dalam mengidentifikasi buku yang benar-benar relevan dari sisi penulis.  
- Precision@K yang tidak terlalu tinggi dapat dipengaruhi oleh banyak faktor seperti fitur yang digunakan, kualitas data, atau cara perhitungan similarity.
