# Metrics Before (Vulnerable Image)

> Bagian ini akan diisi oleh Rafika Az Zahra Kusumastuti — Jobdesk 2: Implementasi Conditional Deployment Gate

## Image yang Digunakan

```text
nginx:1.14
```

## Hasil Pipeline

| Stage             | Status     |
| ----------------- | ---------- |
| Build Application | Success  |
| Security Scan     | Failed   |
| Deploy            | Skipped |
| Send Notification | Failed   |

## Hasil Security Scan (Trivy)

| Severity              | Jumlah |
| --------------------- | ------ |
| Critical              | 27     |
| High                  | 50     |
| Total Vulnerabilities | 77     |

## Screenshot

* `evaluation/screenshots/before-pipeline-failed.png`
* `evaluation/screenshots/before-trivy-report.png`

## Analisis

Trivy berhasil mendeteksi **77 vulnerability** pada image `nginx:1.14`, yang terdiri dari **50 vulnerability kategori HIGH** dan **27 vulnerability kategori CRITICAL**.

Karena konfigurasi pipeline menggunakan:

```yaml
severity: CRITICAL,HIGH
exit-code: 1
```

maka job **Security Scan** otomatis gagal. Akibatnya, job **Deploy** tidak dijalankan karena deployment gate mengharuskan security scan berhasil terlebih dahulu.

## Kesimpulan

Deployment berhasil diblokir sebelum mencapai tahap deployment karena image yang digunakan mengandung vulnerability dengan tingkat keparahan **HIGH** dan **CRITICAL**.

