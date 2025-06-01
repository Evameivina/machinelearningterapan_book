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

Dataset ini terdiri dari beberapa file utama sebagai berikut:

### 1. books.csv  
File ini berisi metadata tentang buku, termasuk ID unik, judul, penulis, tahun terbit, rating rata-rata, bahasa, dan informasi lain terkait buku.

- **Jumlah baris**: 10.000 buku  
- **Jumlah kolom**: 23 kolom  
- **Contoh kolom penting**:
  - `book_id` (int): ID unik buku  
  - `title` (object): Judul buku  
  - `authors` (object): Nama penulis  
  - `average_rating` (float): Rating rata-rata buku  
  - `original_publication_year` (float): Tahun pertama kali terbit  
  - `language_code` (object): Kode bahasa buku  
  - `ratings_count` (int): Jumlah total rating yang diberikan  
  - `ratings_1` sampai `ratings_5` (int): Jumlah rating masing-masing skor 1 sampai 5  
  - `image_url`, `small_image_url` (object): URL gambar sampul buku

#### Data Quality dan Kondisi Dataset:

- **Missing Values**:  
  - Kolom seperti `isbn`, `original_title`, dan `language_code` memiliki nilai kosong (NaN) sekitar 5-10% dari total data.  
  - Nilai kosong ini perlu diperhatikan dalam preprocessing agar tidak mempengaruhi analisis lebih lanjut.  

- **Duplikasi**:  
  - Ditemukan beberapa duplikasi berdasarkan `book_id`, namun jumlahnya sangat kecil (<0.1%). Duplikasi ini akan dibersihkan sebelum analisis.  

- **Outlier**:  
  - Kolom `average_rating` memiliki rentang nilai 1–5 sesuai skala, namun terdapat buku dengan jumlah rating (`ratings_count`) sangat rendah (<5), yang membuat rating kurang representatif.  
  - Terdapat nilai tahun terbit `original_publication_year` yang tidak valid (misal 0 atau negatif) sekitar 0.5% data, yang perlu dibersihkan atau dikoreksi.

### 2. ratings.csv  
File ini berisi data rating yang diberikan oleh pengguna untuk masing-masing buku.

- **Jumlah baris**: 981.756 rating  
- **Jumlah kolom**: 3 (`book_id`, `user_id`, `rating`)  
- **Keterangan kolom**:  
  - `book_id` (int): ID buku  
  - `user_id` (int): ID pengguna  
  - `rating` (int): Rating yang diberikan (1–5)

#### Data Quality dan Kondisi Dataset:

- **Missing Values**:  
  - Tidak terdapat missing values pada kolom utama.  

- **Duplikasi**:  
  - Tidak ditemukan duplikasi persis pada kombinasi `book_id` dan `user_id`.  

- **Distribusi Rating**:  
  - Rating didominasi oleh skor 4 dan 5, yang merupakan indikasi bias rating positif dari pengguna.  

### 3. book_tags.csv dan tags.csv  
File `book_tags.csv` menghubungkan buku dengan tag yang diberikan pengguna, sedangkan `tags.csv` berisi daftar tag unik.

- **book_tags.csv**:
  - Baris: 1.149.780  
  - Kolom: `goodreads_book_id`, `tag_id`, `count` (berapa kali tag tersebut diberikan pada buku)  

- **tags.csv**:  
  - Baris: 34.252 (jumlah tag unik)  
  - Kolom: `tag_id`, `tag_name`  

#### Data Quality dan Kondisi Dataset:

- **Missing Values**:  
  - Tidak ditemukan nilai kosong.  

- **Duplikasi**:  
  - Beberapa tag sama muncul pada buku yang berbeda, ini normal karena satu tag dapat dipakai di banyak buku.  

- **Distribusi Tag**:  
  - Sebagian besar tag memiliki frekuensi rendah, hanya beberapa tag populer dengan frekuensi tinggi.  

### 4. to_read.csv  
File ini berisi daftar buku yang ditandai pengguna untuk dibaca di masa depan.

- **Jumlah baris**: 912.705 data  
- **Kolom**: `book_id`, `user_id`  

#### Data Quality dan Kondisi Dataset:

- **Missing Values**:  
  - Tidak ditemukan missing values.  

