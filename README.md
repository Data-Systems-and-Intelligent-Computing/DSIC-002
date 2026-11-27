# DSIC-002 — Data-Impact Evaluation of Infrastructure-as-Code Storage Guards in an AudioMoth Data Lake

Repositori penelitian **DSIC-002**.

> README ini adalah spesifikasi penelitian yang disusun pembimbing (Supervisor 2 atas permintaan Supervisor 1, 2026-11-27). Isinya rancangan, bukan hasil. Semua angka di bawah yang belum diukur ditulis sebagai rencana.

Judul penelitian yang direkomendasikan:

> **Do Infrastructure-as-Code Guards Protect the Data? A Data-Impact Evaluation of Storage Misconfigurations in an AudioMoth Data Lake**

Alternatif berbahasa Indonesia untuk skripsi:

> **Evaluasi Dampak Data terhadap Penjaga Infrastructure-as-Code pada Miskonfigurasi Penyimpanan Data Lake AudioMoth**

Judul yang mengandung "first", "novel", atau "comprehensive" tidak boleh dipakai sebelum verifikasi literatur Hari 1 selesai.

---

## Keputusan Pembimbing

Penelitian ini memakai **Opsi A** (disetujui Supervisor 1, 2026-11-27).

Objek ilmiahnya:

> **apakah pemeriksaan otomatis terhadap kode Infrastructure-as-Code (IaC) benar-benar mencegah kerusakan data, diukur dari keadaan data yang sebenarnya, bukan dari ada atau tidaknya alarm.**

AudioMoth hanya menjadi **testbed** berupa korpus WAV beku. WAV diperlakukan sebagai **objek biner yang tidak boleh berubah**. Tidak ada klasifikasi spesies, ekstraksi fitur, atau analisis ekologis.

Kontribusi ilmiah **bukan** penggunaan:
- OpenTofu atau Terraform;
- AWS S3, KMS, atau IAM;
- Checkov, tfsec, OPA, atau Conftest;
- AudioMoth;
- SHA-256;
- mutation testing sebagai teknik.

Kontribusi yang diuji:

> **evaluasi terkontrol berbasis mutasi terhadap penjaga IaC (analisis statis kode, analisis `plan`, aturan invarian data, dan audit sesudah deploy) dengan acuan kebenaran berupa kerusakan data nyata pada data lake, sehingga presisi, recall, dan tahap deteksi setiap penjaga dapat diukur terhadap dampak, bukan terhadap label smell.**

Penelitian ini **berbeda** dari:
- **DSIC-2606 (Vania):** kebenaran data sesudah pipeline Spark crash, dengan faktor checkpoint × idempotence. DSIC-002 tidak memakai Spark, Iceberg, kode pipeline, maupun hasil DSIC-2606.
- **DSIC-2604 (Desman):** tata letak Parquet dan latensi kueri. DSIC-002 tidak mengukur performa kueri.

Yang sama dengan DSIC-2606 hanya gagasan metode, yaitu mencocokkan keadaan akhir dengan manifest acuan. Gagasan ini dikutip sebagai inspirasi metode.

---

## Kalimat Jangkar Penelitian

> Penelitian ini menyuntikkan mutasi konfigurasi yang realistis ke lapisan penyimpanan data lake berisi korpus AudioMoth yang dikelola OpenTofu, menjalankan skenario operasi data yang terkendali, lalu mengukur mutasi mana yang benar-benar merusak data dan pada tahap mana (kode, `plan`, aturan invarian data, audit sesudah deploy, atau rekonsiliasi data) kerusakan itu pertama kali terdeteksi.

---

## Publication Boundary

Penelitian ini **tidak** mengklaim:
- Checkov, tfsec, atau OPA "gagal" secara umum;
- hasil berlaku untuk semua penyedia cloud;
- daftar mutasi mencakup semua miskonfigurasi yang mungkin;
- aturan invarian data yang diusulkan aman untuk produksi;
- penelitian ini yang pertama mengevaluasi IaC terhadap data.

Bahasa klaim yang aman:

> **pada data lake uji dan katalog mutasi yang dibekukan, sebanyak X dari Y mutasi yang merusak data tidak terdeteksi oleh pemeriksaan statis mana pun dan baru terungkap lewat rekonsiliasi data.**

Gunakan istilah:

```text
data-impact oracle
detection stage
harmful mutation
escape (mutasi merusak yang lolos semua penjaga statis)
```

Jangan memakai kata "aman", "terlindungi", atau "terjamin" untuk konfigurasi apa pun tanpa menyebut katalog mutasi dan skenario yang diuji.

---

## Posisi terhadap Literatur dan Kebaruan yang Aman

### Karya terdekat (teks lengkap sudah dibaca pembimbing)

**Verdet, Hamdaqa, Da Silva, Khomh (2025).** *Assessing the adoption of security policies by developers in Terraform across different cloud providers.* Empirical Software Engineering 30:74. DOI 10.1007/s10664-024-10610-0.

- Mengukur **adopsi** kebijakan keamanan pada 812 proyek GitHub Terraform dengan Checkov dan tfsec. Kebijakan dipetakan ke 8 kategori: Hard-coded secret, IP Address binding, Admin by default, Encryption at rest, Encryption in transit, Access Policy, Logging/Monitoring, Outdated feature.
- Penulis menyatakan studinya hanya melihat konfigurasi Terraform, dan hasilnya bergantung pada alat.
- **Paket replikasi:** https://github.com/averdet/master-terraform-sec-adoption (commit terakhir 12 Nov 2024; Checkov 2.2.21 menurut README-nya).
- **Diperiksa pembimbing 2026-11-27:** `aws/csv/policies.csv` berisi 121 kebijakan AWS terpilih, dengan sebaran kategori: Access policy 29, Admin by default 6, Encryption at rest 35, Encryption in transit 15, Hard-coded secret 4, IP Address binding 12, Logging/Monitoring 16, Outdated feature 4.
- **Tidak satu pun** dari 121 kebijakan itu menyangkut versioning, backup, lifecycle, retensi, Object Lock, atau pencegahan hapus.
- Artinya, perlindungan data dari kehilangan dan penimpaan berada **di luar** kerangka kategori yang dipakai studi adopsi tersebut.

**Hu, Bu, Wong, Sood, Smiley, Rahman (2023).** *Characterizing Static Analysis Alerts for Terraform Manifests: An Experience Report.* Pre-print; venue SecDev 2023 belum terverifikasi dari PDF.

