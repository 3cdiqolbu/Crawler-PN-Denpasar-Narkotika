# 🏛️ Sistem Temu Kembali Informasi Putusan Pengadilan: Perkara Narkotika PN Denpasar (2023–2025)

**Mata Kuliah:** Temu Kembali Informasi (*Information Retrieval*) — Kelas D  
**Program Studi:** Informatika, Fakultas Teknik, Universitas Muhammadiyah Malang  

**Anggota Kelompok:**
1. **Tricahyo Diqolbu** (`202310370311309`)
2. **M. Rayhan Shaffarukhan Alkamaly** (`202310370311346`)

---

## 📌 Deskripsi Proyek & Dataset

Repositori ini memuat korpus dokumen putusan pengadilan, *pipeline* akuisisi data (*Hybrid Web Crawler*), sistem ekstraksi informasi berbasis *Regular Expression (Regex)* yang diperkaya dengan *HTML Table Metadata Fallback*, serta mesin pencari dokumen hukum (*Legal Information Retrieval*) untuk perkara **Pidana Khusus Narkotika dan Psikotropika** pada **Pengadilan Negeri Denpasar** dalam rentang tahun **2023 hingga 2025**.

Seluruh dokumen putusan yang dihimpun telah melalui proses penyaringan dan validasi otomatis untuk memastikan:
* Berada pada yurisdiksi **Pengadilan Negeri Denpasar (`PN Dps`)**.
* Termasuk dalam klasifikasi **Pidana Khusus — Narkotika dan Psikotropika**.
* Diputus dalam rentang tahun **2023–2025** (untuk tabel utama `Overview.xlsx`).
* **Belum Berkekuatan Hukum Tetap (Non-BHT)** (mengecualikan putusan berstatus *Inkracht*, Kasasi, Peninjauan Kembali, atau Grasi pada tabel metadata halaman detail).

---

## 📂 Struktur Direktori Repositori

```text
Crawler-PN-Denpasar-Narkotika/
│
├── Dataset/
│   ├── Narkotika.zip                             # Basis data 64 dokumen PDF putusan asli Mahkamah Agung (Prioritas 1)
│   └── Folder_TXT_Putusan.zip                    # Arsip 70 dokumen teks (.txt) bersih hasil konversi & noise removal
│
├── Overview/
│   ├── Overview.xlsx                                 # Tabel metadata utama putusan tahun 2023-2025 (Sub-CPMK 2 / Tugas 2)
│
├── Hasil_Ekstraksi_Crawler_PN_Denpasar.csv       # Tabel 15 atribut hasil ekstraksi 70 dokumen unik (Sub-CPMK 3 / Tugas 3)
├── Log_URL_Crawler_MA.csv                        # Log penelusuran URL kandidat & metadata HTML dari crawler
├── Inverted_Index_Putusan_PN_Denpasar.csv        # Tabel Dictionary (5.119 unique terms), DF, CF, IDF & Postings List (Tugas 4)
├── Inverted_Index_Putusan_PN_Denpasar.json       # Struktur Inverted Index dalam format JSON (Sub-CPMK 4 / Tugas 4)
├── crawler_putusan_dpsr.ipynb                    # Notebook utama (End-to-End Pipeline Tugas 3 & Tugas 4)
├── requirements.txt                              # Daftar pustaka Python untuk eksekusi lokal & Local Runtime Colab
├── .gitignore                                    # Konfigurasi pengabaian folder kerja sementara & cache
└── README.md                                     # Dokumentasi utama repositori
```

---

## ⚙️ Metodologi & Arsitektur Sistem

