# Design Decisions: Security Scanning

> Bagian ini akan diisi oleh Hasan — Jobdesk 1: Setup & Implementasi Security Scan Job

Implementasi pemindaian keamanan (*security scanning*) pada pipeline CI/CD proyek ini menggunakan **Trivy**. Pemilihan Trivy didasarkan pada kapabilitasnya sebagai *tool open-source* yang ringan dan dirancang secara *native* untuk *container image scanning*. 

Pada tahap pemindaian ini, *severity threshold* ditetapkan secara ketat pada tingkat **CRITICAL** dan **HIGH**. Jika Trivy mendeteksi kerentanan pada tingkat tersebut, parameter `exit-code: '1'` akan membuat *pipeline* otomatis gagal (*failed status*). 

Keputusan desain ini merupakan perwujudan langsung dari prinsip *Zero Trust* ("*never trust, always verify*") yang dikemukakan oleh **Bhardwaj dkk. (2025)**. Berdasarkan referensi tersebut, *pipeline* konvensional dengan asumsi *implicit trust* sangat berisiko terhadap injeksi *malware* atau *dependency hijacking*. Pemblokiran otomatis ini adalah bentuk verifikasi keamanan berkelanjutan untuk memastikan tidak ada *image* berisiko tinggi yang lolos ke tahap *deployment*.

# Design Decisions: Conditional Deployment

> Bagian ini akan diisi oleh Rafika Az Zahra Kusumastuti — Jobdesk 2: Implementasi Conditional Deployment Gate

Setelah tahap *security scanning* selesai, pipeline CI/CD perlu memastikan bahwa hanya artefak yang telah lolos verifikasi keamanan yang dapat melanjutkan ke proses *deployment*. Untuk memenuhi kebutuhan tersebut, diterapkan mekanisme **Conditional Deployment Gate** menggunakan fitur `needs` pada GitHub Actions.

Implementasi dilakukan dengan menambahkan dependensi pada job deployment sebagai berikut:

deploy:
  needs: security-scan

Konfigurasi tersebut memastikan bahwa job `deploy` hanya akan dijalankan apabila job `security-scan` berhasil diselesaikan dengan status **success**. Sebaliknya, apabila proses pemindaian keamanan gagal, proses deployment akan otomatis dihentikan.

Keputusan desain ini mengadopsi prinsip **Zero Trust** yaitu *"Never Trust, Always Verify"*. Artefak yang berhasil melalui proses build tidak langsung dipercaya untuk masuk ke tahap deployment, melainkan harus melewati proses verifikasi keamanan terlebih dahulu.

Penempatan deployment gate setelah tahap security scanning juga sejalan dengan konsep yang dijelaskan oleh **Shin dkk. (2025)**, yaitu bahwa mekanisme verifikasi perlu diterapkan secara bertahap di sepanjang *Software Development Life Cycle (SDLC)*. Pada implementasi proyek ini, tahap *security scanning* berfungsi sebagai titik verifikasi yang menentukan apakah artefak layak untuk dideploy.

Untuk membuktikan bahwa deployment gate bekerja sesuai tujuan, dilakukan dua skenario pengujian. Pada skenario pertama digunakan image rentan `nginx:1.14`. Hasil pemindaian Trivy menunjukkan 77 kerentanan yang terdiri dari 50 kerentanan kategori HIGH dan 27 kerentanan kategori CRITICAL. Karena ditemukan kerentanan dengan tingkat keparahan tinggi, job `security-scan` gagal dan job `deploy` tidak dijalankan.

Pada skenario kedua digunakan image yang lebih aman sehingga proses security scanning berhasil diselesaikan. Karena job `security-scan` berstatus success, job `deploy` dapat dijalankan hingga selesai.

Hasil pengujian tersebut menunjukkan bahwa mekanisme Conditional Deployment Gate berhasil mencegah artefak yang memiliki kerentanan tinggi masuk ke tahap deployment, sekaligus tetap memungkinkan deployment berjalan secara otomatis ketika artefak telah lolos verifikasi keamanan.