- Menganalisis 491 alarm Checkov (10 repo OSS + 1 repo perusahaan) dan mensurvei 10 praktisi.
- Kategori yang menyentuh data sudah muncul sebagai alarm: **Backup and recovery** (12,2 % perusahaan; 1,7 % OSS) dan alarm versioning S3 `CKV_AWS_21`.
- 5 dari 10 praktisi tidak mau memperbaiki alarm replikasi S3.
- Alarm dinilai dari frekuensi dan persepsi, **bukan** dari apakah data benar-benar rusak.

### Celah yang diuji

```text
Studi sebelumnya:  kode IaC → alat → alarm / pass-rate → (dianggap) risiko
DSIC-002:          kode IaC → mutasi → deploy → operasi data → keadaan data nyata
                                  ↘ alat (T1–T4) → alarm → dibandingkan dengan dampak
```

### Status kebaruan

> Potensi kontribusi teridentifikasi, yaitu evaluasi penjaga IaC berdasarkan dampak pada data. **Verifikasi literatur eksternal masih diperlukan** (Hari 1), terutama dua makalah Springer berikut yang belum terbaca:
> - *Preventing Amazon S3 Cloud Storage Misconfiguration Using Infrastructure-as-Code: A Policy-Enforced Security Framework* (DOI 10.1007/s44227-026-00114-2);
> - *Comparative Analysis of IaC Misconfiguration Detection Through Policy Driven Evaluation and Taint-Aware Static Reasoning* (DOI 10.1007/978-3-032-27160-0_37).

Yang **sudah ada** dan tidak boleh diklaim sebagai kontribusi:
- perbandingan Checkov/tfsec/KICS;
- aturan kebijakan S3;
- mutasi konfigurasi Terraform (sudah ada dataset TF-Mutn di TerraFormer, ICSE 2026);
- pengukuran adopsi kebijakan.

### Aturan pemakaian paket replikasi Verdet

Repositori `averdet/master-terraform-sec-adoption` **tidak memiliki berkas LICENSE** (diperiksa 2026-11-27). Akibatnya:
- **boleh** dibaca, dikutip, dan dirujuk (URL + hash commit);
- **jangan** menyalin berkasnya (CSV, notebook, skrip) ke repo ini;
- pemetaan kebijakan → kategori ditulis ulang sendiri di `configs/taxonomy_map.yaml` dengan merujuk paper sebagai sumber;
- bila berkas asli diperlukan untuk replikasi, minta izin penulis lebih dulu.

---

## Teori Dasar

### Siklus kerja OpenTofu

```text
HCL (.tf)  →  tofu plan  →  plan JSON  →  tofu apply  →  sumber daya cloud
   T1              T2/T3                                   T4        T5 (data)
```

- **HCL** adalah kode yang ditulis manusia. Nilai dari variabel, modul, atau fungsi belum tentu sudah terhitung di tahap ini.
- **Plan JSON** (`tofu show -json plan.out`) memuat nilai yang sudah terhitung dan daftar aksi per resource (`create`, `update`, `delete`, `["delete","create"]` untuk *replacement*).
- **State** adalah catatan OpenTofu tentang sumber daya yang dikelola. Penelitian ini tidak menguji state.

### Kontrol perlindungan data di S3 (diverifikasi ulang di pilot, Hari 4)

| Kontrol | Fungsi | Sifat penting |
|---|---|---|
| Versioning | menyimpan versi lama saat objek ditimpa atau dihapus | sekali diaktifkan hanya bisa `Suspended`, tidak bisa kembali ke "tanpa versioning" |
| Object Lock | mencegah versi objek dihapus atau ditimpa sampai retensi habis | butuh versioning; mode *governance* bisa dilewati pemegang izin `s3:BypassGovernanceRetention`; mode *compliance* tidak bisa dilewati siapa pun |
| Lifecycle | menghapus atau memindahkan objek otomatis menurut umur/prefix | bekerja **asinkron**, eksekusinya bisa tertunda dari waktu eligibilitas |
| SSE-KMS | enkripsi objek dengan kunci KMS | kunci yang dinonaktifkan atau dijadwalkan hapus membuat objek **tak terbaca** |
| IAM / bucket policy | siapa boleh `PutObject`/`DeleteObject` | izin terlalu luas memungkinkan job lain menimpa atau menghapus data |
| Blokir akses publik | mencegah ekspos publik | menyangkut kerahasiaan, **bukan** kehilangan data; dipakai sebagai mutasi kontrol |

**Catatan:** setiap klaim perilaku AWS di tabel ini wajib dibuktikan ulang dengan percobaan kecil di Hari 4 dan dicatat di `docs/aws_semantics_check.md`. Jika berbeda, tabel ini diperbaiki dan hipotesis ditinjau.

### Penjaga (guard) yang dievaluasi

| Tahap | Penjaga | Masukan |
|---|---|---|
| T1 | Checkov, aturan bawaan | direktori HCL |
| T2 | Checkov, aturan bawaan | plan JSON |
| T3 | OPA/Conftest, **aturan invarian data** buatan penelitian | plan JSON |
| T4 | audit sesudah deploy (skrip API AWS) | sumber daya nyata |
| T5 | rekonsiliasi data terhadap manifest | objek di S3 |

T5 adalah **oracle**, yaitu acuan kebenaran, bukan penjaga yang dibandingkan.

### Mutation analysis

Mutasi adalah perubahan kecil dan disengaja pada konfigurasi awal yang benar. Mutasi yang membuat data rusak disebut **harmful**. Penjaga yang memberi alarm pada mutasi harmful menghasilkan *true positive*; alarm pada mutasi tidak harmful adalah *false positive* **terhadap ukuran dampak data**.

---

## Research Question

### RQ Utama

**RQ1.** Dari katalog mutasi konfigurasi lapisan penyimpanan, berapa proporsi yang menyebabkan kerusakan data terukur (hilang, tertimpa tanpa versi yang bisa dipulihkan, atau tak terbaca), dan pada skenario operasi data apa?

### Sub-RQ

**RQ2.** Untuk mutasi yang merusak data, berapa proporsi yang terdeteksi pada setiap tahap T1–T4, dan berapa yang **hanya** terungkap pada T5?

**RQ3.** Diukur terhadap kerusakan data nyata, bagaimana presisi dan recall aturan bawaan Checkov (T1/T2) dibandingkan aturan invarian data (T3)?

**RQ4 (deskriptif).** Kelompok mutasi mana (versi, retensi, lifecycle, siklus hidup resource, IAM, kunci) yang paling sering lolos semua penjaga statis?

---

## Hipotesis

Semua hipotesis boleh ditolak. Hasil yang menolak hipotesis dilaporkan sama lengkapnya.

