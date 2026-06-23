# Metrics After — Zero Trust Active (Secure Image) - Aswalia Novitriasari 5027231012

## Tujuan Pengujian
Skenario ini mensimulasikan kondisi pipeline CI/CD setelah prinsip Zero Trust diterapkan. Ketika pipeline dijalankan, setiap artifact container image wajib melewati proses pemindaian keamanan menggunakan Trivy dengan severity threshold HIGH dan CRITICAL. Deployment ke klaster Kubernetes hanya diperbolehkan jika image dinyatakan aman dan lolos verifikasi scanning.

## Setup Pengujian

| Item | Detail |
|---|---|
| Branch pengujian | `main` |
| Image yang diuji | `nginx:latest` (image terbaru yang sudah ditambal/patched) |
| Konfigurasi pipeline | Job `security-scan` aktif; job `deploy` bergantung pada keberhasilan scan (`needs: security-scan`) |
| Tools scanning | Trivy Action (`aquasecurity/trivy-action@master`) |
| Tanggal pengujian | 23 Juni 2026 |

---

## Hasil Eksekusi Pipeline

Pengujian skenario ini didasarkan pada run pipeline riil di GitHub Actions:

### 1. Skenario Pemblokiran Image Rentan (`nginx:1.14`)
*   **Commit ID**: `e75de63` (Implement conditional deployment gate) oleh `anaspl`
*   **Durasi Run**: **53 detik**
*   **Status Job**:
    *   `build`: **Success**
    *   `security-scan`: **Failed** (Trivy mendeteksi kerentanan di atas threshold)
    *   `deploy`: **Skipped** (Diblokir oleh deployment gate)
    *   `notify`: **Failed** (Pipeline mendeteksi kegagalan deploy)
*   **Prevention Rate**: **100%** (Celah keamanan kritis berhasil diblokir sebelum menyentuh klaster produksi).

### 2. Skenario Sukses Image Aman (`nginx:latest`)
*   **Commit ID**: `406a648` (test secure image) oleh `anaspl`
*   **Durasi Run**: **1 menit 4 detik (64 detik)**
*   **Status Job**:
    *   `build`: **Success**
    *   `security-scan`: **Success** (Lolos verifikasi keamanan)
    *   `deploy`: **Success** (Berhasil dideploy ke klaster Kubernetes)
    *   `notify`: **Success** (Pipeline selesai dengan sukses)

---

## Perbandingan Kinerja & Analisis Overhead

Berdasarkan data baseline dari Skenario Before (Baseline - Commit `ae0182b`):

*   **Total Waktu Baseline (Tanpa Scan)**: **22 detik**
*   **Total Waktu Zero Trust (Dengan Scan & Deploy Sukses)**: **64 detik**
*   **Overhead Latency Absolut**: $64s - 22s = \mathbf{42\text{ detik}}$
*   **Persentase Overhead**:
    $$\text{Overhead (\%)} = \left( \frac{42}{22} \right) \times 100\% = \mathbf{190.91\%}$$

### Analisis Latency & Justifikasi
Penambahan durasi dari 22 detik menjadi 64 detik menghasilkan overhead latency sebesar **42 detik (190.91%)**. 

Overhead ini terjadi karena dua alasan utama:
1.  **Inisialisasi & Download Database Trivy**: Pada GitHub Actions runner (bersih/clean VM), Trivy harus mengunduh database kerentanan terbaru (*vulnerability database* sebesar ~100MB+) di setiap *build run*. Proses ini membutuhkan waktu sekitar 30–40 detik.
2.  **Pemindaian File System Image**: Trivy melakukan pemindaian layer-by-layer terhadap base OS dan library package di dalam container image.

Meskipun peningkatan persentase waktu terkesan tinggi secara matematis (~190%), secara praktis durasi absolut **64 detik** masih berada di bawah batas wajar (di bawah 2 menit). Peningkatan waktu ini sangat dapat ditoleransi dibandingkan risiko masuknya image bercelah keamanan kritis ke dalam lingkungan produksi jika menggunakan *implicit trust* (sebagaimana dirujuk dalam penelitian Bhardwaj dkk., 2025).

---

## Prevention Rate

| Metrik | Hasil |
|---|---|
| Kerentanan Kritis/Tinggi terdeteksi (Trivy Pipeline - ignore unfixed) | 0 (Lolos Threshold) |
| Kerentanan Kritis/Tinggi yang lolos ke deploy | 0 |
| **Prevention rate** | **100%** |

Dengan beralih ke image `nginx:latest`, semua kerentanan OS dan pustaka yang tidak memiliki perbaikan (unfixed) diabaikan, dan tidak ditemukan kerentanan dengan tingkat HIGH/CRITICAL yang melanggar threshold keamanan aktif.

---

## Kesimpulan
Penerapan Zero Trust CI/CD Security pada Skenario After membuktikan peningkatan drastis pada aspek keamanan dengan **Prevention Rate 100%** (berhasil mendeteksi dan memblokir image rentan `nginx:1.14` pada run `e75de63`), dengan konsekuensi overhead penambahan waktu sebesar **42 detik** untuk proses verifikasi keamanan berkelanjutan.
