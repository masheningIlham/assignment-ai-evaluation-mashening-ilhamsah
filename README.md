# Laporan Evaluasi IT Helpdesk Assistant

## 1. Objective Evaluasi

Evaluasi dilakukan untuk memastikan prototype IT Helpdesk Assistant
cukup andal sebelum dipertimbangkan masuk ke tahap produksi. Pengujian
berfokus pada:

1.  **Relevansi dan akurasi jawaban** --- jawaban sesuai dengan
    knowledge base (KB), termasuk saat pertanyaan menggunakan bahasa
    informal atau parafrase.
2.  **Groundedness dan hallucination control** --- sistem tidak
    mengarang kebijakan atau informasi ketika konteks tidak tersedia.
3.  **Ketepatan routing dan tool calling** --- permintaan pengecekan
    tiket diarahkan ke tool/database, bukan dijawab melalui semantic
    search.
4.  **Kinerja** --- mengukur latency routing, embedding, FAISS
    retrieval, dan LLM, serta mengidentifikasi bottleneck.
5.  **Efisiensi biaya** --- memantau input, output, dan total token
    sebagai indikator konsumsi API.
6.  **Robustness dan safety** --- memeriksa perilaku sistem terhadap
    permintaan ambigu, data yang tidak ditemukan, query campuran, dan
    pertanyaan di luar cakupan.

Hasil evaluasi digunakan untuk menentukan isu prioritas, menganalisis
akar penyebab, dan menyusun perbaikan serta rekomendasi sebelum
deployment produksi.

## 2. Sistem yang Dievaluasi

Evaluasi menggunakan **sistem IT Helpdesk Assistant yang dikembangkan
sendiri pada Assignment 4**, bukan reference system.

Alasannya, prototype tersebut telah berhasil dijalankan dan pengujian
sebelumnya menunjukkan sistem dapat menjawab pertanyaan karyawan
berbasis knowledge base serta melakukan pengecekan status tiket melalui
tool call. Dengan demikian, evaluasi dapat berfokus pada kualitas dan
keterbatasan implementasi yang benar-benar akan dikembangkan lebih
lanjut.

Secara garis besar, sistem terdiri dari:

-   **FastAPI** sebagai backend API dengan endpoint `/ask`.
-   **Router** untuk membedakan pertanyaan kebijakan/FAQ dan permintaan
    pengecekan status tiket.
-   **RAG pipeline** yang melakukan embedding, pencarian semantic
    menggunakan FAISS, lalu menyusun jawaban dengan LLM.
-   **Ticket lookup tool** yang membaca data tiket dari
    `mock_tickets.csv`.
-   **Knowledge base** yang berisi informasi People & IT Operations
    dalam `it_helpdesk_knowledge_base.csv`.

Untuk permintaan status tiket, sistem menggunakan lookup berdasarkan ID
tiket. Untuk pertanyaan kebijakan, sistem menggunakan retrieval dan LLM.

## 3. Batasan dan Asumsi

1.  Hasil pengujian hanya berlaku untuk 12 query yang dicantumkan dalam
    laporan; hasil ini belum mewakili seluruh variasi bahasa dan
    perilaku pengguna.
2.  Latency dipengaruhi kondisi jaringan, ketersediaan layanan model,
    antrean API, dan kondisi mesin saat pengujian. Karena itu, satu kali
    pengukuran tidak cukup untuk menyimpulkan performa produksi.
3.  Pengujian belum mencakup load/stress test, concurrency, pengujian
    autentikasi/otorisasi, kebocoran data antar pengguna, serta evaluasi
    keamanan infrastruktur.
4.  Untuk query campuran, implementasi router saat ini memprioritaskan
    intent pengecekan tiket. Sistem belum dirancang untuk menjalankan
    dua intent dan menggabungkan hasilnya dalam satu respons.

## 4. Ringkasan Hasil Evaluasi

