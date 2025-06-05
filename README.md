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
| book_tags         | 999.912 buku          | Tag yang terkait pada buku      |
| ratings           | 981.756 penilaian    | Rating pengguna                 |
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

Pada tahap pemodelan, digunakan metode **Content-Based Filtering (CBF)** untuk merekomendasikan buku berdasarkan kemiripan konten dari fitur-fitur yang tersedia. Pendekatan ini memanfaatkan matriks similarity yang dihitung dari gabungan fitur teks dan numerik, seperti judul buku (menggunakan TF-IDF), penulis, rating, dan tahun terbit, yang telah diproses pada tahap *Data Preparation*.

### Algoritma yang Digunakan

- **Content-Based Filtering dengan Cosine Similarity**  
  Matriks similarity dibangun menggunakan metode cosine similarity terhadap vektor gabungan fitur konten yang telah diproses. Fitur teks (judul) diubah ke dalam representasi numerik dengan TF-IDF, sementara fitur numerik (rating dan tahun terbit) dinormalisasi sebelum digabungkan.

### Fungsi Rekomendasi Buku

Fungsi `recommend_books()` digunakan untuk menghasilkan rekomendasi berdasarkan judul buku tertentu. Fungsi ini mengembalikan daftar buku lain yang paling mirip berdasarkan skor cosine similarity.

Output berisi informasi sebagai berikut:
- Judul buku
- Penulis
- Rating
- Jumlah rating
- Tahun terbit

Rekomendasi dibatasi pada `top_n` buku dengan skor kemiripan tertinggi.

## Evaluation

Untuk mengevaluasi performa sistem rekomendasi Content-Based Filtering, digunakan metrik evaluasi **Precision@K**, **Recall@K**, dan **F1-Score@K** dengan nilai `K` yang divariasikan (3, 5, 10, dan 20). Evaluasi dilakukan terhadap 30 data buku yang dipilih secara acak dari dataset.

### Penjelasan Metrik Evaluasi

- **Precision@K**: proporsi buku relevan (memiliki penulis yang sama) dari total `K` buku yang direkomendasikan.
- **Recall@K**: proporsi buku relevan yang berhasil ditemukan dari seluruh buku relevan yang tersedia.
- **F1-Score@K**: harmonic mean dari Precision dan Recall.

### Hasil Evaluasi

| K  | Precision@K | Recall@K | F1-Score@K |
|----|-------------|----------|------------|
| 3  | 0.5778      | 0.3245   | 0.3438     |
| 5  | 0.5200      | 0.4196   | 0.3846     |
| 10 | 0.4167      | 0.5177   | 0.3814     |
| 20 | 0.3133      | 0.6246   | 0.3540     |

> Evaluasi dilakukan terhadap 30 sampel buku yang dipilih secara acak dari dataset. Nilai precision cenderung menurun seiring bertambahnya `K`, sementara recall meningkat karena cakupan rekomendasi yang lebih luas.