### H1 — Ada Escape

Setidaknya satu mutasi harmful lolos T1, T2, dan T3. Yang diduga paling sering lolos adalah mutasi yang baru berbahaya bila digabung dengan operasi data tertentu.

Jika tidak ada escape, laporkan sebagai temuan, lalu periksa apakah katalog mutasi terlalu mudah (lihat Quality Gate — Katalog Mutasi).

### H2 — Plan Lebih Tajam daripada HCL

Recall T2 lebih tinggi daripada T1 pada mutasi harmful, karena nilai dari variabel, modul, dan dependensi baru terhitung di `plan`.

Jika T1 = T2, laporkan. Kemungkinan penyebabnya: katalog tidak memuat mutasi yang bergantung pada variabel atau modul.

### H3 — Aturan Invarian Data Lebih Presisi

Terhadap ukuran dampak data, presisi T3 lebih tinggi daripada T1/T2 tanpa recall yang lebih rendah. Aturan bawaan diduga memberi banyak alarm untuk mutasi yang tidak merusak data.

### H4 — Kelompok Mutasi Berpengaruh

Proporsi escape berbeda menurut kelompok mutasi.

---

## Dataset

Gunakan:

```text
Subset beku korpus AudioMoth DSIC-2606
```

Asal korpus: rekaman AudioMoth kampus ITERA 23–25 September 2026, firmware `AudioMoth-Firmware-Basic (1.12.1)` (lihat `DSIC-2606/docs/device_provenance.md`). Pemakaiannya diputuskan di **D-03** dan wajib diberi atribusi ke DSIC-2606.

Subset (D-03):
- 30 berkas WAV (sekitar 100 MB), diambil merata dari lokasi yang tersedia (Main, Kantin, Embung D, Kebun Raya);
- urutan dan daftar berkas dibekukan sekali di Hari 2 dan tidak diubah lagi.

Laporkan angka nyata:
- jumlah berkas;
- total byte;
- jumlah lokasi/perangkat;
- rentang waktu rekaman;
- distribusi ukuran berkas.

Isi audio tidak dianalisis. Ukuran kecil disengaja: penelitian ini mengukur nasib setiap objek, bukan throughput.

---

## Ground-Truth Manifest

Setiap objek wajib memiliki:

```text
object_id
source_path
location
start_time
file_size_bytes
sha256
expected_key            # raw/<location>/<YYYY-MM-DD>/<file>.WAV
expected_kms_key_alias
```

Manifest dibekukan (`data/manifests/manifest_v1.csv` + `manifest_v1.sha256`) sebelum mutasi pertama. Manifest tidak boleh diubah sesudah Hari 3.

### Label objek sesudah skenario

| Label | Definisi |
|---|---|
| `INTACT` | versi terbaru ada dan SHA-256 sama dengan manifest |
| `OVERWRITTEN_RECOVERABLE` | versi terbaru beda hash, tetapi versi dengan hash manifest masih ada |
| `OVERWRITTEN_LOST` | versi terbaru beda hash dan tidak ada versi dengan hash manifest |
| `DELETED_RECOVERABLE` | versi terbaru adalah delete marker, tetapi versi dengan hash manifest masih ada |
| `DELETED_LOST` | tidak ada versi dengan hash manifest sama sekali |
| `UNREADABLE` | objek ada tetapi `GetObject` gagal karena kunci/izin dekripsi |

**Kerusakan data (harm)** = setidaknya satu objek berlabel `OVERWRITTEN_LOST`, `DELETED_LOST`, atau `UNREADABLE`.
Label `*_RECOVERABLE` dicatat terpisah sebagai **degradasi**, bukan kerusakan.

---

## Unit Analisis

```text
sel = (mutasi m, skenario s, ulangan r)
```

Untuk setiap sel dicatat:
- label setiap objek (T5);
- harm ya/tidak;
- jumlah objek rusak;
- alarm baru pada T1, T2, T3, T4 **relatif terhadap konfigurasi awal C0**;
- tahap deteksi pertama.

**Alarm relatif (delta):** alarm yang sudah muncul pada C0 dikurangkan. Hanya alarm yang muncul karena mutasi yang dihitung. Tanpa ini, alarm bawaan C0 mengotori hasil.

---

## Arsitektur Sistem Uji

```text
                    Akun AWS sandbox (D-05)
 ┌─────────────────────────────────────────────────────────────┐
 │  s3://dsic002-<run>-raw        (versioning + Object Lock     │
 │     raw/<lokasi>/<tgl>/*.WAV    governance, retensi pendek)  │
 │                                                              │
 │  s3://dsic002-<run>-curated    (versioning)                  │
 │     manifest/, metadata/*.csv                                │
 │                                                              │
 │  s3://dsic002-<run>-staging    (lifecycle 1 hari, boleh)     │
 │                                                              │
 │  KMS key  alias/dsic002-<run>  (SSE-KMS untuk raw, curated)  │
 │                                                              │
 │  IAM role ingest    : PutObject raw/, curated/               │
 │  IAM role cleaner   : DeleteObject staging/ SAJA             │
 │  IAM role reader    : GetObject + kms:Decrypt                │
 └─────────────────────────────────────────────────────────────┘
        ▲ tofu apply (per sel)            ▲ skrip skenario (boto3)
        │                                 │ memakai role sesuai skenario
   repo DSIC-002 (HCL, mutasi, aturan Rego, runner, rekonsiliator)
```

Setiap sel memakai **prefix nama unik** (`<run>`) dan tag `dsic002:run_id`, sehingga sel tidak saling mengganggu dan pembersihan bisa dilakukan per tag.

---

## Konfigurasi Awal Aman (C0)

C0 adalah konfigurasi acuan yang **harus** lolos E0, yaitu nol kerusakan di semua skenario.

| Kontrol | Resource OpenTofu (provider AWS) |
|---|---|
| Versioning raw dan curated | `aws_s3_bucket_versioning` (`Enabled`) |
| Object Lock raw | `object_lock_enabled = true` + `aws_s3_bucket_object_lock_configuration` (governance, retensi default pendek, misalnya 1 hari) |
| Enkripsi | `aws_s3_bucket_server_side_encryption_configuration` (SSE-KMS) + `aws_kms_key` |
| Lifecycle | `aws_s3_bucket_lifecycle_configuration` hanya pada bucket `staging` |
| Pencegahan hapus | `lifecycle { prevent_destroy = true }` pada bucket raw |
| Blokir publik | `aws_s3_bucket_public_access_block` |
| Izin minimum | `aws_iam_role_policy` untuk ingest, cleaner, reader seperti diagram |