### 4.1 Hasil 12 Evaluation Scenarios

  -------------------------------------------------------------------------
  ID                Kategori            Hasil pengujian   Status
  ----------------- ------------------- ----------------- -----------------
  EVAL-01           Relevansi dasar --  Menjawab langkah  **Aman** ---
                    formal              mengakses slip    sesuai KB.
                                        gaji melalui      
                                        portal HR, menu   
                                        Payroll, dan      
                                        tanggal unduh 25. 

  EVAL-02           Relevansi dasar --  Memahami "Cutiku  **Aman** ---
                    informal            tinggal berapa    memahami bahasa
                                        lagi?" dan        informal.
                                        mengarahkan       
                                        pengguna ke       
                                        Ringkasan Cuti    
                                        Saya di portal    
                                        HR.               

  EVAL-03           Parafrase/sinonim   Pertanyaan        **Aman** ---
                                        "training         semantic
                                        session" dikenali retrieval
                                        sebagai           menemukan
                                        pertanyaan        informasi
                                        tentang pelatihan relevan.
                                        tools internal    
                                        saat onboarding.  

  EVAL-04           Di luar cakupan     Menyatakan tidak  **Aman** ---
                                        memiliki          tidak memaksakan
                                        informasi cukup   jawaban.
                                        tentang subsidi   
                                        transportasi.     

  EVAL-05           Di luar cakupan     Tidak mengarang   **Aman** ---
                                        identitas pembuat tidak mengarang
                                        sistem ketika     nama.
                                        informasi tidak   
                                        tersedia.         

  EVAL-06           Akurasi tool call   `TICKET-0015`     **Aman** --- data
                                        menghasilkan      sesuai ID yang
                                        ringkasan tiket   diminta.
                                        Wi-Fi lantai 3,   
                                        status Selesai,   
                                        tanggal           
                                        2026-08-22.       

  EVAL-07           Data tidak          `TICKET-0067`     **Aman** ---
                    ditemukan           menghasilkan      kondisi not-found
                                        pesan tiket tidak ditangani.
                                        ditemukan.        

  EVAL-08           Tanpa ID tiket      Sistem meminta ID **Aman dengan
                                        dengan format     catatan** ---
                                        `TICKET-XXXX`.    tidak crash; ada
                                                          typo tanda petik
                                                          pada teks format
                                                          di hasil awal.

  EVAL-09           Query campuran      Untuk pertanyaan  **Terbatas /
                                        sisa cuti         perlu perbaikan**
                                        sekaligus status  --- sesuai
                                        tiket, router     prioritas router
                                        meminta ID tiket  saat ini, tetapi
                                        dan tidak         belum mendukung
                                        menjawab          multi-intent.
                                        pertanyaan cuti.  

  EVAL-10           Konsistensi         Pertanyaan        **Aman** ---
                    parafrase           tentang rincian   jawaban
                                        "paycheck"        konsisten.
                                        menghasilkan      
                                        langkah yang sama 
                                        dengan EVAL-01.   

  EVAL-11           Query               Tiga dari empat   **Perlu
                    kompleks/panjang    sub-pertanyaan    perbaikan** ---
                                        terjawab; alur    cakupan retrieval
                                        pengajuan cuti    dan latency.
                                        tidak ditemukan   
                                        dalam konteks.    
                                        LLM latency       
                                        mencapai 74.9452  
                                        detik.            

  EVAL-12           Baseline query      Jawaban berupa    **Aman**
                    singkat (hai)       sapaan/helpdesk   
                                        introduction.     
                                        Angka latency 
                                        hanya 1.2862 
                                        detik
  -------------------------------------------------------------------------

### 4.2 Temuan utama

1.  **Kualitas jawaban dasar cukup baik.** Sistem menangani pertanyaan
    formal, informal, dan parafrase.
2.  **Fallback di luar cakupan berjalan baik.** EVAL-04 dan EVAL-05
    menunjukkan sistem dapat menyatakan keterbatasan knowledge base.
3.  **Jalur tool call bekerja pada skenario dasar.** ID tiket valid, ID
    tidak ditemukan, dan permintaan tanpa ID ditangani sesuai kondisi.
