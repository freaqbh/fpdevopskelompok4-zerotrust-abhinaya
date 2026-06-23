# State-of-the-Art Analysis: Container Security and Gate Automation
**Eksplorasi Tooling & State of the Art** — Farand Febriansyah (NRP: 5027231084)

---

## 1. Lanskap Keamanan Kontainer Modern

Di era komputasi awan (*cloud-native*), kontainerisasi telah mengubah cara aplikasi dipaketkan dan dijalankan. Namun, container image sering kali membawa ketergantungan OS (seperti package Debian/Alpine) dan library aplikasi (seperti npm, pip, go modules) yang memiliki kerentanan keamanan bawaan. 

Untuk mengamankan rantai pasok perangkat lunak, industri DevOps mengadopsi pemindaian kerentanan otomatis (*Automated Vulnerability Scanning*) yang diintegrasikan langsung ke dalam pipeline CI/CD. Terdapat beberapa alat pemindaian ternama di industri, masing-masing dengan karakteristik unik:

| Dimensi Perbandingan | Aqua Security Trivy | Snyk Container | Anchore Engine | CoreOS Clair |
|---|---|---|---|---|
| **Lisensi / Biaya** | Open-source (Apache-2.0), sepenuhnya gratis. | Freemium (Terbatas untuk akun gratis). | Open-source & Enterprise. | Open-source (Apache-2.0). |
| **Kemudahan Integrasi** | Sangat mudah (single binary, official GitHub Action). | Mudah, membutuhkan registrasi token API. | Cukup rumit (memerlukan setup database PostgreSQL & daemon). | Rumit (arsitektur client-server dengan PostgreSQL terdedikasi). |
| **Kecepatan Scan** | **Sangat Cepat** (pemindaian lokal layer kontainer secara instan). | Cepat (menggunakan SaaS backend untuk analisis). | Lambat (proses indexing layer membutuhkan waktu lama). | Sedang (memerlukan waktu sinkronisasi database database CVE). |
| **Cakupan Pemindaian** | OS Packages + Language Packages + IaC + Secrets + License. | OS Packages + Language Packages + License. | OS Packages + Language Packages. | Fokus utama pada OS Packages saja. |
| **Pembaruan DB** | Otomatis di-update setiap run dari Aqua Security. | Otomatis di-update via Snyk Intel Engine. | Sinkronisasi manual/berkala. | Sinkronisasi berkala dengan database upstream. |

---

## 2. Mengapa Trivy Menjadi Pilihan Terbaik Kelompok Kami?

Berdasarkan analisis perbandingan di atas dan eksperimen mandiri yang kami jalankan, kami memilih **Trivy** sebagai landasan utama *vulnerability scanner* proyek ini dengan pertimbangan:

1.  **Ringan dan Native**: Trivy tidak memerlukan server database terpisah seperti Clair atau Anchore. Proses scanning dapat berjalan secara independen di dalam Virtual Machine pembuat (*runner*) GitHub Actions dengan overhead instalasi mendekati nol.
2.  **Akurasi Database**: Trivy didukung oleh database kerentanan Aqua Security yang diperbarui secara *real-time*. Keberadaan opsi `ignore-unfixed` sangat membantu tim developer untuk mengabaikan kerentanan yang belum memiliki patch resmi dari pemelihara OS, sehingga menghindari kepanikan palsu (*false alerts*).
3.  **Dukungan Komunitas**: Integrasi resmi via `aquasecurity/trivy-action` sangat stabil dan memiliki dokumentasi yang luar biasa lengkap untuk mengembalikan *exit code* tertentu (misalnya `exit-code: 1` saat menemukan kerentanan kritis) yang menjadi prasyarat pembangunan gerbang bersyarat (*conditional gate*).

---

## 3. Otomatisasi Gerbang Deployment (Deployment Gating)

Dalam literatur akademik modern, penempatan gerbang deployment (*deployment gating*) telah bergeser dari model persetujuan manual (*manual approval workflow*) menuju model **Declarative Policy-as-Code (PaC)**. 
*   **Pendekatan Konvensional**: Menggunakan form persetujuan manual di ITSM (seperti Jira) atau otorisasi manual di dashboard GitHub. Kelemahannya adalah lambat dan rentan terhadap kesalahan manusia (*human error*).
*   **Pendekatan State-of-the-Art (Zero Trust)**: Menggunakan kebijakan otomatis berdasarkan hasil scan mesin. Pada tingkat enterprise yang kompleks, hal ini diterapkan menggunakan **OPA (Open Policy Agent)** atau **Kyverno** sebagai admission controller di Kubernetes untuk menolak container image yang tidak memiliki tanda tangan (*signature*) valid dari pipeline.
*   **Penyederhanaan Kelompok Kami**: Untuk skala proyek kelas TaskFlow API, kami menyederhanakan mekanisme gating ini dengan memanfaatkan fitur dependensi antar job di GitHub Actions (`needs: security-scan`). Langkah ini memastikan bahwa deployment tidak akan pernah dipicu kecuali job pemindaian Trivy berhasil diselesaikan tanpa ada *exit code* kesalahan.

---

## 4. Kesimpulan

Dengan mengombinasikan **Trivy** untuk pemindaian kerentanan OS/library dan **GitHub Actions Job Dependencies** sebagai gerbang otomatis, pipeline kami berhasil menerapkan pertahanan keamanan berlapis yang efisien dan selaras dengan standar *state-of-the-art* industri DevOps saat ini.