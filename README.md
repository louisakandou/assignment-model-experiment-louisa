# AI Model Experiment & Evaluation — Sentiment Analysis

## Problem Statement

**Objective:**
Objective adalah mengotomatisasi klasifikasi sentiment review dari pelanggan dihalaman produk, supaya sentimen bisa di ketahui tanpa proses manual. ini sangat membantu untuk data jumlah review/ulasan yang sangat banyak

**Target / Label:**
klasifikasi hanya di bagi menjadi positif dan negatif ( Binary classification) , tidak ada ulasan netral 

**Batasan dan Asumsi:**
- Tidak ada kelas netral, ulasan ambigu dipaksa masuk salah satu
- Hanya 40 teks unik dari 200 baris
- Dataset kemungkinan sintetis, bukan ulasan pelanggan asli
- Teks berbahasa Indonesia

## Dataset

- 200 baris, 4 kolom: review_id, product_name, review_text, sentiment
- Input = review_text
- Target = sentiment
- Hanya ada 40 teks unik, tiap kalimat muncul rata-rata 5 kali
- Distribusi label 110 positif : 90 negatif

## Cara Menjalankan

1. Clone repository ini

2. Buat virtual environment:

```
python3 -m venv venv
source venv/bin/activate
```

Windows: `venv\Scripts\activate`

3. Install dependencies:

```
pip install -r requirements.txt
```

4. Buat file `.env` di folder project, isi dengan:

```
GEMINI_API_KEY=your_api_key_here
```

5. Jalankan `notebook/starter_notebook.ipynb`

## Ringkasan Eksperimen

**Pendekatan 1 — Model Klasik (Scikit-learn).** Teks ulasan diubah menjadi
representasi numerik menggunakan TF-IDF, kemudian model Logistic Regression
dilatih pada 160 ulasan training set dan diuji pada 40 ulasan test set.

**Pendekatan 2 — LLM API (Gemini).** Klasifikasi dilakukan melalui Gemini API
dengan prompt zero-shot tanpa proses training. Output model dinormalisasi
(`.strip().lower()`) agar dapat dibandingkan dengan label dataset. Karena tier
gratis dibatasi 20 permintaan per hari, hanya 14 ulasan yang berhasil
diklasifikasikan.

**Menjaga perbandingan tetap adil.** Kedua pendekatan dievaluasi pada 14 ulasan
yang sama persis dari test set yang sama.

## Hasil Evaluasi

| Metrik | Model Klasik | Gemini API |
|---|---|---|
| Accuracy | 1.00 | 1.00 |
| Precision (negatif) | 1.00 | 1.00 |
| Recall (negatif) | 1.00 | 1.00 |
| F1-score (negatif) | 1.00 | 1.00 |
| Precision (positif) | 1.00 | 1.00 |
| Recall (positif) | 1.00 | 1.00 |
| F1-score (positif) | 1.00 | 1.00 |
| Jumlah data uji | 14 | 14 |
| Waktu proses | < 1 detik | ± 3 menit |
| Perlu training | Ya | Tidak |
| Butuh koneksi internet | Tidak | Ya |
| Biaya per prediksi | Nol setelah training | Per token, terus-menerus |

## Analisa Trade Off dan LImitation 

### Performa

Kedua pendekatan menghasilkan metrik identik (accuracy 1.00), sehingga tidak ada
yang unggul secara performa. Namun penyebabnya berbeda:

- **Model klasik** mencapai 1.00 karena sebagian besar kalimat di test set sudah
  muncul di training set, sehingga model mengingat alih-alih memahami.
- **Gemini** mencapai 1.00 tanpa melihat data latih sama sekali, tetapi dibantu
  oleh dataset yang kalimatnya straightforward dan tidak mengandung ambiguitas.

Dengan kata lain, dataset ini terlalu mudah sehingga metrik evaluasi kehilangan
daya pembeda antara kedua pendekatan.

### Trade-off

**Effort implementasi.** Model klasik memerlukan beberapa tahap: split data,
TF-IDF, dan training. Gemini tidak memerlukan training sama sekali — cukup
merancang prompt — sehingga jauh lebih cepat disiapkan.

**Kecepatan.** Model klasik memproses seluruh 40 ulasan dalam hitungan milidetik.
Gemini membutuhkan sekitar 13 detik per ulasan karena batas 5 permintaan per menit.

**Biaya.** Model klasik hanya memerlukan satu kali training, setelah itu prediksi
tidak berbiaya. Gemini menagih per token pada setiap prediksi, sehingga biayanya
terus bertambah seiring jumlah ulasan. Untuk implementasi nyata diperlukan tier
berbayar, karena tier gratis memiliki batas kuota.

### Limitation Model Klasik

- **Data leakage** — 25 dari 26 kalimat unik di test set sudah ada di training set
- Hanya mengenali kata yang pernah dilihat saat training; buta terhadap kosakata
  baru, bahasa gaul, dan sarkasme
- Perlu dilatih ulang secara berkala ketika bahasa pelanggan berubah

### Limitation Gemini API

- Kuota tier gratis: 5 permintaan per menit dan 20 permintaan per hari
- Bergantung pada layanan pihak ketiga — model `gemini-2.0-flash` dihentikan Google
  di tengah pengerjaan, dan server mengembalikan error 503 berulang kali
- Percobaan yang gagal tetap terhitung


## Rekomendasi Technical Approach 

**Rekomendasi: Model klasik (Scikit-learn), dengan syarat perbaikan dataset.**

**Mengapa metrik tidak dapat dijadikan dasar.** Kedua pendekatan menghasilkan
accuracy 1.00 pada test set yang sama, sehingga metrik evaluasi tidak dapat
memisahkan keduanya. Keputusan harus diambil berdasarkan pertimbangan operasional.

**Dasar pertimbangan:**

1. **Biaya jangka panjang.** Model klasik hanya memerlukan satu kali training,
   setelah itu setiap prediksi tidak berbiaya. Gemini menagih per token pada setiap
   prediksi, sehingga biaya terus bertambah seiring volume ulasan.

2. **Kecepatan.** Untuk fitur yang tampil di halaman produk, model klasik memproses
   seluruh test set dalam milidetik, sementara Gemini membutuhkan sekitar 13 detik
   per ulasan pada tier gratis.

3. **Kemandirian sistem.** Selama pengerjaan, model `gemini-2.0-flash` dihentikan
   oleh Google dan server mengembalikan error 503 berulang kali. Model klasik
   berjalan di infrastruktur sendiri dan tidak bergantung pada keputusan penyedia
   layanan.

**Kelemahan dari pilihan ini.** Model klasik hanya mengenali kata yang pernah
dilihatnya saat training, sehingga lemah menghadapi kosakata baru, bahasa gaul,
dan sarkasme. Pada aspek ini Gemini jelas lebih unggul.

**Syarat yang harus dipenuhi:**

- Dataset diperluas dengan ulasan pelanggan asli yang lebih beragam
- Tidak ada duplikasi kalimat antara training set dan test set
- Model dilatih ulang secara berkala mengikuti perkembangan bahasa pelanggan
- Dilakukan evaluasi ulang pada dataset yang lebih menantang, karena hasil
  eksperimen ini belum mencerminkan performa sebenarnya