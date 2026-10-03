# AI Model Experiment & Evaluation — Sentiment Analysis

## Problem Statement

**Objective:**
Objective adalah mengotomatisasi klasifikasi sentiment review dari pelanggan dihalaman produk, supaya sentimen bisa di ketahui tanpa proses manual. ini sangat membantu untuk data jumlah review/ulasan yang sangat banyak

**Target / Label:**
klasifikasi hanya di bagi menjadi positif dan negatif ( Binary classification) , tidak ada ulasan netral 

**Batasan dan Asumsi:**
-Tidak ada kelas netral, ulasan ambigu dipaksa masuk salah satu
-Hanya 40 teks unik dari 200 baris
--Dataset kemungkinan sintetis, bukan ulasan pelanggan asli
Teks berbahasa Indonesia


## Dataset

-200 baris, 4 kolom. review_id, product_name, review_text, sentiment
-input = review_text
-target = sentiment 
-Hanya ada 40 teks unik, dan tiap kalimat muncul rata rata 5 kali 
-distribusi label 110 positif : 90 negatif.Distribusi label cukup seimbang (110:90), sehingga accuracy masih layak dipakai sebagai metrik. Namun karena ada kebocoran data antara training dan test set, nilai accuracy yang dihasilkan tetap tidak mencerminkan performa sebenarnya


## Cara Menjalankan

1. Clone repository ini
2. Buat virtual environment:
   (Windows: `venv\Scripts\activate`)
3. Install dependencies:
4. Buat file `.env` di folder project, isi dengan:GEMINI_API_KEY=your_api_key_here
5. Jalankan `notebook/starter_notebook.ipynb`