4.  **Router belum mendukung multi-intent.** Pada EVAL-09, prioritas
    ticket status membuat pertanyaan cuti tidak diproses.
5.  **Query kompleks mengungkap keterbatasan top-k.** Dengan konteks
    retrieval yang dibatasi tiga dokumen, salah satu dari empat
    sub-pertanyaan tidak memperoleh konteks yang dibutuhkan.
6.  **LLM menjadi bottleneck latency.** EVAL-11 membutuhkan 74.9452
    detik pada tahap LLM, jauh di atas skenario lainnya.

### 4.3 2 isu paling signifikan 

Di luar 12 skenario sebelumnya, juga telah dilakukan testing input empty string yang mana
sistem menghasilkan 422 Unprocessable Content. Selain itu, saat sistem diberikan query
complex yang memiliki > 3 pertanyaan, sistem hanya mampu menjawab 3 pertanyaan saja.
Kedua isu ini akan digunakan pada section 5 Root Cause Analysis

## 5. Root Cause Analysis

### 5.1 Complex Query

**Root cause yang paling mungkin:**

-   Konfigurasi retrieval membatasi konteks pada `top_k=3`.
-   Satu query mengandung beberapa sub-pertanyaan yang membutuhkan
    beberapa bagian knowledge base.
-   Retrieval dilakukan terhadap query gabungan. Embedding query panjang
    dapat lebih merepresentasikan tema dominan dan tidak selalu
    mengambil dokumen untuk setiap sub-pertanyaan.

### 5.2 Input Empty String

**Root cause yang paling mungkin:**

-   Validasi dari Pydantic min_length=1
-   Menggunakan fungsi validate query raise ValueError jika input tidak valid


## 6. Perbaikan dan Perbandingan Before vs After

### 6.1 Perbaikan 1 --- retrieval untuk query multi-topik

**Perubahan sementara:**

Karena limitasi waktu, langkah yang diambil adalah
meningkatkan  `top_k` dari 3 menjadi 5.

| Komponen | Deskripsi |
|---|---|
| **Issue ID** | ISSUE-01 |
| **Kategori** | Relevansi |
| **Permasalahan** | Tidak semua pertanyaan terjawab ketika pengguna memberikan query kompleks yang terdiri dari lebih dari 3 pertanyaan sekaligus. |
| **Evidence Before** | Sistem hanya menjawab 3 dari 4 pertanyaan pada EVAL-011. Pertanyaan mengenai alur pengajuan cuti tidak terjawab karena informasi yang relevan tidak masuk ke dalam konteks retrieval. |
| **Root Cause** | Semantic search hanya mengambil 3 konteks knowledge base paling relevan (`top_k=3`). Ketika satu query mengandung beberapa pertanyaan dengan topik berbeda, dokumen yang relevan untuk seluruh sub-pertanyaan tidak selalu terambil. |
| **Perbaikan yang Dilakukan** | Meningkatkan parameter `top_k` dari 3 menjadi 5 agar lebih banyak dokumen relevan dapat dimasukkan ke dalam konteks LLM. |
| **Alternatif Perbaikan Ideal** | Mengimplementasikan query decomposition, yaitu memecah query kompleks menjadi beberapa sub-pertanyaan, melakukan semantic search untuk setiap sub-pertanyaan secara terpisah, kemudian menggabungkan hasil retrieval sebagai konteks LLM. Pendekatan ini belum diimplementasikan karena keterbatasan waktu dan resource. |

#### Perbandingan Before vs After