- **Duplikasi**:  
  - Ada kemungkinan pengguna menandai buku yang sama lebih dari sekali, perlu dicek dan dibersihkan jika ada.
    
### 5. sample_book.xml  
File ini merupakan file tambahan yang tidak digunakan dalam analisis ini.


## Data Preparation

Pada tahap ini, dilakukan beberapa teknik persiapan data (data preparation) untuk memastikan kualitas data yang baik dan konsisten sebelum melanjutkan ke tahap modeling. Teknik data preparation yang dilakukan, antara lain:

1. **Pemeriksaan Struktur Data**
   - Melihat ukuran dataset `ratings` dan `books` untuk memahami skala data:
     ```python
     print(ratings.shape)  # (981756, 3)
     print(books.shape)    # (10000, 23)
     ```
   - Melihat preview data dan statistik deskriptif untuk mengevaluasi struktur dan distribusi data.

2. **Cek Jumlah Unik**
   - Menghitung jumlah pengguna (`user_id`) dan jumlah buku (`book_id`) unik:
     ```python
     print('Jumlah user_id (pengguna): ', len(ratings.user_id.unique()))
     print('Jumlah book_id (buku): ', len(ratings.book_id.unique()))
     ```

3. **Gabungan Data Buku**
   - Menggabungkan seluruh `book_id` dari beberapa sumber dataset (`books`, `ratings`, `book_tags`, `to_read`) untuk mendapatkan daftar buku unik:
     ```python
     books_all = np.concatenate((
         books.book_id.unique(),
         ratings.book_id.unique(),
         book_tags.goodreads_book_id.unique(),
         to_read.book_id.unique()
     ))
     books_all = np.sort(np.unique(books_all))
     ```

4. **Gabungan Data Pengguna**
   - Menggabungkan seluruh `user_id` unik dari dataset `ratings`:
     ```python
     user_all = np.sort(np.unique(ratings.user_id.unique()))
     ```

5. **Penanganan Missing Values**
   - Mengisi nilai kosong pada kolom `language_code` dengan modus (nilai yang paling sering muncul):
     ```python
     mode_language = books['language_code'].mode()[0]
     books['language_code'] = books['language_code'].fillna(mode_language)
     ```
   - Mengisi nilai kosong pada kolom `year` dengan median tahun:
     ```python
     median_year = books['year'].median()
     books['year'] = books['year'].fillna(median_year)
     ```

6. **TF-IDF Vectorization (Content-Based Filtering)**
   - Mengubah data teks (misalnya deskripsi buku) menjadi representasi numerik dengan TF-IDF untuk mengukur kemiripan konten antar buku.
   - Teknik ini memungkinkan sistem rekomendasi berbasis konten menghasilkan rekomendasi yang relevan berdasarkan fitur teks.

7. **Label Encoding (Collaborative Filtering)**
   - Mengubah variabel kategorikal `user_id` dan `book_id` menjadi nilai numerik menggunakan Label Encoding agar sesuai dengan algoritma Collaborative Filtering:
     ```python
     from sklearn.preprocessing import LabelEncoder

     user_encoder = LabelEncoder()
     book_encoder = LabelEncoder()

     ratings['user'] = user_encoder.fit_transform(ratings['user_id'])
     ratings['book'] = book_encoder.fit_transform(ratings['book_id'])
     ```

8. **Pembuatan Label Biner dari Rating**
   - Mengonversi rating asli ke label biner untuk keperluan klasifikasi:
     - Rating ≥ 4 diberi label 1 (positif)
     - Rating < 4 diberi label 0 (negatif)
     ```python
     ratings['rating_binary'] = ratings['rating'].apply(lambda x: 1 if x >= 4 else 0)
     ```

9. **Pembagian Data Menjadi Data Latih dan Validasi**
   - Membagi data rating menjadi data latih dan validasi dengan rasio 80:20 menggunakan `train_test_split`:
     ```python
     from sklearn.model_selection import train_test_split

     train, val = train_test_split(ratings, test_size=0.2, random_state=42)
     ```

### Alasan Tahapan Data Preparation