### 1. Akuisisi Data Hibrida (*Database Priority, Auto-TopUp Crawler & Merge* — Sub-CPMK 3)
Sistem akuisisi dirancang menggunakan pendekatan **Hybrid Fault-Tolerant Crawler** terhadap portal [Direktori Putusan Mahkamah Agung RI](https://putusan3.mahkamahagung.go.id/):
* **Kepatuhan Etika Crawler (`robots.txt` & *Jitter Politeness*)**: Memeriksa izin akses direktori melalui `urllib.robotparser` pada `/robots.txt` dan menerapkan jeda waktu acak (*jitter politeness delay*) antara **2,0 hingga 3,5 detik** antar-permintaan HTTP untuk menghindari *rate-limiting* WAF.
* **Penanganan *Geo-Blocking* (`HTTP 403`) & Rotasi Sesi TLS**: Mengintegrasikan *Persistent TLS Fingerprint Impersonation* (`curl_cffi` dengan rotasi profil `chrome120`, `chrome119`, `edge101`, `safari15_5`), koneksi langsung *Local Runtime* (IP Indonesia), serta *Web Archive Mirror Fallback*.
* **Prioritas Basis Data & Penggabungan Otomatis (*Auto-Merge*)**:
  1. Mengekstrak **64 dokumen PDF** dari arsip terverifikasi `Dataset/Narkotika.zip` sebagai **Prioritas 1 (`Basis Data Manual`)**.
  2. Menghitung kekurangan kuota terhadap `target_jumlah_dokumen` (misal target 70 dokumen $\rightarrow$ kurang 6 dokumen), lalu menjalankan *crawler* untuk mengunduh **6 dokumen PDF baru** secara *live* dari server Mahkamah Agung (`Live Web Crawler`).
  3. Menggabungkan seluruh berkas di `./Folder_PDF`, memeriksa duplikasi menggunakan kunci `No Putusan`, dan mengekstrak nama berkas asli dari *header* `Content-Disposition`.

### 2. Pembersihan *Noise*, Ekstraksi *Regex* 15 Atribut & *HTML Table Fallback* (Sub-CPMK 3)
Dokumen PDF dikonversi menjadi teks utuh menggunakan `PyMuPDF` (`pymupdf`), dibersihkan dari serpihan *watermark* Mahkamah Agung serta karakter Unicode rusak, lalu diekstraksi menggunakan pola *Regex* berlapis yang dikombinasikan dengan *fallback* tabel HTML (`<table class="table">`) menjadi **15 atribut terstruktur**:

1. `No` (Nomor Urut: `1` s/d `70`)
2. `No Putusan` (Tingkat keterisian: **100%**)
3. `Lembaga Peradilan` (`PN Denpasar` — **100%**)
4. `Barang Bukti` (Tingkat keterisian: **100%**)
5. `Amar Putusan` (Diambil secara presisi dari blok `MENGADILI:` terakhir — **100%**)
6. `Nama Terdakwa` (Tingkat keterisian: **100%**)
7. `Umur / Tgl Lahir` (Tingkat keterisian: **100%**)
8. `Jenis Kelamin` (Tingkat keterisian: **100%**)
9. `Pekerjaan Terdakwa` (Tingkat keterisian: **98,6%**)
10. `Tuntutan JPU` (Tingkat keterisian: **98,6%**)
11. `Pasal Terbukti` (Pasal UU RI No. 35 Tahun 2009 tentang Narkotika — **97,1%**)
12. `Tanggal Putusan` (Tingkat keterisian: **100%**)
13. `Hakim Ketua` (Tingkat keterisian: **100%**)
14. `Nama File PDF` (Tingkat keterisian: **100%**)
15. `Sumber Akuisisi` (`Basis Data Manual` / `Live Web Crawler` — **100%**)

### 3. *Text Preprocessing* & Normalisasi Domain Hukum (Sub-CPMK 4)
Seluruh teks dari **70 dokumen putusan utuh** (*full-text*) diproses melalui 5 tahapan terstandar:
1. **Case Folding**: Penyeragaman seluruh karakter menjadi huruf kecil.
2. **Tokenization & Alphabet Filtering**: Pemisahan token alfabet ($\ge 3$ karakter).
3. **Equivalence Classing (Normalisasi Istilah Narkotika)**: Penyeragaman ejaan tidak baku dan sinonim domain narkotika/hukum (`shabu`/`sabu-sabu` $\rightarrow$ `sabu`, `metamfetamina`/`methamphetamine` $\rightarrow$ `metamfetamin`, `mariyuana`/`kanabis` $\rightarrow$ `ganja`, `extacy`/`ineks` $\rightarrow$ `ekstasi`, `subsidair` $\rightarrow$ `subsider`).
4. **Stopword Removal**: Penghapusan kata umum bahasa Indonesia (`PySastrawi`) yang diperkaya dengan *custom legal stopwords*.
5. **Cached Stemming**: Pemulihan kata berimbuhan ke bentuk dasar menggunakan algoritma *Nazief & Adriani* (`PySastrawi`) dengan optimasi *in-memory dictionary caching* (memproses 6.999 kata unik dalam **1,94 detik**).

### 4. *Inverted Index* & Model Perankingan TKI (Sub-CPMK 4)
Sistem membangun **Inverted Index** dari $N = 70$ dokumen dengan ukuran *Vocabulary* $M = 5.119$ *unique terms* dan rata-rata panjang dokumen $avgdl = 4.651,4$ *terms*/dokumen. Pencarian dan perankingan dokumen diimplementasikan melalui dua model utama:

* **Vector Space Model (Sublinear TF-IDF + Cosine Similarity)**:
  $$w_{t,d} = \left(1 + \log_{10} tf_{t,d}\right) \times \log_{10}\left(\frac{N}{df_t}\right)$$
  dengan normalisasi panjang vektor Euclidean (*L2 Norm*) untuk menghitung skor kemiripan kosinus antara vektor kueri dan dokumen.

* **Probabilistic Information Retrieval (Okapi BM25)**:
  $$RSV_{BM25}(q, d) = \sum_{t \in q} \log_{10}\left(1 + \frac{N - df_t + 0.5}{df_t + 0.5}\right) \cdot \frac{(k_1 + 1) tf_{t,d}}{k_1 \left((1 - b) + b \times \frac{dl}{avgdl}\right) + tf_{t,d}}$$
  menggunakan parameter penjenuhan frekuensi $k_1 = 1.5$ dan normalisasi panjang dokumen $b = 0.75$.

---

## 📦 Kebutuhan Pustaka (*Requirements*)

Seluruh pustaka Python yang dibutuhkan telah dirangkum di dalam berkas `requirements.txt`:

```text
pymupdf>=1.24.0
beautifulsoup4>=4.12.0
requests>=2.31.0
urllib3>=2.0.0
pandas>=2.0.0
numpy>=1.24.0
openpyxl>=3.1.0
cloudscraper>=1.2.71
curl_cffi>=0.7.0
PySastrawi>=1.2.0
scikit-learn>=1.3.0
jupyter>=1.0.0
notebook>=7.0.0
jupyter_http_over_ws>=0.0.8
```

Untuk menginstal seluruh kebutuhan pustaka sekaligus di komputer lokal, jalankan:
```bash
pip install -r requirements.txt
```

---

## 🚀 Panduan Menjalankan Proyek

### Opsi 1: Menghubungkan Google Colab ke *Local Runtime* (IP Indonesia — Sangat Disarankan)
Menggunakan fitur **Connect to a local runtime** memungkinkan antarmuka Google Colaboratory berjalan menggunakan perangkat keras lokal dan **koneksi internet Indonesia (Non-Cloud IP)** sehingga *crawler* bebas dari pemblokiran *Geo-Blocking* `HTTP 403` Mahkamah Agung:

1. **Kloning Repositori & Instalasi Dependensi Lokal**  
   Buka **PowerShell** atau **Terminal VS Code**, arahkan ke folder kerja, lalu instal pustaka yang diperlukan:
   ```powershell
   git clone https://github.com/3cdiqolbu/Crawler-PN-Denpasar-Narkotika.git
   cd Crawler-PN-Denpasar-Narkotika
   pip install -r requirements.txt
   ```

2. **Nyalakan Server Jupyter Lokal dengan Izin CORS Google Colab**  
   Jalankan perintah berikut di PowerShell pada direktori proyek (pastikan `Narkotika.zip` berada di direktori tersebut):
   ```powershell
   python -m jupyter server --ServerApp.allow_origin='https://colab.research.google.com' --port=8888 --ServerApp.port_retries=0
   ```
   *(Jika menggunakan versi Jupyter Notebook klasik, gunakan perintah: `jupyter notebook --NotebookApp.allow_origin='https://colab.research.google.com' --port=8888 --NotebookApp.port_retries=0`)*.

3. **Salin URL Token Localhost**  
   Di terminal PowerShell akan muncul URL beserta token autentikasi seperti contoh berikut:
   ```text
   http://localhost:8888/tree?token=a1b2c3d4e5f6...
   ```
   Salin seluruh URL tersebut.

4. **Hubungkan dari Antarmuka Google Colaboratory**  
   * Buka berkas `crawler_putusan_dpsr.ipynb` di [Google Colaboratory](https://colab.research.google.com/).
   * Klik tanda panah kecil di sebelah tombol **Connect** (pojok kanan atas layar).
   * Pilih menu **"Connect to a local runtime..."**.
   * Tempelkan (*paste*) URL `http://localhost:8888/?token=...` ke dalam kolom **Backend URL**, lalu klik **Connect**.
   * Setelah indikator berubah menjadi **Connected (Local)**, jalankan seluruh sel melalui menu **Runtime** $\rightarrow$ **Run all** (`Ctrl + F9`).

### Opsi 2: Eksekusi Langsung di *Cloud* Google Colaboratory
1. Buka berkas `crawler_putusan_dpsr.ipynb` di Google Colaboratory menggunakan *Cloud Runtime* standar.
2. Jalankan **Runtime** $\rightarrow$ **Run all** (`Ctrl + F9`).
3. Apabila berkas `Narkotika.zip` belum diunggah ke sesi Colab, *pipeline* akan mengunduhnya secara otomatis dari `GITHUB_RAW_ZIP_URL` (`https://raw.githubusercontent.com/3cdiqolbu/Crawler-PN-Denpasar-Narkotika/main/Dataset/Narkotika.zip`) serta memanfaatkan *TLS Chrome Impersonation* dan *Web Archive Mirror* untuk mengekstrak kandidat putusan tambahan.

### Opsi 3: Eksekusi Langsung di Visual Studio Code (Jupyter Extension)
1. Buka folder `Crawler-PN-Denpasar-Narkotika` di **Visual Studio Code**.
2. Buka berkas `crawler_putusan_dpsr.ipynb`, pilih *kernel* Python 3.11 lokal, lalu klik tombol **Run All**.

---

## ⚖️ Lisensi & Ketentuan Penggunaan Data

Seluruh dokumen putusan pengadilan di dalam repositori ini bersumber dari publikasi resmi **Direktori Putusan Mahkamah Agung Republik Indonesia** berdasarkan Undang-Undang Nomor 14 Tahun 2008 tentang Keterbukaan Informasi Publik (UU KIP). Korpus dan kode sumber ini disusun semata-mata untuk kepentingan akademik, riset, dan pembelajaran pada mata kuliah Temu Kembali Informasi.