**Larangan:** mode Object Lock *compliance* tidak dipakai di C0. Bucket compliance tidak bisa dibersihkan sebelum retensi habis. Mode ini hanya boleh muncul sebagai satu percobaan terkendali atas keputusan Supervisor 1.

---

## Katalog Mutasi (draf v0, dibekukan Hari 8)

Setiap mutasi harus punya: ID, kelompok, perubahan HCL, kategori asal di literatur, skenario yang relevan, dan prediksi dampak **yang ditulis sebelum dijalankan**.

| ID | Kelompok | Perubahan dari C0 | Skenario relevan | Prediksi dampak |
|---|---|---|---|---|
| M-V1 | Versi | versioning bucket `curated` → `Suspended` | S-A | tertimpa tanpa versi lama |
| M-V2 | Versi | versioning dihapus dari kode (resource dihilangkan) | S-A | perlu diamati: konfigurasi AWS mungkin tetap `Enabled` |
| M-R1 | Retensi | konfigurasi default retention Object Lock dihapus | S-A, S-B | objek baru tak terkunci |
| M-R2 | Retensi | retensi default diperpendek ke minimum | S-B (setelah retensi habis) | hapus lebih awal |
| M-R3 | Retensi | role `cleaner` diberi `s3:BypassGovernanceRetention` | S-B | governance lock dilewati |
| M-L1 | Lifecycle | aturan expiration dipindah dari `staging` ke bucket raw | S-D | delete marker, versi lama tetap |
| M-L2 | Lifecycle | M-L1 + `noncurrent_version_expiration` | S-D | versi lama ikut hilang (kecuali terkunci) |
| M-L3 | Lifecycle | filter prefix aturan dikosongkan (berlaku seluruh bucket) | S-D | seperti M-L1 pada bucket sasaran |
| M-D1 | Siklus resource | `force_destroy = true` pada bucket raw | S-C | perlu diamati bersama Object Lock |
| M-D2 | Siklus resource | `prevent_destroy` dihapus + nama bucket diubah (replacement) | S-C | bucket dan isi diganti |
| M-D3 | Siklus resource | `prevent_destroy` dihapus saja, tanpa perubahan lain | S-C | tidak ada dampak (kontrol semu) |
| M-I1 | IAM | `cleaner` diberi `s3:DeleteObject*` pada `raw/*` | S-B | hapus oleh job pembersihan |
| M-I2 | IAM | `ingest` diberi akses ke seluruh bucket raw tanpa syarat | S-A | tertimpa |
| M-I3 | IAM | bucket policy `Principal: *` untuk `s3:PutObject` dalam akun | S-A | tertimpa oleh role lain |
| M-K1 | Kunci | `is_enabled = false` pada kunci KMS | S-E | tak terbaca |
| M-K2 | Kunci | key policy mencabut `kms:Decrypt` untuk role reader | S-E | tak terbaca oleh reader |
| M-K3 | Kunci | `deletion_window_in_days` diubah (kunci tetap aktif) | S-E | tidak ada dampak (kontrol semu) |
| M-C1 | Kontrol | blokir akses publik dimatikan | semua | tidak ada kerusakan data |
| M-C2 | Kontrol | access logging dimatikan | semua | tidak ada kerusakan data |
| M-C3 | Kontrol | enkripsi diganti SSE-S3 (bukan KMS) | semua | tidak ada kerusakan data |

**Catatan:**
- Mutasi kontrol (M-C*, M-D3, M-K3) wajib ada. Tanpanya presisi tidak bisa diukur.
- Beberapa prediksi sengaja ditulis "perlu diamati". Prediksi ini dikunci di logbook sebelum dijalankan, lalu dibandingkan dengan hasil.
- Katalog boleh bertambah sampai Hari 8. Sesudah itu dibekukan; mutasi baru masuk "eksplorasi", bukan hasil utama.
- Setiap mutasi diberi rujukan kategori: Verdet (8 kategori), Hu (10 kategori alarm), Rahman 2019, War 2025, atau Oliveira 2025 (*Configuration Data*). Mutasi tanpa rujukan ditandai `novel-operator` dan dibahas terpisah.

---

## Skenario Operasi Data

| ID | Skenario | Dijalankan oleh | Tujuan |
|---|---|---|---|
| S-A | ingest ulang: menulis berkas dengan kunci sama tetapi isi berbeda (byte disisipi) | role ingest (dan role lain untuk M-I3) | menguji perlindungan dari penimpaan |
| S-B | pembersihan: job menghapus prefix yang salah ketik (`raw/` alih-alih `staging/`) | role cleaner | menguji perlindungan dari penghapusan |
| S-C | `tofu apply` sesudah mutasi, lalu `tofu destroy` terkendali bila mutasi menyangkut siklus resource | operator | menguji replacement/destroy |
| S-D | waktu berlalu, minimal 48 jam, untuk lifecycle | — | menguji aturan asinkron |
| S-E | baca seluruh objek sebagai role reader | role reader | menguji keterbacaan |

Skenario S-A dan S-B adalah **kesalahan operasional yang realistis**, bukan serangan. Penelitian ini tidak mengevaluasi penyerang yang sengaja merusak.

---

## Aturan Invarian Data (T3, draf, dibekukan Hari 13)

Aturan ditulis dalam Rego dan diuji dengan Conftest pada plan JSON. Aturan **ditulis dan dibekukan sebelum** hasil matriks utama terlihat, untuk mencegah aturan disetel agar cocok dengan hasil.

| ID | Invarian |
|---|---|
| DI-1 | bucket raw dan curated memiliki versioning `Enabled` di plan |
| DI-2 | bucket raw memiliki Object Lock aktif dengan retensi default ≥ ambang yang ditetapkan |
| DI-3 | tidak ada aturan lifecycle `expiration` atau `noncurrent_version_expiration` yang berlaku pada bucket raw |
| DI-4 | tidak ada aksi `delete` atau `["delete","create"]` pada bucket raw di `resource_changes` |
| DI-5 | tidak ada pernyataan IAM yang memberi `s3:DeleteObject*`, `s3:PutObject` atau `s3:BypassGovernanceRetention` pada `raw/*` kepada role selain ingest (dan ingest tanpa bypass) |
| DI-6 | kunci KMS yang dipakai raw/curated tetap `is_enabled = true`, dan key policy tetap memberi `kms:Decrypt` kepada role reader |
| DI-7 | `force_destroy` tidak `true` pada bucket raw |

Aturan DI-* dibuat dari **kebutuhan perlindungan data**, bukan dari katalog mutasi. Hubungan setiap aturan dengan kebutuhannya ditulis di `docs/invariants_rationale.md`.

