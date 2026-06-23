# Reading Notes: Paper 2 — Shin dkk., 2025
**Eksplorasi Tooling & State of the Art** — Farand Febriansyah (NRP: 5027231084)

---

## 1. Identitas Paper
*   **Judul**: *Enhancing Cloud-Native DevSecOps: A Zero Trust Approach for the Financial Sector*
*   **Penulis**: Daemin Shin, Jiyoon Kim, I Wayan Adi Juliawan Pawana, Ilsun You
*   **Tahun**: 2025
*   **Publikasi/Venue**: Computer Standards & Interfaces, Volume 93, Elsevier
*   **Indeks**: Scopus & Web of Science (SCIE) (DOI: 10.1016/j.csi.2025.103975)

---

## 2. Latar Belakang & Klaim Utama
Sektor finansial saat ini beramai-ramai melakukan migrasi ke infrastruktur cloud-native berbasis microservices untuk mendukung kelincahan bisnis digital. Namun, peningkatan adopsi ini memperluas permukaan serangan (*attack surface*) karena arsitektur microservices melibatkan banyak komunikasi antar-layanan yang dinamis. 

Sayangnya, organisasi finansial sering kali kekurangan materi acuan dan kerangka kerja praktis untuk mengadopsi model Zero Trust secara mandiri di atas pipeline DevSecOps mereka. Paper ini mengusulkan kerangka kerja (*framework*) evaluasi kebijakan Zero Trust untuk mengisi kekosongan panduan tersebut, dengan fokus mengintegrasikan prinsip verifikasi terus-menerus tanpa asumsi kepercayaan implisit terhadap pengguna, perangkat, maupun entitas internal.

---

## 3. Metodologi Penelitian
Langkah-langkah metodologi yang diambil oleh penulis meliputi:
1.  **Analisis Relasional**: Memetakan hubungan timbal balik antara inisiatif cloud-native, keamanan berbasis kontainer, DevSecOps microservices, dan prinsip inti Zero Trust.
2.  **Analisis Tahap SDLC (Stage-by-Stage)**: Melakukan tinjauan menyeluruh terhadap Software Development Life Cycle (SDLC) untuk mengidentifikasi celah *implicit trust* di setiap titik transisi (misalnya saat serah terima kode dari developer ke build agent, dan dari registry ke deployment).
3.  **Perancangan Framework 7 Domain**: Membagi arsitektur Zero Trust DevSecOps finansial menjadi 7 domain keamanan.
4.  **Evaluasi dengan 9 Kriteria Assessment**: Menilai efektivitas framework yang dirancang menggunakan 9 kriteria pengukuran, kemudian memvalidasinya melalui studi kasus riil pada salah satu ekosistem microservices finansial di Korea Selatan.

---

## 4. Temuan Kunci yang Relevan
*   **Implicit Trust di Sepanjang SDLC**: Kepercayaan implisit (*implicit trust*) tidak hanya terjadi saat kode dikirim ke klaster produksi, melainkan dapat muncul di mana saja sepanjang siklus SDLC jika tidak ada verifikasi.
*   **Konsistensi Kebijakan**: Gate keamanan (seperti vulnerability scan) tidak boleh berdiri sebagai utilitas terisolasi. Gate tersebut harus dirancang sebagai bagian dari rantai kebijakan yang terintegrasi untuk menjamin integritas perangkat lunak dari hulu ke hilir.
*   **Justifikasi Posisi Gate**: Temuan ini memperkuat keputusan Kelompok 4 untuk menempatkan Security Scan sebagai gerbang penentu (*conditional deployment gate*) yang mutlak di antara fase Build dan Deploy.

---

## 5. Keterbatasan & Asumsi
*   **Masalah Aksesibilitas (Paywall)**: Naskah lengkap jurnal ini berada di bawah lisensi Elsevier (ScienceDirect). Karena keterbatasan akses institusi, beberapa detail mengenai rincian dari 7 domain dan 9 kriteria penilaian belum dijabarkan secara rinci pada level teknis operasional.
*   **Spesifik terhadap Regulasi Regional**: Studi kasus pada paper ini sangat patuh pada kepatuhan hukum finansial nasional Korea Selatan, sehingga generalisasinya untuk proyek kelas skala kecil atau lingkungan tim non-finansial perlu disesuaikan agar tidak menambah kompleksitas birokrasi release.

---

## 6. Poin Kritis & Relevansi Proyek
*   **Penyederhanaan Kriteria**: Sembilan kriteria penilaian dalam paper dirancang untuk tingkat kepatuhan regulasi perbankan. Untuk proyek TaskFlow API kami, kriteria tersebut disederhanakan menjadi dua fokus utama: integritas container image (Trivy Scan) dan pencegahan deployment tak sah (Deployment Gate).
*   **Peran di Proyek Kelompok 4**: Menjadi justifikasi konseptual utama dalam `research/03-design-decisions.md` untuk membuktikan bahwa penempatan scan gate kami secara bertahap di sepanjang alur SDLC GitHub Actions adalah keputusan arsitektur yang benar secara teoritis.
