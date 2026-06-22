# Metrics After (Secure Image)

> Bagian ini akan diisi oleh Rafika Az Zahra Kusumastuti — Jobdesk 2: Implementasi Conditional Deployment Gate

## Image yang Digunakan

```text
nginx:latest
```

## Hasil Pipeline

| Stage             | Status    |
| ----------------- | --------- |
| Build Application | Success |
| Security Scan     | Success |
| Deploy            | Success |
| Send Notification | Success |

## Screenshot

* `evaluation/screenshots/after-pipeline-success.png`

## Analisis

Setelah image diganti menjadi `nginx:latest`, proses security scanning berhasil diselesaikan tanpa menemukan vulnerability yang melanggar threshold yang telah ditentukan.

Karena job **Security Scan** berstatus **success**, deployment gate mengizinkan job **Deploy** untuk dijalankan hingga selesai.

## Kesimpulan

Deployment berhasil dijalankan karena image telah lolos proses verifikasi keamanan yang diterapkan pada pipeline Zero Trust CI/CD.

