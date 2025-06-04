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

**Dataset yang digunakan:** [Goodbooks-10k](https://www.kaggle.com/datasets/zygmunt/goodbooks-10k)

### File yang digunakan:
- `books.csv` : Informasi metadata buku (judul, penulis, deskripsi, genre, rating, dll)

### Kolom penting:
- `book_id`
- `title`
- `authors`
- `description`
- `average_rating`

---

## Data Preparation

Langkah-langkah yang dilakukan:
- Mengecek data null dan menghapus baris yang tidak memiliki deskripsi.
- Menggunakan `TfidfVectorizer` untuk mengubah teks deskripsi buku menjadi representasi numerik.
- Menghapus stopwords agar hanya kata-kata penting yang diproses.
- Menghitung cosine similarity antar buku berdasarkan representasi TF-IDF-nya.

---

## Modeling

### Content-Based Filtering

Model ini mencari kemiripan antar buku berdasarkan **konten deskripsi buku** menggunakan teknik **TF-IDF (Term Frequency-Inverse Document Frequency)**. Kemudian, dihitung kemiripan antar deskripsi menggunakan **cosine similarity**.

**Langkah-langkah:**
1. Menggunakan `TfidfVectorizer` untuk mengubah deskripsi menjadi vektor numerik.
2. Menghitung kemiripan antar vektor menggunakan `cosine_similarity`.
3. Menyimpan hasil similarity matrix untuk digunakan dalam pencarian rekomendasi.

**Contoh hasil rekomendasi:**
Jika pengguna menyukai buku *The Hobbit*, maka sistem akan merekomendasikan buku-buku dengan deskripsi yang memiliki kemiripan tinggi, seperti:
- *Eragon*
- *The Fellowship of the Ring*
- *The Silmarillion*

---

## Evaluation

Sistem rekomendasi ini dievaluasi secara kualitatif dengan melihat hasil kemiripan konten dari deskripsi buku. Karena tidak digunakan data interaksi pengguna (seperti rating), maka evaluasi lebih bersifat observasional dan berbasis konten semata.

**Kelebihan:**
- Tidak memerlukan data pengguna.
- Dapat memberikan rekomendasi meskipun pengguna baru (cold start user).

**Kekurangan:**
- Tidak memperhitungkan opini pengguna lain.
- Bergantung sepenuhnya pada kualitas deskripsi buku.

---

## Kesimpulan

Sistem rekomendasi buku berbasis Content-Based Filtering ini mampu memberikan rekomendasi buku yang relevan dengan preferensi pengguna berdasarkan kesamaan deskripsi. Sistem ini sangat cocok digunakan untuk aplikasi perpustakaan atau platform digital yang ingin menyediakan rekomendasi otomatis tanpa mengandalkan data rating pengguna.

Diharapkan, penerapan sistem ini dapat membantu meningkatkan minat baca, terutama di kalangan pelajar dan generasi muda, dengan cara memberikan rekomendasi buku yang sesuai dan menarik [1].

---

## Daftar Pustaka

1. S. D. Lubis, H. H. M. Sitepu, dan D. R. Irawan, "Sistem Rekomendasi Buku Menggunakan Content-Based Filtering Pada Aplikasi Perpustakaan," *Jurnal Coscitech*, vol. 10, no. 2, 2023. [https://ejurnal.umri.ac.id/index.php/coscitech/article/view/5131/2462](https://ejurnal.umri.ac.id/index.php/coscitech/article/view/5131/2462)

2. Zając, Z. "goodbooks-10k." Kaggle. [https://www.kaggle.com/datasets/zygmunt/goodbooks-10k](https://www.kaggle.com/datasets/zygmunt/goodbooks-10k)
