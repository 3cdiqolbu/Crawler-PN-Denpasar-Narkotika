# 🏛️ Sistem Temu Kembali Informasi Putusan Pengadilan: Perkara Narkotika PN Denpasar (2023–2025)

**Mata Kuliah:** Temu Kembali Informasi (*Information Retrieval*) — Kelas D  
**Program Studi:** Informatika, Fakultas Teknik, Universitas Muhammadiyah Malang  

**Anggota Kelompok:**
1. **Tricahyo Diqolbu** (`202310370311309`)
2. **M. Rayhan Shaffarukhan Alkamaly** (`202310370311346`)

---

## 📌 Deskripsi Proyek & Dataset
Repositori ini memuat korpus dokumen putusan pengadilan, *pipeline* akuisisi data (*Hybrid Web Crawler*), sistem ekstraksi informasi berbasis *Regular Expression (Regex)*, serta mesin pencari dokumen hukum (*Legal Information Retrieval*) untuk perkara **Pidana Khusus Narkotika dan Psikotropika** pada **Pengadilan Negeri Denpasar** dalam rentang tahun **2023 hingga 2025**.

Seluruh dokumen putusan yang dihimpun telah melalui proses penyaringan dan validasi untuk memastikan:
- Berada pada yurisdiksi **Pengadilan Negeri Denpasar (`PN Dps`)**.
- Termasuk dalam klasifikasi **Pidana Khusus — Narkotika dan Psikotropika**.
- Diputus dalam rentang tahun **2023–2025**.
- **Belum Berkekuatan Hukum Tetap (Non-BHT)** (mengecualikan putusan berstatus *Inkracht*, Kasasi, Peninjauan Kembali, atau Grasi).

---

## 📂 Struktur Direktori Repositori

```text
Dataset-Narkotika_309_346/
│
├── Dataset/
│   ├── Narkotika.zip                             # Basis data dokumen PDF putusan asli Mahkamah Agung (Auto-Fallback)
│   └── Folder_TXT_Putusan.zip                    # Arsip dokumen teks (.txt) bersih hasil konversi & noise removal
│
├── Overview.xlsx                                 # Tabel metadata utama 50 putusan tahun 2023-2025 (Sub-CPMK 2 / Tugas 2)
├── Overview_100.xlsx                             # Tabel perluasan kuota target putusan (Merge Basis Data + Crawler)
├── Hasil_Ekstraksi_Crawler_PN_Denpasar.csv       # Tabel 15 atribut hasil ekstraksi Regex presisi tinggi (Sub-CPMK 3 / Tugas 3)
├── Inverted_Index_Putusan_PN_Denpasar.csv        # Tabel Dictionary, DF, CF, IDF & Postings List (Sub-CPMK 4 / Tugas 4)
├── Inverted_Index_Putusan_PN_Denpasar.json       # Struktur Inverted Index dalam format JSON (Sub-CPMK 4 / Tugas 4)
│
├── crawler_putusan_dpsr.ipynb                    # Notebook utama (End-to-End Pipeline Tugas 3 & Tugas 4)
├── .gitignore                                    # Konfigurasi pengabaian folder kerja sementara & cache
└── README.md                                     # Dokumentasi utama repositori
```

---

## ⚙️ Metodologi & Arsitektur Sistem