---

## Audit Sesudah Deploy (T4)

Skrip `src/audit/post_deploy.py` memeriksa sumber daya nyata lewat API AWS:
- `GetBucketVersioning`, `GetObjectLockConfiguration`, `GetBucketLifecycleConfiguration`;
- `DescribeKey` (status kunci), `GetKeyPolicy`;
- `iam simulate-principal-policy` untuk aksi `s3:DeleteObject`/`s3:PutObject` pada `raw/*` per role.

AWS Config **tidak** dipakai sebagai default (D-08), karena biayanya bergantung pada jumlah item dan evaluasi. Bila pembanding layanan terkelola diperlukan, Config dijalankan pada subset kecil dan dicatat terpisah.

---

## Metrik

### Primer

| Metrik | Definisi |
|---|---|
| Harm rate | proporsi sel dengan harm, per mutasi dan per kelompok |
| Recall tahap Tk | sel harmful yang menghasilkan alarm delta di Tk ÷ semua sel harmful |
| Presisi tahap Tk | sel harmful dengan alarm delta di Tk ÷ semua sel dengan alarm delta di Tk |
| Escape rate | sel harmful tanpa alarm di T1, T2, T3 ÷ semua sel harmful |
| Tahap deteksi pertama | tahap terawal yang memberi alarm untuk sel harmful |

### Sekunder

| Metrik | Definisi |
|---|---|
| Objek rusak per sel | jumlah objek `*_LOST` atau `UNREADABLE` |
| Degradasi | jumlah objek `*_RECOVERABLE` |
| Biaya deteksi | waktu eksekusi T1–T4 per sel |
| Alarm delta per mutasi | jumlah aturan bawaan yang baru menyala |

---

## Matriks Eksperimen

```text
mutasi (≈20, termasuk kontrol)  ×  skenario relevan (1–3 per mutasi)  ×  ulangan
+ C0 × semua skenario (E0)
```

- Sel yang perilakunya deterministik: **1 ulangan** pada matriks utama, ditambah **1 ulangan konfirmasi** untuk setiap sel harmful. Bila keduanya berbeda, sel ditandai nondeterministik dan diulang sampai 5 kali.
- Sel lifecycle (S-D): **3 ulangan** paralel, karena eksekusinya asinkron.
- Jumlah sel akhir dicatat di Hari 14 setelah katalog dan skenario relevan dibekukan.

### Pairing

Untuk satu sel, T1–T4 dijalankan pada **konfigurasi yang sama**. Perbandingan antar-tahap adalah perbandingan **berpasangan**.

### Urutan

Urutan sel diacak dengan seed tetap (`configs/experiment.yaml`), kecuali batch lifecycle yang dimulai lebih awal karena butuh waktu.

---

## Reset Protocol

Setiap sel:
1. Buat `run_id` baru.
2. `tofu apply` C0 dengan prefix `run_id`.
3. Unggah korpus sesuai manifest, lalu verifikasi `INTACT` 30/30.
4. Terapkan patch mutasi, lalu jalankan T1, `tofu plan` → T2, T3.
5. `tofu apply` mutasi → T4.
6. Jalankan skenario.
7. Rekonsiliasi T5 → label objek.
8. Simpan bukti.
9. Pembersihan: hapus versi objek (dengan bypass governance oleh role admin eksperimen), hapus bucket, jadwalkan hapus kunci KMS sel itu (masa tunggu minimum).
10. Verifikasi tidak ada resource bertag `run_id` yang tersisa.

Kegagalan pembersihan dicatat sebagai insiden, bukan diabaikan.

---

## Raw Run Log Minimum

```text
run_id
timestamp_start / timestamp_end (UTC)
git_commit
mutation_id
scenario_id
repetition
tofu_version / aws_provider_version / checkov_version / conftest_version / opa_version
c0_alarm_snapshot_hash
t1_alarms_delta (list rule id)
t2_alarms_delta
t3_violations (list DI-*)
t4_findings
plan_actions_raw_bucket
object_labels (30 baris)
harm (bool)
cleanup_ok (bool)
notes / anomalies
```

Semua keluaran mentah (SARIF Checkov, JSON Conftest, plan JSON, hasil API) disimpan di `results/raw/<run_id>/`. Plan JSON bisa memuat ARN atau ID akun, jadi direktori ini **tidak di-push** sebelum disanitasi (lihat Aturan Reproducibility).

---

## Analisis Statistik

### Proporsi

Harm rate, recall, presisi, dan escape rate dilaporkan sebagai proporsi dengan **interval Wilson 95 %**.

### Perbandingan berpasangan antar-tahap

Perbandingan recall T1 vs T2 dan T2 vs T3 pada **sel harmful yang sama** memakai **uji McNemar eksak**, karena datanya biner dan berpasangan. Laporkan tabel 2×2 lengkap, bukan hanya nilai p.

### Kelompok mutasi (H4)

Deskriptif per kelompok dengan interval Wilson. Uji Fisher eksak hanya bila jumlah sel memadai. Karena jumlah mutasi per kelompok kecil, H4 diperlakukan sebagai **eksploratif**.

### Unit statistik

Unit statistik adalah **mutasi**, bukan objek. Tiga puluh objek dalam satu sel tidak independen. Jumlah objek rusak hanya dipakai sebagai ukuran besarnya dampak.

### Larangan

- Jangan mengklaim signifikansi untuk perbandingan dengan n kecil tanpa melaporkan tabel kontingensinya.
- Jangan menggabungkan sel eksplorasi dengan sel katalog beku.

---

## Eksperimen

### E0 — Ground Truth & C0 Sanity

C0 × semua skenario. Syarat lulus: 0 harm, 30/30 `INTACT` (atau `*_RECOVERABLE` sesuai rancangan pada S-A/S-B), alarm C0 tercatat sebagai baseline delta.

### E1 — Karakterisasi Dampak (RQ1, H4)

Semua mutasi × skenario relevan, kecuali lifecycle. Keluaran: tabel harm per mutasi dan per kelompok.

### E2 — Deteksi Statis (RQ2, H1, H2)

T1, T2, T3 untuk setiap sel E1 dan E5. Keluaran: funnel tahap deteksi, escape list, McNemar T1 vs T2.

### E3 — Deteksi Sesudah Deploy (RQ2)

T4 untuk setiap sel. Keluaran: sel yang lolos T1–T3 tetapi tertangkap T4.

### E4 — Presisi dan Recall terhadap Dampak (RQ3, H3)

