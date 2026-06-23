# Reading Notes: Paper 1 — Bhardwaj dkk., 2025
**Eksplorasi Tooling & State of the Art** — Farand Febriansyah (NRP: 5027231084)

---

## 1. Identitas Paper
*   **Judul**: *Zero Trust CI/CD Pipeline: A Blueprint for Secure Software Delivery in Modern DevSecOps*
*   **Penulis**: Avdhesh Kumar Bhardwaj, Praveen Anugula, Shubhankar Shilpi, Piyush Ranjan
*   **Tahun**: 2025
*   **Publikasi/Venue**: 1st IEEE Uttar Pradesh Section Women in Engineering International Conference (UPWIECON 2025)
*   **Indeks**: IEEE Xplore (DOI: 10.1109/UPWIECON67212.2025.11390387)

---

## 2. Klaim Utama & Latar Belakang
Pipeline CI/CD konvensional yang ada saat ini mengandalkan asumsi kepercayaan berbasis perimeter (*perimeter-based trust*). Begitu sebuah entitas (seperti developer atau build agent) berada di dalam jaringan internal, mereka langsung dipercaya penuh (*implicit trust*). Asumsi ini membuat pipeline rentan terhadap serangan beruntun seperti pencurian kredensial, peracunan dependensi (*dependency poisoning*), dan perusakan hasil build (*build tampering*). 

Paper ini mengusulkan cetak biru (*blueprint*) arsitektur CI/CD berbasis **Zero Trust** dengan prinsip utama **"Never Trust, Always Verify"** yang diimplementasikan di setiap fase pengiriman perangkat lunak, bukan hanya di level jaringan atau infrastruktur.

---

## 3. Metodologi Penelitian
Penulis membagi kerangka kerja Zero Trust CI/CD menjadi tiga pilar utama:
1.  **Identity & Access Control Enforcement**: Menerapkan RBAC (Role-Based Access Control) dan ABAC (Attribute-Based Access Control) untuk membatasi hak akses paling minim (*least privilege*). Hubungan ini direpresentasikan dalam matriks formal $A(u, r, p)$ yang menentukan hak pengguna $u$, peran $r$, dan hak istimewa $p$. Selain MFA untuk user manusia, verifikasi identitas mesin dinamis digunakan untuk build agents otomatis.
2.  **Secure Artifact Generation and Verification**: Setiap build artifact ditandatangani secara kriptografis menggunakan kunci privat ($S = \text{Sign}(K_{\text{priv}}, H(B))$) dan disertai manifestasi komponen SBOM (*Software Bill of Materials*). Sebelum deployment, tanda tangan diverifikasi menggunakan kunci publik ($K_{\text{pub}}$) untuk memastikan integritas artifact. Jika verifikasi gagal, deployment ditolak.
3.  **Continuous Runtime Threat Monitoring**: Menggunakan SIEM (Security Information and Event Management) dan SOAR (Security Orchestration, Automation, and Response) untuk mendeteksi anomali perilaku sistem menggunakan model deteksi ringan dengan skor anomali $A_s = M(X)$. Jika $A_s > \theta$ (threshold), alert diaktifkan dan sistem otomatis melakukan rollback.

Ketiga pilar ini disatukan dalam **Algorithm 1 (Zero Trust CI/CD Enforcement Workflow)** yang mencakup autentikasi -> otorisasi akses -> tanda tangan artifact -> verifikasi signature -> scan kerentanan -> deploy -> runtime monitoring -> rollback otomatis jika anomali melanggar threshold.

---

## 4. Temuan Kunci & Hasil Eksperimen
Penulis menguji model ini menggunakan toolchain: **GitLab CI, HashiCorp Vault, OPA (Open Policy Agent), Trivy, Istio, dan Falco** dengan hasil:
*   **Mitigasi Serangan**: Tingkat pencegahan serangan (*attack prevention rate*) naik dari **50%** pada pipeline konvensional menjadi **100%** terhadap ancaman pencurian kredensial, dependency hijacking, dan modifikasi artifact.
*   **Overhead Kinerja**: Waktu eksekusi bertambah sangat minimal: Build (+6.7%, dari 120s menjadi 128s), Test (+5.5%, dari 90s menjadi 95s), dan Deploy (+5%, dari 60s menjadi 63s).
*   **Deteksi Anomali**: Model deteksi mendapati *true positive rate* sebesar 96.2% dan *false positive rate* sebesar 3.1%, dengan latency deteksi rata-rata 2.5 detik.

---

## 5. Asumsi & Keterbatasan Paper
*   **Lingkungan Terkontrol**: Eksperimen dijalankan pada klaster cloud-native lokal buatan yang disimulasikan serangannya, bukan di klaster produksi enterprise berskala besar yang memiliki kompleksitas jaringan riil.
*   **Fokus yang Belum Dieksplorasi**: Penulis mengakui bahwa sistem prediksi ancaman berbasis kecerdasan buatan (AI-driven threat prediction) dan manajemen SBOM otomatis secara dinamis masih berupa wacana pengembangan di masa depan (*future work*) dan belum dibahas secara mendalam di paper ini.

---

## 6. Poin Kritis & Relevansi Proyek
*   **Perbedaan Toolchain**: Klaster dan pipeline kelompok kami (menggunakan GitHub Actions, tanpa Vault, Istio, atau Falco) lebih sederhana dibanding toolchain paper. Namun, prinsip dasar continuous scanning (menggunakan Trivy) dan conditional gate (menggunakan dependensi `needs` pada GitHub Actions) berhasil diimplementasikan penuh.
*   **Peran di Proyek Kelompok 4**: Paper ini menjadi landasan teori utama untuk menambahkan job `security-scan` dan mekanisme deployment gate, serta memandu kami dalam memformat metrik evaluasi perbandingan sebelum-sesudah.
