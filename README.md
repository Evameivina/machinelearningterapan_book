# Laporan Proyek Machine Learning - Eva Meivina Dwiana

## Domain Proyek

# Sistem Rekomendasi Buku - Goodbooks-10k

## Latar Belakang Proyek

Proyek ini bertujuan untuk membangun sistem rekomendasi buku menggunakan dataset **Goodbooks-10k** yang tersedia di platform Kaggle. Dataset ini mencakup informasi lebih dari 10.000 buku, termasuk rating pengguna, tag, serta metadata seperti judul, penulis, tahun terbit, dan bahasa. Melalui proyek ini, dilakukan eksplorasi dan analisis data untuk memahami karakteristik buku dan preferensi pengguna, dengan tujuan akhir mengembangkan sistem rekomendasi yang mampu menyajikan rekomendasi buku secara personal dan relevan.

Kebutuhan akan sistem rekomendasi yang efektif semakin meningkat, terutama dalam era digital saat ini, di mana pengguna dihadapkan pada begitu banyak pilihan. Sistem rekomendasi memainkan peran penting dalam membantu pengguna menemukan konten yang sesuai dengan preferensi mereka, serta meningkatkan pengalaman pengguna secara keseluruhan. 

Menurut riset oleh Zhang et al. (2019), sistem rekomendasi terbukti mampu meningkatkan interaksi pengguna dan mendorong penjualan pada berbagai platform digital, termasuk platform literasi dan e-commerce buku.

**Referensi:**  
Zhang, S., Yao, L., Sun, A., & Tay, Y. (2019). *Deep Learning based Recommender System: A Survey and New Perspectives.* ACM Computing Surveys (CSUR), 52(1), 1-38.  
DOI: [https://doi.org/10.1145/3285029](https://doi.org/10.1145/3285029)

### Problem Statements
1. Bagaimana memberikan rekomendasi buku berdasarkan kemiripan konten (judul dan deskripsi)?
2. Bagaimana membangun sistem yang mampu memberikan top-N rekomendasi personal kepada pengguna?

### Goals
1. Mengembangkan sistem *content-based recommendation* menggunakan TF-IDF dan cosine similarity.
2. Menyajikan daftar 10 buku yang direkomendasikan kepada pengguna berdasarkan satu buku referensi input.

### Solution Approach

- **Solution 1**: Menggunakan *TF-IDF Vectorization* pada fitur teks (judul/deskripsi) dan mengukur *cosine similarity* antar buku.
- **Solution 2 (opsional)**: Sistem berbasis *collaborative filtering* menggunakan matrix rating pengguna (tidak diterapkan penuh dalam notebook ini, tapi dapat dikembangkan).

## Data Understanding

Dataset yang digunakan adalah **Goodbooks-10k**, tersedia di Kaggle pada tautan berikut:  
🔗 [https://www.kaggle.com/datasets/zygmunt/goodbooks-10k](https://www.kaggle.com/datasets/zygmunt/goodbooks-10k)

Dataset ini terdiri dari beberapa file utama, antara lain:

- `books.csv`  
  Berisi metadata buku seperti ID, judul, penulis, tahun publikasi, rating rata-rata, dan lain-lain.
- `ratings.csv`  
  Berisi rating yang diberikan oleh pengguna ke masing-masing buku.
- `book_tags.csv` dan `tags.csv`  
  Informasi mengenai tag yang digunakan pengguna untuk menandai buku.
- `to_read.csv`  
  Daftar buku yang ditandai oleh pengguna untuk dibaca di masa depan.
- `sample_book.xml`  
  File tambahan yang tidak digunakan dalam analisis ini.

### Statistik Dataset:

- Jumlah buku: **10.000**
- Jumlah tag buku: **10.000** (berdasarkan `book_tags`)
- Jumlah penilaian pengguna: **981.756**
- Jumlah pengguna unik yang memberikan rating: **53.424**
- Jumlah tag unik: **34.252**
- Jumlah data daftar "to-read": **912.705**

### Contoh Data

#### books.csv

| id | book_id | title                      | authors              | average_rating | language_code | original_publication_year |
|----|---------|----------------------------|----------------------|----------------|----------------|----------------------------|
| 1  | 2767052 | The Hunger Games           | Suzanne Collins      | 4.34           | eng            | 2008                       |
| 2  | 3       | Harry Potter #1            | J.K. Rowling         | 4.44           | eng            | 1997                       |
| 3  | 41865   | Twilight                   | Stephenie Meyer      | 3.59           | eng            | 2005                       |
| 4  | 2657    | To Kill a Mockingbird      | Harper Lee           | 4.26           | eng            | 1960                       |
| 5  | 4671    | The Great Gatsby           | F. Scott Fitzgerald  | 3.91           | eng            | 1925                       |

#### ratings.csv

| book_id | user_id | rating |
|---------|---------|--------|
| 1       | 314     | 5      |
| 1       | 439     | 3      |
| 1       | 588     | 5      |
| 1       | 1169    | 4      |
| 1       | 1185    | 4      |

---

### Struktur dan Tipe Data `books.csv`

- Total kolom: **23**
- Total baris: **10.000**
- Beberapa kolom penting dan tipe datanya:

| Kolom                      | Tipe Data | Keterangan                                 |
|---------------------------|-----------|--------------------------------------------|
| `book_id`                 | int       | ID unik untuk buku                         |
| `title`                   | object    | Judul buku                                 |
| `authors`                 | object    | Nama penulis                               |
| `original_publication_year` | float   | Tahun terbit pertama                       |
| `average_rating`          | float     | Rata-rata rating buku                      |
| `language_code`           | object    | Kode bahasa buku                           |
| `ratings_count`           | int       | Jumlah rating yang diberikan               |
| `ratings_1` - `ratings_5` | int       | Jumlah rating berdasarkan skala 1-5        |
| `image_url`, `small_image_url` | object | URL gambar sampul                          |

Sebagian kolom seperti `isbn`, `original_title`, dan `language_code` mengandung nilai kosong (`NaN`), sehingga perlu diperhatikan dalam proses preprocessing.

ces][['title', 'authors']]
