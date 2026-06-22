# Analysis

> Bagian ini akan diisi oleh Rafika Az Zahra Kusumastuti — Jobdesk 2: Implementasi Conditional Deployment Gate

## Perbandingan Sebelum dan Sesudah Implementasi Zero Trust

Project ini mengimplementasikan konsep **Zero Trust CI/CD Security** dengan menambahkan dua komponen utama:

1. **Automated Vulnerability Scanning** menggunakan Trivy.
2. **Conditional Deployment Gate** menggunakan GitHub Actions.

### Skenario 1 – Image Rentan

Pada skenario pertama digunakan image:

```text
nginx:1.14
```

Hasil pemindaian Trivy menunjukkan:

* Total Vulnerabilities: 77
* High: 50
* Critical: 27

Karena ditemukan vulnerability dengan severity **HIGH** dan **CRITICAL**, job **Security Scan** gagal sehingga deployment tidak dijalankan.

Hasil pipeline:

| Stage             | Status  |
| ----------------- | ------- |
| Build Application | Success |
| Security Scan     | Failed  |
| Deploy            | Skipped |

### Skenario 2 – Image Aman

Pada skenario kedua digunakan image:

```text
nginx:latest
```

Hasil pipeline:

| Stage             | Status  |
| ----------------- | ------- |
| Build Application | Success |
| Security Scan     | Success |
| Deploy            | Success |

Karena image berhasil melewati proses verifikasi keamanan, deployment dapat dijalankan hingga selesai.

## Dampak Implementasi

Sebelum implementasi Zero Trust, artefak hasil build dapat langsung melanjutkan ke tahap deployment tanpa verifikasi keamanan tambahan.

Setelah implementasi Zero Trust, setiap artefak wajib melewati proses security scanning terlebih dahulu. Deployment hanya dapat dijalankan apabila hasil pemindaian memenuhi kebijakan keamanan yang telah ditentukan.

Implementasi ini berhasil menunjukkan prinsip:

> **Never Trust, Always Verify**

dengan memastikan bahwa artefak yang mengandung vulnerability tidak dapat mencapai tahap deployment.

## Kesimpulan

Berdasarkan hasil pengujian, kombinasi **Trivy Security Scan** dan **Conditional Deployment Gate** berhasil meningkatkan keamanan pipeline CI/CD dengan mencegah artefak yang memiliki vulnerability kategori **HIGH** maupun **CRITICAL** masuk ke tahap deployment.