| Aspek | Before | After |
|---|---|---|
| **Top-k retrieval** | 3 dokumen | 5 dokumen |
| **Jumlah pertanyaan** | 4 pertanyaan dalam 1 query | 4 pertanyaan dalam 1 query |
| **Alur pengajuan cuti** | Tidak terjawab karena konteks tidak mencakup informasi terkait. | Terjawab: pengajuan melalui portal HR, minimal 3 hari kerja sebelumnya, dan memerlukan persetujuan atasan langsung. |
| **Cara melihat sisa cuti** | Terjawab. | Terjawab. |
| **Ketentuan sisa cuti tahun berikutnya** | Terjawab. | Terjawab. |
| **Ketentuan cuti bersama** | Terjawab. | Terjawab. |
| **Jumlah pertanyaan terjawab** | 3 dari 4 (75%) | 4 dari 4 (100%) |
| **Kualitas jawaban** | Sebagian informasi tidak berhasil ditemukan melalui retrieval. | Seluruh pertanyaan berhasil dijawab berdasarkan konteks yang diperoleh. |

**Hasil Perbaikan:** Sistem kini mampu menjawab keempat pertanyaan dalam satu query setelah parameter retrieval ditingkatkan menjadi `top_k=5`.

**Catatan Teknis:**

Peningkatan `top_k` berhasil mengatasi permasalahan pada skenario pengujian ini. Namun, pendekatan tersebut belum tentu optimal untuk seluruh query kompleks.

Beberapa hal yang perlu diperhatikan:

1. Semakin besar `top_k`, semakin banyak konteks yang dikirimkan ke LLM sehingga berpotensi meningkatkan input tokens dan latency.
2. Dokumen tambahan belum tentu semuanya relevan sehingga dapat meningkatkan noise dalam konteks.
3. Query decomposition dapat menjadi solusi lanjutan karena setiap sub-pertanyaan memiliki proses retrieval tersendiri.

**Rekomendasi:** Lakukan pengujian dengan variasi jumlah sub-pertanyaan dan bandingkan `top_k=3`, `top_k=5`, serta query decomposition untuk menemukan keseimbangan antara relevansi jawaban, latency, dan penggunaan token.

  -----------------------------------------------------------------------


### 6.2 Perbaikan 2 --- Handling Empty String dan White Spaces

**Perubahan yang disarankan:**

-   Jika sistem mendapatkan query kosong maka sistem memberikan pesan default

| Komponen | Deskripsi |
|---|---|
| **Issue ID** | ISSUE-02 |
| **Kategori** | Unexpected Input Handling |
| **Permasalahan** | Sistem memberikan error `422 Unprocessable Content` ketika pengguna mengirimkan input berupa empty string atau whitespace. |
| **Evidence Before** | Ketika menerima empty string, sistem mengembalikan HTTP 422 karena input tidak memenuhi validasi request. |
| **Root Cause** | Sistem menggunakan validasi Pydantic `min_length=1` pada parameter `query`, sehingga input kosong ditolak sebelum diproses oleh router. Selain itu, belum terdapat mekanisme fallback untuk menangani input kosong atau whitespace. |
| **Perbaikan yang Dilakukan** | Menghapus parameter `min_length=1` dan query validator, kemudian menambahkan penanganan khusus pada router untuk mendeteksi empty string atau whitespace. |
| **Fallback Response** | Sistem memberikan pesan default: "Halo! Aku adalah Helpdesk Internal. Masukkan pertanyaanmu terkait kebijakan perusahaan atau layanan IT." |

#### Perbandingan Before vs After

**Before vs after:**

| Aspek | Before | After |
|---|---|---|
| **Validasi input** | Menggunakan Pydantic `min_length=1`. | Validasi panjang minimum dihapus dan input kosong ditangani pada router. |
| **Empty string** | Menghasilkan HTTP 422 Unprocessable Content. | Menghasilkan pesan default yang informatif. |
| **Whitespace** | Berpotensi menyebabkan input tidak tertangani dengan baik. | Dikenali sebagai input kosong dan menghasilkan pesan default. |
| **System stability** | Request dengan input kosong ditolak oleh validasi API. | Sistem dapat menangani input kosong tanpa menghasilkan error 422. |
| **Response** | `422 Unprocessable Content`. | `Halo! Aku adalah Helpdesk Internal. Masukkan pertanyaanmu terkait kebijakan perusahaan atau layanan IT.` |