- **Pemeriksaan struktur data** penting untuk mengetahui apakah terdapat data yang hilang, outlier, atau format tidak konsisten.
- **Penggabungan data unik** diperlukan untuk membangun matriks interaksi pengguna-buku yang komprehensif.
- **Penanganan missing values** mencegah error dan bias pada model akibat data kosong.
- **TF-IDF vectorization** memungkinkan analisis kemiripan buku berdasarkan isi teks.
- **Label encoding** diperlukan agar data kategorikal dapat diproses algoritma Collaborative Filtering.
- **Pembuatan label biner** mempermudah klasifikasi preferensi pengguna.
- **Pembagian data latih dan validasi** berguna untuk evaluasi performa model yang lebih objektif.


## Modeling

Tahapan modeling pada proyek ini menggunakan pendekatan **Collaborative Filtering berbasis Neural Network** untuk membangun sistem rekomendasi buku. Algoritma yang digunakan adalah arsitektur custom berbasis **Embedding Layers dan Dot Product** menggunakan **TensorFlow Keras**.

### Tahapan Modeling

1. **Encoding Identitas**
   - Data ID pengguna dan buku diubah ke bentuk numerik menggunakan `LabelEncoder`, agar dapat digunakan dalam embedding.

2. **Pembuatan Label**
   - Rating dikonversi menjadi label biner:
     - `1` jika rating ≥ 4 (suka)
     - `0` jika rating < 4 (tidak suka)

3. **Splitting Data**
   - Data dibagi menjadi 80% data latih dan 20% data validasi menggunakan `train_test_split` dengan stratifikasi.

4. **Arsitektur Model**
   - Model `RecommenderNet` terdiri dari:
     - **Embedding Layer** untuk user dan book
     - **Bias Embedding** untuk user dan book
     - **Dot Product** sebagai skor interaksi
     - **Dropout Layer** untuk regularisasi
     - **Output Layer**: aktivasi sigmoid

5. **Parameter dan Hyperparameter**
   - Embedding size: 64
   - Loss function: Binary Crossentropy
   - Optimizer: Adam (`learning_rate = 5e-5`)
   - Metrics: Accuracy, AUC, RMSE
   - Callbacks:
     - `EarlyStopping` (monitor: val_auc, patience: 5)
     - `ReduceLROnPlateau` (monitor: val_auc, factor: 0.5, patience: 3)

6. **Training**
   - Model dilatih selama maksimum 40 epoch dengan batch size 256.

### Kelebihan Model

- Mampu menangkap hubungan laten antar user dan buku.
- Flexible untuk dilakukan improvement (misalnya tuning embedding size, dropout, dsb).
- Tidak membutuhkan fitur konten buku secara eksplisit.

### Kekurangan Model

- Tidak bekerja optimal pada user/buku baru (cold-start problem).
- Membutuhkan cukup banyak data interaksi untuk hasil optimal.


## Evaluation

## Modeling

Pada proyek ini, sistem rekomendasi buku dikembangkan menggunakan dua pendekatan utama, yaitu **Content-Based Filtering** dan **Collaborative Filtering berbasis Neural Network**. 

### 1. Content-Based Filtering

Pendekatan ini menggunakan fitur konten buku (judul dan penulis) untuk merekomendasikan buku yang mirip secara teks.

- **TF-IDF Vectorizer** digunakan untuk mengubah data teks (kombinasi judul dan penulis buku) menjadi representasi numerik.
- **Cosine Similarity** digunakan untuk menghitung kemiripan antar buku berdasarkan vektor TF-IDF tersebut.

#### Tahapan:

1. Data judul dan penulis buku digabungkan menjadi satu kolom `combined`.
2. TF-IDF Vectorizer dilatih pada data `combined` dengan mengabaikan stop words bahasa Inggris.
3. Dihasilkan matriks TF-IDF untuk setiap buku.
4. Matriks cosine similarity dihitung dari matriks TF-IDF tersebut, menghasilkan matriks kesamaan antar buku.
5. Sistem merekomendasikan buku dengan skor similarity tertinggi untuk buku yang diberikan.

#### Contoh output rekomendasi:

Misal, rekomendasi Top-10 buku mirip untuk buku *"The Hunger Games (The Hunger Games, #1)"* adalah:

| Title                                      | Author           | Year   | Avg Rating |
|--------------------------------------------|------------------|--------|------------|
| Catching Fire (The Hunger Games, #2)       | Suzanne Collins  | 2009   | 4.30       |
| Mockingjay (The Hunger Games, #3)           | Suzanne Collins  | 2010   | 4.03       |
| The Hunger Games Trilogy Boxset             | Suzanne Collins  | 2010   | 4.49       |
| The Hunger Games: Official Illustrated Movie| Kate Egan        | 2012   | 4.51       |
| The Hunger Games Tribute Guide               | Emily Seife      | 2012   | 4.40       |
| Hunger (Gone, #2)                            | Michael Grant    | 2009   | 4.02       |
| A Hunger Like No Other (Immortals After Dark)| Kresley Cole    | 2006   | 4.21       |
| Nemesis Games (The Expanse, #5)              | James S.A. Corey | 2015   | 4.37       |
| The Quillan Games (Pendragon, #7)            | D.J. MacHale     | 2005   | 4.19       |
| The World of the Hunger Games (Hunger Games) | Kate Egan       | 2012   | 4.48       |

### 2. Collaborative Filtering Berbasis Neural Network

Pendekatan ini menggunakan interaksi pengguna dengan buku berupa rating untuk memprediksi preferensi dan merekomendasikan buku.

#### Tahapan Modeling:

1. **Encoding Identitas**
   - ID pengguna dan buku dikonversi ke bentuk numerik menggunakan `LabelEncoder` untuk embedding.

2. **Pembuatan Label**
   - Rating dikonversi menjadi label biner:
     - `1` jika rating ≥ 4 (menandakan suka)
     - `0` jika rating < 4 (tidak suka)

3. **Splitting Data**
   - Data dibagi menjadi 80% latih dan 20% validasi menggunakan `train_test_split` dengan stratifikasi.

4. **Arsitektur Model**
   - Model `RecommenderNet` terdiri dari:
     - Embedding Layer untuk pengguna dan buku
     - Bias embedding untuk pengguna dan buku
     - Dot Product antara embedding sebagai skor interaksi
     - Dropout Layer untuk regularisasi
     - Output layer dengan aktivasi sigmoid (prediksi probabilitas suka)

5. **Parameter dan Hyperparameter**
   - Embedding size: 64
   - Loss function: Binary Crossentropy
   - Optimizer: Adam (learning rate 5e-5)
   - Metrics: Accuracy, AUC, RMSE
   - Callbacks: EarlyStopping dan ReduceLROnPlateau

6. **Training**
   - Model dilatih maksimum 40 epoch dengan batch size 256.

#### Kelebihan Model:

- Menangkap hubungan laten antar pengguna dan buku.
- Mudah di-tune dan ditingkatkan.
- Tidak membutuhkan data fitur konten buku.

#### Kekurangan Model:

- Kurang efektif untuk pengguna atau buku baru (cold-start).
- Membutuhkan banyak data interaksi untuk hasil optimal.

### Output Top-N Rekomendasi Collaborative Filtering

Setelah model terlatih, rekomendasi buku untuk pengguna tertentu diberikan berdasarkan skor prediksi tertinggi.

Misal, untuk pengguna dengan ID tertentu, Top-5 rekomendasi buku yang diprediksi disukai adalah:

| Book ID  | Title                            | Predicted Score |
|----------|---------------------------------|-----------------|
| 2251306  | Brain Rules: 12 Principles...    | 0.87            |
| 5989573  | Scott Pilgrim, Volume 5          | 0.85            |
| 40024    | The Alienist                    | 0.83            |
| 234184   | Stone of Tears                  | 0.82            |
| 4800764  | Sweet Persuasion                | 0.80            |


### Hasil Evaluasi

- **Akurasi Validasi Terbaik**: ~70.6%
- **AUC Validasi Terbaik**: ~0.7277
- **Confusion Matrix & Classification Report** menunjukkan performa cukup seimbang antara prediksi suka dan tidak suka.
- **ROC Curve** menghasilkan AUC 0.7277, yang menunjukkan kemampuan klasifikasi model cukup baik meski masih bisa ditingkatkan.

### Penutup

Model ini dipilih sebagai solusi terbaik karena:

- Memiliki hasil evaluasi yang stabil dan membaik pada tiap epoch awal.
- AUC dan akurasi meningkat secara bertahap tanpa overfitting berkat penggunaan early stopping dan regulasi.
- Model cukup sederhana untuk di-deploy dan dikembangkan lebih lanjut (misalnya dengan peningkatan embedding atau hybrid filtering).