Gabungan E1–E3 + mutasi kontrol. Keluaran: presisi/recall per tahap dengan interval Wilson.

### E5 — Lifecycle Asinkron

Mutasi M-L* × S-D, 3 ulangan. Dicatat waktu dari apply sampai objek pertama terdampak.

### E6 — Clean Reproduction

Clone bersih, prefix baru, satu blok sel (minimal semua mutasi harmful + 3 kontrol) dijalankan ulang. Label harus identik dengan run pertama untuk sel deterministik. Subset yang sama dijalankan di LocalStack (D-09) untuk mendokumentasikan perbedaan emulator sebagai ancaman validitas.

---

## Quality Gate — Dataset

- subset dipilih dan dibekukan;
- manifest + SHA-256 lengkap, 0 hash hilang;
- atribusi DSIC-2606 tertulis;
- unggah C0 menghasilkan 30/30 `INTACT`.

## Quality Gate — Lingkungan

- akun sandbox, anggaran, dan alarm tagihan aktif;
- versi OpenTofu, provider AWS, Checkov, Conftest/OPA dikunci di `configs/environment.yaml`;
- `docs/aws_semantics_check.md` selesai (setiap baris tabel kontrol S3 dibuktikan);
- pembersihan per tag terbukti bekerja pada satu sel dummy.

## Quality Gate — Katalog Mutasi

- setiap mutasi punya rujukan kategori dan prediksi tertulis;
- minimal 3 mutasi kontrol;
- setiap mutasi lolos `tofu validate` dan menghasilkan plan (mutasi yang gagal plan dicatat sebagai "terdeteksi oleh plan error", bukan dibuang);
- katalog dibekukan dengan hash.

## Quality Gate — Penjaga

- Checkov dijalankan pada HCL dan plan JSON C0, hasilnya disimpan sebagai baseline;
- aturan DI-* punya unit test Conftest (kasus lolos dan gagal) dan dibekukan sebelum matriks utama;
- T4 diuji pada C0 (0 temuan) dan pada satu mutasi yang pasti terlihat.

## Quality Gate — Matriks Utama

- semua sel katalog beku selesai atau tercatat gagal-instrumen dengan alasan;
- 0 resource yatim bertag `dsic002:run_id`;
- setiap sel punya raw log lengkap.

## Quality Gate — Artikel

- setiap angka di naskah dapat dilacak ke `results/processed/` dan skrip analisis;
- klaim kebaruan sesuai hasil verifikasi literatur Hari 1;
- Publication Boundary dipatuhi.

---

## Threats to Validity

### Construct validity
- "Kerusakan data" hanya diukur dalam tiga bentuk (hilang, tertimpa-tak-pulih, tak terbaca). Kebocoran kerahasiaan **tidak** diukur.
- Alarm delta bergantung pada baseline C0. C0 yang berbeda akan menghasilkan delta berbeda.

### Internal validity
- Aturan DI-* ditulis oleh peneliti yang juga menulis katalog mutasi, sehingga ada risiko aturan "menghafal" mutasi. Mitigasi: aturan diturunkan dari kebutuhan perlindungan data, dibekukan sebelum matriks, dan dasar setiap aturan ditulis.
- Lifecycle asinkron dapat membuat sel S-D tampak tidak harmful hanya karena belum dieksekusi. Mitigasi: minimal 48 jam dan pengecekan berulang.

### External validity
- Satu penyedia (AWS), satu jenis data lake, korpus kecil.
- Versi Checkov berbeda dari studi Verdet (2.2.21); kumpulan aturan bawaan berubah antarversi.
- Katalog mutasi buatan peneliti, bukan sampel miskonfigurasi nyata dari repositori publik.

### Emulator validity
- Bila LocalStack dipakai, perilaku Object Lock, lifecycle, dan KMS bisa berbeda dari AWS. Hasil utama hanya dari AWS asli.

### Novelty validity
- Dua makalah Springer belum terbaca saat README ini ditulis.

---

## Grafik dan Tabel Wajib

1. Diagram arsitektur sistem uji dan alur T1–T5.
2. Tabel katalog mutasi: kelompok, rujukan kategori, prediksi, hasil.
3. Heatmap mutasi × skenario: harm (ya/tidak) dan jumlah objek rusak.
4. Funnel tahap deteksi untuk sel harmful (T1 → T2 → T3 → T4 → hanya T5).
5. Tabel presisi/recall per tahap dengan interval Wilson.
6. Tabel McNemar T1 vs T2 dan T2 vs T3.
7. Daftar escape dengan penjelasan kualitatif per mutasi.
8. Garis waktu E5: waktu dari apply sampai dampak lifecycle.

---

## Struktur Artikel

1. Introduction
2. Background (IaC, perlindungan data S3, analisis statis IaC)
3. Related Work (Verdet 2025, Hu 2023, Rahman 2019, War 2025, Oliveira 2025, Sokolowski 2024, Spielmann 2026)
4. Study Design (sistem uji, C0, katalog mutasi, skenario, T1–T5, oracle)
5. Results (RQ1–RQ4)
6. Discussion (mengapa escape terjadi; implikasi untuk penulis aturan)
7. Threats to Validity
8. Conclusion

---

# Rencana Kerja 1 Bulan — Pembagian Hari

## Minggu 1 — Fondasi dan C0

## Hari 1 — Freeze RQ, Scope, dan Verifikasi Literatur

- baca dua makalah Springer (D-07) dan catat apakah celah tetap, menyempit, atau gugur;
- telusuri Google Scholar dengan sitasi lanjutan dari Verdet 2025 dan Hu 2023; catat query dan tanggal;
- freeze RQ, hipotesis, dan publication boundary;
- buat research logbook (`docs/logbook.md`).

Output:

```text
docs/literature_check_v1.md
RQ v1, boundary v1
```

## Hari 2 — Freeze Korpus

- pilih 30 WAV dari korpus DSIC-2606 (D-03);
- catat lokasi, waktu, ukuran;
- tulis atribusi.

Output:

```text
data/manifests/corpus_v1.txt
```

## Hari 3 — Ground-Truth Manifest

- hitung SHA-256 setiap berkas;
- isi `expected_key`;
- bekukan manifest + hash manifest.

Output:

```text
data/manifests/manifest_v1.csv
data/manifests/manifest_v1.sha256
```

## Hari 4 — Akun, Versi, dan Bukti Perilaku AWS

- siapkan akun sandbox, anggaran, dan alarm tagihan (D-05);
- kunci versi OpenTofu, provider AWS, Checkov, Conftest/OPA;
- buktikan setiap baris tabel "Kontrol perlindungan data di S3" dengan percobaan kecil (versioning suspend, governance bypass, KMS disable → `GetObject` gagal, dsb.).