**Hasil Perbaikan:** Sistem kini dapat menangani input berupa empty string maupun whitespace tanpa mengalami error. Alih-alih mengembalikan HTTP 422, sistem memberikan pesan informatif yang mengarahkan pengguna untuk memasukkan pertanyaan terkait kebijakan perusahaan atau layanan IT.

  -----------------------------------------------------------------------


## 7. Hasil Safety dan Guardrail Testing

Pengujian safety yang telah dilakukan masih bersifat fungsional dan
terbatas pada cakupan knowledge base serta validasi input tool.

  -----------------------------------------------------------------------
  Pengujian               Evidence                Penilaian
  ----------------------- ----------------------- -----------------------
  Pertanyaan di luar KB   EVAL-04 menolak memberi **Lulus pada skenario
                          informasi subsidi       ini** --- fallback
                          transportasi yang tidak bekerja.
                          tersedia.               

  Pertanyaan identitas    EVAL-05 tidak mengarang **Lulus pada skenario
  yang tidak tersedia     nama pembuat sistem.    ini** --- tidak membuat
                                                  klaim tanpa sumber.

  ID tiket tidak          EVAL-07 memberi pesan   **Lulus pada skenario
  ditemukan               not-found untuk         ini** --- tidak
                          `TICKET-0067`.          mengembalikan data
                                                  tiket lain.

  ID tiket tidak          EVAL-08 meminta ID      **Lulus dengan
  disertakan              dengan format yang      catatan** --- perbaiki
                          diharapkan dan tidak    typo formatting.
                          crash.                  

  Query campuran          EVAL-09 memprioritaskan **Terbatas** --- bukan
                          ticket lookup dan       crash, tetapi
                          meminta ID.             pertanyaan lain tidak
                                                  dijawab.
  -----------------------------------------------------------------------

## 8. Rekomendasi Teknis Lanjutan

Prioritas berikut belum dibuktikan selesai oleh 12 skenario yang
tersedia.

  -----------------------------------------------------------------------
  Prioritas               Rekomendasi             Alasan
  ----------------------- ----------------------- -----------------------
  P0                      Tambahkan timeout,      Mencegah request
                          penanganan exception,   menggantung terlalu
                          dan batas retry untuk   lama; retry harus
                          LLM API                 dibatasi agar tidak
                                                  memperbesar biaya.

  P0                      Terapkan autentikasi    Data tiket dapat memuat
                          dan otorisasi pada      informasi internal dan
                          ticket lookup           tidak boleh dapat
                                                  diakses hanya dengan
                                                  menebak ID.

  P1                      Implementasikan         Memungkinkan FAQ dan
                          multi-intent routing    ticket status diproses
                                                  dalam satu query,
                                                  dengan respons
                                                  terstruktur untuk tiap
                                                  intent.

  P1                      Tambahkan evaluasi      Mengukur kualitas
                          retrieval: Recall@k,    retrieval secara
                          MRR, dan pemeriksaan    terpisah dari kualitas
                          relevansi dokumen       generasi jawaban.

  P1                      Buat dataset evaluasi   Dua belas query belum
                          yang lebih luas dan     cukup untuk mengukur
                          ground truth per        generalisasi dan
                          pertanyaan              regresi.

  P1                      Tambahkan               Memastikan konten
                          prompt-injection dan    user/dokumen tidak
                          adversarial test        mengubah instruksi
                                                  sistem.

  P1                      Jalankan pengujian      Mengurangi bias dari
                          berulang dan laporkan   satu kali pengukuran
                          p50/p95 latency         dan mengukur tail
                                                  latency.

  P2                      Evaluasi caching untuk  Berpotensi mengurangi
                          embedding/query atau    latency dan pemanggilan
                          respons statis          API untuk input
                                                  berulang.

  P2                      Pantau token dan biaya  Membantu membuat
                          per intent/model        estimasi biaya
                                                  operasional dan budget
                                                  produksi.
                                                  
  -----------------------------------------------------------------------
