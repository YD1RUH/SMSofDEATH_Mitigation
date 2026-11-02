# 🛡️ SMSofDeath Mitigation Tool

![Platform](https://img.shields.io/badge/Platform-Windows%2010%2F11-0078D6?logo=windows&logoColor=white)
![Language](https://img.shields.io/badge/C%2B%2B-17-00599C?logo=cplusplus&logoColor=white)
![GUI](https://img.shields.io/badge/GUI-Dear%20ImGui-18BFFF)
![Graphics](https://img.shields.io/badge/Render-DirectX%2011-107C10?logo=microsoft&logoColor=white)
![Android](https://img.shields.io/badge/Android-ADB-3DDC84?logo=android&logoColor=white)
![CVE](https://img.shields.io/badge/CVE-2026--58882-red)
![Type](https://img.shields.io/badge/Type-Defensive%20Tool-blue)
![Standalone](https://img.shields.io/badge/EXE-Standalone%20(no%20DLL)-success)

Aplikasi Windows berbasis GUI untuk **mendeteksi, memitigasi, dan memulihkan** perangkat Android dari serangan **SMS of Death** (**CVE-2026-58882**), memanfaatkan **ADB (Android Debug Bridge)**.

Referensi resmi: <https://smsofdeath.com/>

---

## 📖 Apa itu SMS of Death? (Penjelasan untuk Orang Awam)

**SMS of Death** adalah celah keamanan pada sistem Android. Seseorang dapat mengirim **satu pesan khusus yang sudah dirusak sengaja** ke nomor HP Anda, dan itu bisa membuat bagian telepon di HP Anda **mati mendadak**.

### Apa yang membuatnya berbahaya?

- **Tanpa interaksi sama sekali (zero-click).** Anda tidak perlu membuka pesan, mengklik tautan, atau memasang aplikasi apa pun. Pesan cukup *sampai* ke HP Anda.
- **Pesannya tidak terlihat.** Pesan ini bukan SMS biasa yang berisi teks. Ia adalah pesan data teknis di belakang layar, jadi Anda mungkin tidak melihat notifikasi apa pun.
- **Panggilan bisa terputus**, termasuk **panggilan darurat** (112/911/110) yang sedang berlangsung.
- **HP bisa terjebak reboot loop.** Pesan berbahaya tersimpan di memori pesan, lalu diproses ulang setiap HP menyala. Akibatnya HP crash lagi dan restart berulang-ulang, bahkan bisa berujung ke reset pabrik otomatis oleh sistem.

### Mengapa ini bisa terjadi? (Versi sederhana)

Bayangkan petugas pos yang menerima paket bertuliskan "isi: **minus 2 miliar** barang". Petugas itu mencoba menyiapkan kotak dengan ukuran yang mustahil, lalu **panik dan berhenti bekerja**. Tidak ada pengaman yang menangkap kesalahan itu.

Di Android, komponen yang membaca pesan jenis **WAP Push** menerima angka ukuran yang dimanipulasi sehingga berubah menjadi angka negatif. Android lalu mencoba membuat ruang memori dengan ukuran negatif, lalu muncul error `NegativeArraySizeException`. Error ini tidak ditangani, sehingga proses telepon (`com.android.phone`) ikut mati.

### Siapa yang terdampak?

| Versi Android | API | Status |
|---|---|---|
| Android 11 – 15+ | 30 – 35+ | Rentan sebelum patch |
| Android 14 & 15 | 34, 35 | **Sudah ada patch** (Security Patch **Oktober 2026** atau lebih baru) |
| Android 12 & 13 | 32, 33 | ⚠️ **Tidak mendapat patch dari Google**, tetap rentan |

> Patch AOSP: commit `735d967` — *"Fix remote DoS in decodeWapPdu"*.

Tool ini membantu Anda menutup celah tersebut **tanpa menunggu update dari pabrikan HP**.

---

## ✨ Fitur

| Tab | Fungsi |
|---|---|
| **Devices** | Menampilkan daftar perangkat ADB dan info singkat perangkat |
| **Assessment** | Mengecek kerentanan: versi Android, tanggal patch keamanan, status WAP Push, status proses telepon, lalu menghitung tingkat risiko |
| **Mitigate** | Menonaktifkan WAP Push (mitigasi utama) dan memulihkan proses `com.android.phone` |
| **Reboot Loop Fix** | Menghapus pesan berbahaya dari antrean SMS mentah untuk HP yang terjebak reboot loop |
| **Monitor** | Memantau logcat secara real-time dengan sorotan otomatis jika ada tanda serangan |
| **Log** | Riwayat semua aksi dan perintah yang dijalankan aplikasi |

---

## 📦 Kebutuhan

- Windows 10 / 11 (64-bit)
- HP Android dengan kabel USB
- **`adb.exe`** (Android Platform-Tools)
  - Unduh: <https://developer.android.com/studio/releases/platform-tools>
  - Letakkan `adb.exe` (beserta `AdbWinApi.dll` dan `AdbWinUsbApi.dll` bawaan paketnya) **di folder yang sama dengan `SMSofDeathMitigation.exe`**, atau daftarkan ke PATH Windows.
- Aplikasi sudah *standalone*: tidak perlu menginstal Visual C++ Redistributable atau DLL tambahan.

---

## 🚀 Persiapan Sebelum Digunakan

### Aktifkan USB Debugging di HP

1. Buka **Pengaturan → Tentang Ponsel**.
2. Ketuk **Nomor Build** sebanyak 7 kali sampai muncul pesan "Anda sekarang adalah developer".
3. Kembali ke **Pengaturan → Opsi Developer** (lokasi bisa berbeda tiap merek).
4. Aktifkan **USB Debugging**.

### Hubungkan HP ke PC

1. Sambungkan HP dengan kabel USB. Gunakan kabel yang mendukung transfer data, bukan kabel khusus pengisi daya.
2. Jika muncul dialog **"Allow USB debugging?"** di HP, centang *Always allow from this computer*, lalu ketuk **Allow**.

### Jalankan Aplikasi

Jalankan `SMSofDeathMitigation.exe`. Tidak perlu hak Administrator.

Pada tab **Devices**, bagian *adb.exe path* harus menampilkan **[OK]** berwarna hijau. Jika **[NOT FOUND]**, isi path lengkap ke `adb.exe` lalu tekan **Enter**.

---

## 🧭 Panduan Penggunaan

### Langkah Cepat (HP Normal)

1. Tab **Devices** → klik **Refresh Device List** → pilih HP Anda (status harus `[device]`).
2. Tab **Assessment** → klik **Run Assessment**.
3. Lihat hasil **Risk**:
   - 🔴 **CRITICAL / HIGH** → lanjut ke langkah 4.
   - 🟡 **MEDIUM** → disarankan tetap melakukan mitigasi.
   - 🟢 **LOW** → HP relatif aman.
4. Tab **Mitigate** → klik **DISABLE WAP Push [Recommended]**.
5. Jalankan **Run Assessment** sekali lagi untuk memastikan status WAP Push sudah `DISABLED (protected)`.

---

### Tab 1: Devices

- **Refresh Device List**: memuat ulang perangkat yang terhubung.
- Status perangkat:
  - `device`: siap digunakan ✅
  - `unauthorized`: izinkan debugging pada layar HP
  - `offline`: cabut dan colok ulang kabel
- Setelah Assessment dijalankan, info HP (merek, model, versi Android, patch, status WAP Push, tingkat risiko) tampil di bagian bawah.

### Tab 2: Assessment

Klik **Run Assessment** untuk melakukan 4 pemeriksaan:

| Pemeriksaan | Arti |
|---|---|
| **Android Version** | Apakah versi Android masuk rentang yang terdampak |
| **Security Patch** | Apakah patch keamanan HP sudah Oktober 2026 atau lebih baru |
| **WAP Push** | Apakah fitur WAP Push masih aktif (jalur serangan) |
| **com.android.phone** | Apakah proses telepon sedang berjalan normal |

Label hasil: `PASS` (aman), `FAIL` (rentan), `WARN` (perlu perhatian), `INFO` (informasi).

> ⚠️ **Catatan penting:** HP Android 12/13 akan tetap ditandai rentan walaupun tanggal patch-nya baru, karena versi tersebut memang tidak menerima perbaikan ini.

### Tab 3: Mitigate

**Mitigasi utama: Disable WAP Push**
- Klik **DISABLE WAP Push [Recommended]** untuk menutup jalur serangan.
- Klik **Re-enable WAP Push** jika ingin mengaktifkan kembali.
- **Efek samping:** pesan konfigurasi operator via WAP Push (misalnya setelan APN otomatis) tidak akan diterima. Pada HP modern ini jarang dibutuhkan, dan MMS biasanya tetap berfungsi karena APN sudah terkonfigurasi.

**Pemulihan proses telepon**
- **Check PID**: mengecek apakah `com.android.phone` berjalan.
- **Force-stop (triggers restart)**: menghentikan paksa proses telepon agar Android menjalankannya ulang. Gunakan jika sinyal hilang setelah serangan.

### Tab 4: Reboot Loop Fix 🚨

Gunakan **hanya jika HP Anda restart terus-menerus** (kira-kira setiap 90 detik) setelah diserang.

Anda punya jendela waktu sempit, sekitar **30–60 detik setelah HP selesai booting**. Siapkan aplikasi dan HP sudah terhubung sebelum mulai.

| Langkah | Aksi |
|---|---|
| **Step 1** | Klik **Check boot_completed**. Nilai `1` berarti HP sudah selesai booting, segera lanjut. |
| **Step 2** | Klik **Query content://sms/raw** untuk melihat daftar pesan mentah. Cari baris dengan `destination_port=134020` dan catat kolom `address` (pengirim). |
| **Step 3** | Klik **Grant WRITE_SMS to shell** agar ADB boleh menghapus data pesan. |
| **Step 4** | Isi **Sender address** dari Step 2 lalu klik **Delete Entry**. Jika dikosongkan, **semua** pesan WAP Push (`destination_port=134020`) akan dihapus. |
| **Step 5** | Klik **Revoke WRITE_SMS from shell** untuk mengembalikan izin ke pengaturan semula. |

Jika sebelum Step 4 HP sudah restart, ulangi dari Step 1 pada siklus boot berikutnya.

Setelah berhasil, **langsung lakukan mitigasi** di tab **Mitigate** (Disable WAP Push) agar tidak terkena lagi.

**⛔ Opsi terakhir: CLEAR com.android.providers.telephony**
Menghapus **seluruh SMS/MMS dan pengaturan APN/operator** pada HP. Pakai hanya jika Step 1–5 tidak berhasil. Pencadangan pesan tidak mungkin dilakukan saat HP dalam reboot loop.

### Tab 5: Monitor

Memantau log sistem HP secara langsung untuk mendeteksi serangan.

- **Start Monitor / Stop Monitor**: memulai atau menghentikan pemantauan.
- **Alerts only**: hanya menampilkan baris yang mencurigakan.
- **Auto-scroll**: otomatis menggulir ke log terbaru.
- **Filter**: menyaring log berdasarkan kata kunci.
- Baris berwarna **merah** menandakan indikator serangan, antara lain: `NegativeArraySizeException`, `WapPushOverSms`, `decodeWapPdu`, `InboundSmsHandler`, `FATAL EXCEPTION`, `RescueParty`.
- Banner merah `!!! ... INDICATOR(S) DETECTED !!!` muncul jika ada tanda serangan terdeteksi.

### Tab 6: Log

Riwayat aksi aplikasi (refresh perangkat, assessment, mitigasi, dan lainnya) lengkap dengan waktu. Berguna untuk dokumentasi atau pelaporan.

---

## 💡 Tips Keamanan Tambahan

- Perbarui Android ke patch **Oktober 2026 atau lebih baru** begitu pabrikan HP merilisnya.
- Pengguna **Android 12/13**: patch kemungkinan tidak akan datang. Biarkan WAP Push **tetap nonaktif**.
- Di lingkungan berisiko tinggi, siapkan perangkat cadangan khusus untuk panggilan darurat.
- Perintah manual tanpa aplikasi ini:
  ```
  adb shell settings put global wap_push_enabled 0
  ```

---

## ❓ Pertanyaan Umum (FAQ)

**Apakah HP saya harus di-root?**
Tidak. Semua langkah berjalan lewat ADB biasa.

**Apakah data saya aman?**
Mitigasi WAP Push dan pemantauan tidak menghapus data. Hanya tombol **Delete Entry** (pesan WAP Push mentah) dan **CLEAR telephony** (destruktif) yang menghapus data.

**Apakah tool ini bisa memperbaiki celah secara permanen?**
Tidak. Tool ini **menutup jalur serangan** (WAP Push). Perbaikan permanen hanya lewat update keamanan resmi. Untuk Android 12/13, menonaktifkan WAP Push adalah pertahanan terbaik yang tersedia.

**`[NOT FOUND]` pada ADB path?**
Pastikan `adb.exe` berada di folder yang sama dengan aplikasi atau isi path lengkapnya, lalu tekan **Enter**.

**HP tidak muncul di daftar?**
Periksa kabel data, aktifkan USB Debugging, izinkan dialog debugging di HP, lalu klik **Refresh Device List**.

---

## ⚠️ Disclaimer

Tool ini dibuat untuk tujuan **defensif**: melindungi dan memulihkan perangkat milik Anda sendiri. Gunakan hanya pada perangkat yang Anda miliki atau yang Anda punya izin untuk mengelolanya. Tool ini tidak menjamin perlindungan penuh dan tidak menggantikan pembaruan keamanan resmi. Penulis tidak bertanggung jawab atas kehilangan data akibat penggunaan fitur destruktif.

---

## 🧩 Teknologi

- **C++17**
- **Dear ImGui** (Win32 + DirectX 11 backend)
- **Android Debug Bridge (ADB)**

## 🔗 Referensi

- Situs penjelasan: <https://smsofdeath.com/>
- CVE: **CVE-2026-58882**
- Patch AOSP: commit `735d967` — *Fix remote DoS in decodeWapPdu*
- Android Platform-Tools: <https://developer.android.com/studio/releases/platform-tools>