Output:

```text
configs/environment.yaml
docs/aws_semantics_check.md
```

## Hari 5 — Bangun C0

- modul OpenTofu untuk tiga bucket, KMS, tiga role IAM;
- `tofu apply` C0 dengan prefix uji;
- uji pembersihan per tag pada satu sel dummy.

## Hari 6 — Rekonsiliator (T5)

- `src/reconcile/` memberi label setiap objek sesuai tabel label;
- unit test untuk keenam label (dibuat dengan kasus buatan di bucket uji).

## Hari 7 — Skenario dan E0

- skrip S-A, S-B, S-C, S-E (S-D hanya menunggu);
- jalankan E0: C0 × semua skenario;
- simpan baseline alarm Checkov C0 (HCL dan plan).

Output:

```text
Gate 1: Dataset + Lingkungan + E0 lulus
```

## Minggu 2 — Mutasi dan Penjaga

## Hari 8 — Freeze Katalog Mutasi

- lengkapi katalog dengan rujukan kategori per mutasi;
- tulis prediksi dampak di logbook sebelum menjalankan apa pun;
- bekukan katalog (`configs/mutations.yaml` + hash).

## Hari 9 — Implementasi Mutasi Versi dan Retensi

- patch M-V*, M-R* dalam bentuk berkas `mutations/<id>.patch` atau overlay variabel;
- setiap patch lolos `tofu validate`.

## Hari 10 — Implementasi Mutasi Lifecycle dan Siklus Resource

- patch M-L*, M-D*.

## Hari 11 — Implementasi Mutasi IAM, Kunci, dan Kontrol

- patch M-I*, M-K*, M-C*.

## Hari 12 — T1 dan T2

- runner Checkov pada HCL dan plan JSON (`tofu show -json`);
- pastikan Checkov membaca plan JSON keluaran OpenTofu dengan benar (bandingkan dengan satu kasus yang hasilnya diketahui);
- hitung alarm delta terhadap baseline C0.

## Hari 13 — T3: Aturan Invarian Data

- tulis DI-1..DI-7 di Rego;
- unit test Conftest;
- tulis `docs/invariants_rationale.md`;
- **bekukan aturan** sebelum matriks utama.

## Hari 14 — T4 dan Pilot

- `src/audit/post_deploy.py`;
- pilot ujung-ke-ujung pada 3 mutasi (satu harmful pasti, satu kontrol, satu "perlu diamati");
- hitung jumlah sel final dan perkiraan durasi.

Output:

```text
Gate 2: Katalog + Penjaga + pilot lulus
```

## Minggu 3 — Eksekusi

## Hari 15 — Mulai E5 (Lifecycle)

- jalankan M-L* × S-D, 3 ulangan, lebih awal karena butuh ≥ 48 jam;
- catat waktu apply.

## Hari 16 — Matriks Utama Blok 1

- mutasi kelompok Versi dan Retensi.

## Hari 17 — Matriks Utama Blok 2

- mutasi kelompok Siklus Resource dan IAM.

## Hari 18 — Matriks Utama Blok 3

- mutasi kelompok Kunci dan Kontrol.

## Hari 19 — Ulangan Konfirmasi

- ulang setiap sel harmful sekali;
- tandai sel nondeterministik dan ulang sampai 5 kali bila perlu.

## Hari 20 — Kelengkapan dan Insiden

- cek setiap sel punya raw log;
- ulang sel gagal-instrumen (alasan dicatat, run lama diarsipkan, tidak dihapus);
- verifikasi 0 resource yatim.

## Hari 21 — Baca Hasil E5

- cek objek bucket sasaran lifecycle;
- bila belum tereksekusi, tunggu dan catat; jangan menyimpulkan "tidak harmful" sebelum batas waktu yang ditetapkan.

Output:

```text
Gate 3: Matriks utama lengkap
raw results v1
```

## Minggu 4 — Analisis dan Penulisan

## Hari 22 — Analisis RQ1 dan H4

- tabel dan heatmap harm.

## Hari 23 — Analisis RQ2, H1, H2

- funnel tahap deteksi;
- McNemar T1 vs T2;
- daftar escape.

## Hari 24 — Analisis RQ3 dan H3

- presisi/recall per tahap;
- McNemar T2 vs T3.

## Hari 25 — Analisis Escape

- penjelasan kualitatif setiap escape: mengapa tidak ada aturan yang menangkapnya.

## Hari 26 — E6 Clean Reproduction

- clone bersih, prefix baru, ulang satu blok;
- bandingkan label.

## Hari 27 — Freeze Grafik dan Tabel

- semua grafik dari skrip, bukan dibuat manual.

## Hari 28 — Penulisan Hasil

## Hari 29 — Penulisan Diskusi dan Threats

## Hari 30 — Freeze Artefak

- sanitasi `results/raw/` (hapus ID akun/ARN);
- tag rilis;
- verifikasi tagihan dan 0 resource tersisa;
- `tofu destroy` seluruh lingkungan eksperimen.

---

## Definition of Done — Bulan Pertama

- [ ] `docs/literature_check_v1.md` memuat hasil dua makalah Springer dan penelusuran Scholar.
- [ ] Manifest beku dengan hash; atribusi DSIC-2606 tertulis.
- [ ] `docs/aws_semantics_check.md` membuktikan setiap baris tabel kontrol S3.
- [ ] E0 lulus: C0 nol kerusakan.
- [ ] Katalog mutasi beku, setiap mutasi punya rujukan dan prediksi tertulis sebelum dijalankan.
- [ ] Aturan DI-* beku dengan unit test sebelum matriks utama.
- [ ] Semua sel katalog beku selesai, dengan raw log lengkap.
- [ ] 0 resource yatim; tagihan sesuai anggaran.
- [ ] Tabel 2–6 dan grafik 3–4 dihasilkan dari skrip.
- [ ] E6 menghasilkan label identik untuk sel deterministik.

---

## Struktur Repository