### 1. Akuisisi Data Hibrida (*Hybrid Web Crawler & Auto-Merge* — Sub-CPMK 3)
Sistem akuisisi dirancang menggunakan pendekatan **Hybrid Fault-Tolerant Crawler** terhadap portal [Direktori Putusan Mahkamah Agung RI](https://putusan3.mahkamahagung.go.id/):
- **Kepatuhan Etika Crawler (`robots.txt` & *Politeness*)**: Memeriksa izin akses direktori melalui `urllib.robotparser` pada `/robots.txt` dan menerapkan *implicit politeness delay* sebesar **2 detik** antar-permintaan HTTP.
- **Penanganan *Geo-Blocking* (`HTTP 403`)**: Mengintegrasikan *TLS Fingerprint Impersonation* (`curl_cffi` Chrome 120), koneksi langsung *Local Runtime* (IP Indonesia), serta *Web Archive Mirror Fallback* sehingga *notebook* dapat dijalankan baik di komputer lokal maupun di *Cloud* Google Colaboratory tanpa *error*.
- **Prioritas Basis Data & Deduplikasi**: Memprioritaskan ekstraksi arsip terverifikasi pada `Dataset/Narkotika.zip`, menambal kekurangan kuota secara otomatis melalui *live crawling*, dan mengeliminasi duplikasi dokumen berdasarkan kunci unik `No Putusan`.

### 2. Pembersihan *Noise* & Ekstraksi *Regex* 15 Atribut (Sub-CPMK 3)
Dokumen PDF dikonversi menjadi teks utuh menggunakan `PyMuPDF` (`pymupdf`), dibersihkan dari serpihan *watermark* Mahkamah Agung serta karakter Unicode rusak, lalu diekstraksi menggunakan pola *Regex* berlapis menjadi **15 atribut terstruktur**:
1. `No` (Nomor Urut)
2. `No Putusan` (Contoh: `108/Pid.Sus/2023/PN Dps`)
3. `Lembaga Peradilan` (`PN Denpasar`)
4. `Barang Bukti` (Daftar lengkap barang bukti narkotika & non-narkotika)
5. `Amar Putusan` (Diambil secara presisi dari blok `MENGADILI:` terakhir)
6. `Nama Terdakwa`
7. `Umur / Tgl Lahir`
8. `Jenis Kelamin`
9. `Pekerjaan Terdakwa`
10. `Tuntutan JPU`
11. `Pasal Terbukti` (Pasal UU RI No. 35 Tahun 2009 tentang Narkotika)
12. `Tanggal Putusan`
13. `Hakim Ketua` (Mendukung ekstraksi dari kalimat penutup maupun blok tanda tangan)
14. `Nama File PDF`
15. `Sumber Akuisisi` (`Basis Data Manual` / `Live Web Crawler`)

### 3. *Text Preprocessing* & Normalisasi Domain Hukum (Sub-CPMK 4)
Seluruh teks putusan (*full-text*) diproses melalui 5 tahapan terstandar:
1. **Case Folding**: Penyeragaman seluruh karakter menjadi huruf kecil.
2. **Tokenization & Alphabet Filtering**: Pemisahan token alfabet ($\ge 3$ karakter).
3. **Equivalence Classing (Normalisasi Istilah Narkotika)**: Penyeragaman ejaan tidak baku dan sinonim domain narkotika/hukum (`shabu`/`sabu-sabu` $\rightarrow$ `sabu`, `metamfetamina`/`methamphetamine` $\rightarrow$ `metamfetamin`, `mariyuana`/`kanabis` $\rightarrow$ `ganja`, `extacy`/`ineks` $\rightarrow$ `ekstasi`, `subsidair` $\rightarrow$ `subsider`).
4. **Stopword Removal**: Penghapusan kata umum bahasa Indonesia (`PySastrawi`) yang diperkaya dengan *custom legal stopwords*.
5. **Cached Stemming**: Pemulihan kata berimbuhan ke bentuk dasar menggunakan algoritma *Nazief & Adriani* (`PySastrawi`) dengan optimasi *in-memory dictionary caching*.

### 4. *Inverted Index* & Model Perankingan TKI (Sub-CPMK 4)
Sistem membangun **Inverted Index** yang memetakan setiap *term* di dalam *Vocabulary* ke *Document Frequency* ($df_t$), *Collection Frequency* ($cf_t$), bobot $IDF_t$, serta *Postings List* (`docID:tf`). Pencarian dan perankingan dokumen diimplementasikan melalui dua model utama:

- **Vector Space Model (Sublinear TF-IDF + Cosine Similarity)**:
  $$w_{t,d} = \left(1 + \log_{10} tf_{t,d}\right) \times \log_{10}\left(\frac{N}{df_t}\right)$$
  dengan normalisasi panjang vektor Euclidean (*L2 Norm*) untuk menghitung skor kemiripan kosinus antara vektor kueri dan dokumen.

- **Probabilistic Information Retrieval (Okapi BM25)**:
  $$RSV_{BM25}(q, d) = \sum_{t \in q} \log_{10}\left(1 + \frac{N - df_t + 0.5}{df_t + 0.5}\right) \cdot \frac{(k_1 + 1) tf_{t,d}}{k_1 \left((1 - b) + b \times \frac{dl}{avgdl}\right) + tf_{t,d}}$$
  menggunakan parameter penjenuhan frekuensi $k_1 = 1.5$ dan normalisasi panjang dokumen $b = 0.75$.

---

## 🚀 Cara Menjalankan Proyek

### Opsi 1: Eksekusi Langsung di Google Colaboratory (Cloud)
1. Buka berkas `crawler_putusan_dpsr.ipynb` di Google Colaboratory.
2. Tekan menu **Runtime $\rightarrow$ Run all** (`Ctrl + F9`).
3. *Notebook* akan secara otomatis mengunduh `Dataset/Narkotika.zip` dari repositori GitHub ini melalui variabel `GITHUB_RAW_ZIP_URL` apabila berkas belum tersedia di penyimpanan lokal Colab.

### Opsi 2: Eksekusi Menggunakan *Local Runtime* (VS Code / Jupyter Lokal — IP Indonesia)
1. Kloning repositori ini ke komputer lokal:
   ```bash
   git clone https://github.com/3cdiqolbu/Dataset-Narkotika_309_346.git
   cd Dataset-Narkotika_309_346
   ```
2. Instal seluruh dependensi pustaka Python:
   ```bash
   pip install pymupdf beautifulsoup4 requests pandas openpyxl urllib3 cloudscraper curl_cffi PySastrawi scikit-learn
   ```
3. Jalankan `crawler_putusan_dpsr.ipynb` melalui VS Code atau hubungkan ke Google Colaboratory melalui *Local Runtime* Jupyter Server.

---

## ⚖️ Lisensi & Ketentuan Penggunaan Data
Seluruh dokumen putusan pengadilan di dalam repositori ini bersumber dari publikasi resmi **Direktori Putusan Mahkamah Agung Republik Indonesia** berdasarkan Undang-Undang Nomor 14 Tahun 2008 tentang Keterbukaan Informasi Publik (UU KIP). Korpus dan kode sumber ini disusun semata-mata untuk kepentingan akademik, riset, dan pembelajaran pada mata kuliah Temu Kembali Informasi.