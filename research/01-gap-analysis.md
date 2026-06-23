# Gap Analysis: Implicit Trust vs. Zero Trust CI/CD Security
**README & Reproducibility Validation** — Nisrina Atiqah Dwiputri Ridzki (NRP: 5027231075)

---

## 1. Pendahuluan

Di era modern rekayasa perangkat lunak, pipeline Integrasi Berkelanjutan dan Pengiriman Berkelanjutan (CI/CD) telah menjadi tulang punggung pengiriman aplikasi secara cepat dan otomatis. Namun, kecepatan ini sering kali tidak diimbangi dengan mekanisme keamanan yang memadai. Proyek TaskFlow API pada modul-modul DevOps sebelumnya telah berhasil menerapkan otomatisasi build dan deployment menggunakan GitHub Actions. Meskipun demikian, pipeline tersebut masih memiliki celah keamanan serius yang kami identifikasi sebagai **Implicit Trust Gap**.

---

## 2. Definisi Masalah: Celah Keamanan Tradisional (Before)

Sebelum diterapkannya pembaruan keamanan, pipeline TaskFlow API beroperasi di bawah asumsi **Implicit Trust** (Kepercayaan Implisit). Karakteristik dari pipeline lama ini meliputi:

1.  **Build to Deploy Tanpa Verifikasi**: Setelah kode aplikasi di-push dan container image berhasil di-build, pipeline langsung melanjutkan ke proses deploy ke klaster Kubernetes tanpa adanya verifikasi keamanan di antaranya.
2.  **Perimeter-Based Security Fallacy**: Pipeline mengasumsikan bahwa selama aksi tersebut dipicu oleh akun GitHub tim yang sah, maka seluruh kode, dependensi, dan artifact container yang dihasilkan otomatis bersifat aman untuk dijalankan.

### Mengapa Pendekatan Ini Berbahaya?
Seperti yang dianalisis oleh **Bhardwaj dkk. (2025)**, serangan terhadap rantai pasok perangkat lunak (*software supply chain attacks*) terus meningkat. Model keamanan berbasis perimeter konvensional sangat rentan terhadap:
*   **Credential Theft / Compromised Secrets**: Kredensial runner CI/CD atau API key klaster Kubernetes yang dicuri dapat digunakan oleh pihak tak bertanggung jawab untuk memicu deployment liar.
*   **Dependency Hijacking / Poisoning**: Kode aplikasi mungkin menggunakan library eksternal yang tanpa disadari telah disusupi oleh celah keamanan kritis (*vulnerable library*).
*   **Build Tampering**: Artefak container yang dihasilkan bisa saja diubah secara jahat di tengah jalan saat dikirim ke registry jika integritasnya tidak divalidasi.

---

## 3. Gap Analysis Matrix

Berikut adalah perbandingan mendalam antara kondisi pipeline TaskFlow API sebelum dan setelah perbaikan berdasarkan prinsip Zero Trust:

| Dimensi Analisis | Pipeline Konvensional (Before) | Pipeline Zero Trust (After) | Gap Keamanan yang Diselesaikan |
|---|---|---|---|
| **Asumsi Kepercayaan** | *Implicit Trust* (Dipercaya begitu berhasil masuk pipeline). | *Zero Trust* ("Never Trust, Always Verify"). | Menghilangkan kepercayaan buta terhadap artifact internal. |
| **Vulnerability Scanning** | Tidak ada pemindaian otomatis di sepanjang pipeline. | Pemindaian otomatis menggunakan Trivy terintegrasi. | Mencegah masuknya image yang memiliki celah keamanan tinggi/kritis. |
| **Deployment Gate** | Kode langsung dikirim ke klaster produksi segera setelah build sukses. | Deployment terkunci (`needs: security-scan`) dan hanya jalan jika scan sukses. | Menolak deployment otomatis terhadap image yang rentan. |
| **Mitigasi Supply Chain** | Rentan terhadap eksploitasi celah OS dan pustaka pihak ketiga. | Memastikan pustaka dan OS image selalu diperbarui dan aman dari celah kritis. | Meminimalisasi permukaan serangan (*attack surface mitigation*). |
| **Prevention Rate** | **0%** (Image rentan `nginx:1.14` lolos sepenuhnya). | **100%** (Image rentan berhasil diblokir sebelum deploy). | Menghilangkan lolosnya kerentanan kritis ke cluster. |

---

## 4. Landasan Teoretis dari Paper Referensi

Keputusan untuk menutupi celah keamanan ini berlandaskan pada temuan akademis utama kami:
*   **Bhardwaj dkk. (2025)** menekankan bahwa perlindungan terbaik untuk integritas perangkat lunak adalah melalui **Continuous Verification** (Verifikasi Berkelanjutan). Setiap artifact harus diperiksa kerentanannya di setiap transisi fase pipeline secara otomatis.
*   **Shin dkk. (2025)** menyimpulkan bahwa celah *implicit trust* di sepanjang siklus hidup SDLC hanya bisa dihilangkan dengan menempatkan gerbang keamanan bertahap. Gerbang ini harus bertindak sebagai *quality gate* yang mencegah kode cacat/rentan berpindah ke lingkungan operasional klaster.

---

## 5. Kesimpulan

Tanpa adanya pembaharuan Zero Trust ini, pipeline TaskFlow API konvensional membiarkan image kontainer seperti `nginx:1.14` yang membawa **113 kerentanan** (31 Critical dan 82 High) lolos masuk ke klaster produksi hanya dalam waktu 22 detik. Implementasi pemindaian keamanan Trivy dan Conditional Deployment Gate adalah solusi mutlak untuk menutup gap keamanan ini, memastikan bahwa setiap rilis aplikasi ke klaster Kubernetes telah terbukti aman dari kerentanan kategori High dan Critical.