```text
DSIC-002/
├── README.md
├── .gitignore
├── .env.example                 # tanpa kredensial; profil AWS dari luar repo
├── pyproject.toml
├── Makefile
├── configs/
│   ├── environment.yaml         # versi alat terkunci
│   ├── dataset.yaml
│   ├── mutations.yaml           # katalog beku
│   ├── scenarios.yaml
│   ├── taxonomy_map.yaml        # pemetaan mutasi → kategori literatur (ditulis ulang, bukan salinan Verdet)
│   ├── experiment.yaml          # seed, urutan, ulangan
│   └── analysis.yaml
├── infra/
│   ├── modules/datalake/        # C0
│   └── envs/sandbox/
├── mutations/                   # satu patch/overlay per mutasi
├── policies/
│   ├── rego/                    # DI-1..DI-7
│   └── tests/
├── data/
│   └── manifests/               # manifest beku (WAV tidak di-commit)
├── src/
│   ├── manifest/
│   ├── runner/                  # siklus satu sel
│   ├── scenarios/
│   ├── guards/                  # T1–T3
│   ├── audit/                   # T4
│   ├── reconcile/               # T5
│   └── analysis/
├── tests/
├── scripts/
├── results/
│   ├── raw/                     # tidak di-push sebelum disanitasi
│   ├── processed/
│   ├── figures/
│   └── tables/
├── docs/
│   ├── logbook.md
│   ├── literature_check_v1.md
│   ├── aws_semantics_check.md
│   └── invariants_rationale.md
└── paper/
```

---

## Tools Penelitian

| Alat | Peran | Catatan |
|---|---|---|
| OpenTofu | IaC, `plan`, `apply`, plan JSON | versi dikunci Hari 4 |
| Provider AWS (`hashicorp/aws`) | resource S3, KMS, IAM | versi dikunci Hari 4 |
| Checkov | T1, T2 | versi dikunci; versi Verdet (2.2.21) dicatat sebagai pembanding |
| OPA + Conftest | T3 | versi dikunci |
| Python + boto3 | skenario, T4, T5, analisis | lingkungan via `uv` |
| statsmodels / scipy | Wilson, McNemar eksak, Fisher | — |
| LocalStack | pilot dan perbandingan emulator (opsional) | bukan sumber hasil utama |

---

## Aturan Reproducibility dan Keselamatan

1. **Akun sandbox saja.** Konfigurasi tidak boleh menunjuk akun atau bucket yang berisi data nyata di luar eksperimen.
2. **Tanpa kredensial di repo.** Profil AWS dibaca dari luar repo; `.env.example` hanya contoh nama variabel.
3. **Prediksi sebelum hasil.** Prediksi setiap mutasi dan aturan DI-* dibekukan sebelum dijalankan; perubahan sesudahnya dicatat sebagai eksplorasi.
4. **Run tidak dihapus.** Run yang gagal diarsipkan dengan alasan, lalu diganti run baru dengan `run_id` baru.
5. **Sanitasi sebelum push.** Plan JSON dan hasil API disaring dari ID akun, ARN, dan nama bucket sebelum dipublikasikan.
6. **Pembersihan diverifikasi.** Setiap sel diakhiri dengan pengecekan resource bertag; anomali tagihan dicatat.
7. **Tidak memakai Object Lock compliance** kecuali atas keputusan Supervisor 1.
8. **Paket replikasi Verdet dirujuk, tidak disalin** (tidak berlisensi).

---

## Keputusan Pembimbing yang Berlaku

Diputuskan Supervisor 1, 2026-11-27.

| ID | Keputusan |
|---|---|
| D-01 | Opsi A: penjaga IaC dinilai terhadap kerusakan data nyata |
| D-02 | ID proyek DSIC-002 |
| D-03 | Data = subset 30 WAV korpus DSIC-2606, dengan atribusi |
| D-04 | Tanpa Iceberg; curated berisi manifest/metadata CSV/Parquet sederhana |
| D-05 | Akun AWS sandbox terpisah dengan anggaran dan alarm tagihan |
| D-06 | Dikerjakan mahasiswa; pembimbing menyiapkan pagar akun |
| D-07 | Dua makalah Springer dibaca Hari 1 sebelum RQ dikunci |
| D-08 | AWS Config tidak dipakai sebagai default T4 (hanya subset kecil bila perlu) |
| D-09 | LocalStack di E6, hanya untuk mendokumentasikan perbedaan emulator |

**Masih terbuka:** K-1, yaitu mahasiswa yang mengerjakan DSIC-002.

---

## Referensi Kerja

Status verifikasi per 2026-11-27.

| Rujukan | Status |
|---|---|
| Verdet, A., Hamdaqa, M., Da Silva, L., Khomh, F. (2025). Assessing the adoption of security policies by developers in Terraform across different cloud providers. *Empirical Software Engineering* 30:74. DOI 10.1007/s10664-024-10610-0 | VERIFIED, teks lengkap |
| Hu, H., Bu, Y., Wong, K., Sood, G., Smiley, K., Rahman, A. (2023). Characterizing Static Analysis Alerts for Terraform Manifests: An Experience Report. Pre-print | teks lengkap dibaca; venue PARTIALLY VERIFIED |
| Rahman, A., Parnin, C., Williams, L. (2019). The Seven Sins: Security Smells in Infrastructure as Code Scripts. ICSE 2019. DOI 10.1109/ICSE.2019.00033 | PARTIALLY VERIFIED |
| War, A. et al. (2025). Security smells in infrastructure as code: a taxonomy update beyond the seven sins. arXiv:2509.18761 | VERIFIED (abstrak) |
| Oliveira, W., Paiva, F., Pereira, T. E., Brunet, J. (2025). A Defect Taxonomy for Infrastructure as Code: A Replication Study. arXiv:2505.01568 | VERIFIED (abstrak) |
| Sokolowski, D., Spielmann, D., Salvaneschi, G. (2024). Automated Infrastructure as Code Program Testing. *IEEE TSE* 50(6):1585–1599. DOI 10.1109/TSE.2024.3393070 | VERIFIED (halaman pertama) |
| Spielmann, D. et al. (2026). Metamorphic Testing for Infrastructure-as-Code Engines. OOPSLA 2026. DOI 10.1145/3798226 | VERIFIED (halaman konferensi) |
| Jana, P. et al. (2026). TerraFormer: Automated Infrastructure-as-Code with LLMs Fine-Tuned via Policy-Guided Verifier Feedback. ICSE 2026. arXiv:2601.08734 | VERIFIED (abstrak) |
| Ghosh, S., Shetty, M., Bansal, C., Nath, S. (2022). How to Fight Production Incidents? An Empirical Study on a Large-scale Cloud Service. SoCC '22. DOI 10.1145/3542929.3563482 | VERIFIED (teks PDF) |
| Paket replikasi Verdet: https://github.com/averdet/master-terraform-sec-adoption | diperiksa 2026-11-27; tanpa LICENSE |
| Dua makalah Springer (DOI 10.1007/s44227-026-00114-2; 10.1007/978-3-032-27160-0_37) | NOT VERIFIED |
