# Panduan Setup: Kelola Materi Sepenuhnya dari Google Sheet

Setelah setup ini selesai (sekali saja), workflow harian Anda jadi:

- **Materi baru** → upload file HTML-nya ke folder `modules/` di GitHub, lalu tambah **1 baris** di Google Sheet. Tidak ada file kode lain yang perlu diedit.
- **Sembunyikan/tampilkan materi** → centang/hilangkan centang di Sheet.
- **Lihat hasil kuis siswa** → buka tab HasilKuis di Sheet, terisi otomatis.

## 1. Buat Google Sheet dengan 2 tab

Buat Spreadsheet baru, nama bebas misalnya **"Data Portal Informatika XI"**. Buat 2 tab:

### Tab "Materi" — pengganti penuh modules.json

Header di baris pertama, urutan kolom **harus sama persis** seperti ini:

| ID | Judul | Deskripsi | Kategori | Semester | Icon | Gambar | File | Tanggal | Tampilkan |
|---|---|---|---|---|---|---|---|---|---|
| INF-101 | Dekomposisi Masalah | Simulasi memecah masalah besar jadi langkah kecil. | Berpikir Komputasional | 1 | 🧩 | | modules/decomposisi-masalah.html | 2026-07-15 | TRUE |
| INF-102 | Kuis Algoritma Dasar | Latihan soal urutan langkah & flowchart. | Algoritma & Pemrograman | 1 | 🧠 | https://i.imgur.com/xxxxx.jpg | modules/kuis-algoritma-dasar.html | 2026-07-22 | TRUE |
| INF-202 | Kuis Jaringan Komputer | Kuis topologi jaringan & internet. | Jaringan Komputer | 2 | 🌐 | images/jaringan.jpg | modules/kuis-jaringan-komputer.html | 2026-08-09 | FALSE |

Penjelasan kolom:
- **ID** — kode unik bebas, misal `INF-103`. Harus sama dengan `MODUL_ID` yang ditulis di dalam file HTML materi (lihat langkah 4).
- **Semester** — isi `1` atau `2` saja.
- **Icon** — satu emoji, tampil di label bawah gambar (boleh dikosongkan, nanti default 📄).
- **Gambar** — **boleh dikosongkan**. Kalau kosong, kartu akan otomatis diberi warna pastel + ikon sebagai gambar. Kalau diisi, ada 2 cara:
  - **Tempel link gambar langsung** dari internet (misal dari [imgur.com](https://imgur.com), Unsplash, atau link "Get link" Google Drive yang sudah diubah ke format direct-view).
  - **Atau** upload gambar ke folder baru bernama `images/` di repo GitHub Anda, lalu tulis path-nya, misal `images/jaringan.jpg`.
- **File** — path relatif ke file HTML-nya, **persis** seperti nama file yang Anda upload ke folder `modules/`.
- **Tanggal** — bebas format, hanya untuk catatan Anda sendiri (belum ditampilkan di portal).
- **Tampilkan** — blok kolom ini → **Insert → Checkbox**. Centang = materi tampil & bisa dibuka siswa.

### Tab "HasilKuis"
Header saja, terisi otomatis setiap siswa (atau kelompok) submit kuis:

| Waktu | Mode | Nama | NIS | No Absen | Kelas | Anggota Kelompok | Materi | Skor | Total |
|---|---|---|---|---|---|---|---|---|---|

Penjelasan:
- **Mode** — otomatis terisi `Individu` atau `Kelompok`, tergantung pilihan siswa saat mulai kuis.
- **Nama** — nama siswa (mode individu) atau nama kelompok (mode kelompok).
- **NIS** dan **No Absen** — hanya terisi untuk mode individu, kosong untuk mode kelompok.
- **Anggota Kelompok** — daftar nama anggota (dipisah koma), hanya terisi untuk mode kelompok.
- **Kelas** — selalu diminta, di kedua mode.

Dengan susunan ini Anda bisa langsung **Sort/Filter** di Sheet berdasarkan Kelas, Mode, atau Materi untuk merekap nilai.

## 2. Pasang Apps Script

Di Spreadsheet: **Extensions → Apps Script**. Hapus isi default, ganti dengan:

```javascript
function doGet(e) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName("Materi");
  const rows = sheet.getDataRange().getValues();
  const header = rows.shift();

  const idx = {};
  header.forEach((h, i) => idx[h.trim()] = i);

  const data = rows
    .filter(r => r[idx["ID"]] !== "")
    .map(r => ({
      id: String(r[idx["ID"]]).trim(),
      title: r[idx["Judul"]],
      description: r[idx["Deskripsi"]],
      category: r[idx["Kategori"]],
      semester: String(r[idx["Semester"]]).trim(),
      icon: r[idx["Icon"]] || "📄",
      image: r[idx["Gambar"]] || "",
      file: r[idx["File"]],
      tanggal: r[idx["Tanggal"]] ? String(r[idx["Tanggal"]]) : "",
      tampilkan: r[idx["Tampilkan"]] === true
    }));

  return ContentService.createTextOutput(JSON.stringify(data))
    .setMimeType(ContentService.MimeType.JSON);
}

function doPost(e) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName("HasilKuis");
  const data = JSON.parse(e.postData.contents);
  sheet.appendRow([
    data.waktu,
    data.mode,
    data.nama,
    data.nis || "",
    data.absen || "",
    data.kelas || "",
    data.anggota || "",
    data.materi,
    data.skor,
    data.total
  ]);
  return ContentService.createTextOutput(JSON.stringify({ status: "ok" }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

Simpan project (nama bebas, misal "API Portal Informatika").

## 3. Deploy sebagai Web App

1. **Deploy → New deployment**.
2. Klik ikon gear ⚙️ → pilih **Web app**.
3. Isi: **Execute as: Me**, **Who has access: Anyone**.
4. **Deploy**, setujui izin akses yang diminta (wajar, karena ini script milik Anda sendiri).
5. Salin **Web app URL** (`https://script.google.com/macros/s/xxxxx/exec`).

## 4. Tempel URL — satu-satunya file yang diedit

Buka `config.js`, ganti:
```javascript
const CONTROL_URL = "https://script.google.com/macros/s/GANTI_DENGAN_ID_DEPLOYMENT/exec";
```
dengan URL asli dari langkah 3. Upload ulang ke GitHub. Setelah ini, **tidak ada file kode lain yang perlu disentuh lagi**.

## 5. Menambah materi baru (workflow rutin)

1. Buat file HTML materinya dengan pola lengkap berikut (lihat `belajar-cpp-xi.html` di folder `modules/` sebagai contoh jadi):

   **a. CSS overlay loading** — taruh di `<style>`, sebelum `</head>`:
   ```css
   #guard-loading{
     position:fixed; inset:0; background:var(--bg); z-index:999;
     display:flex; flex-direction:column; align-items:center; justify-content:center; gap:14px;
     font-family:var(--font-mono); color:var(--text-dim); font-size:13px;
   }
   #guard-loading .spin{
     width:34px; height:34px; border-radius:50%;
     border:3px solid var(--surface-2); border-top-color:var(--amber);
     animation:guardspin .8s linear infinite;
   }
   @keyframes guardspin{to{transform:rotate(360deg);}}
   ```
   Sesuaikan warna (`--bg`, `--surface-2`, `--amber`, dst) dengan palet file materi Anda.

   **b. Overlay + pembungkus konten** — tepat setelah `<body>`:
   ```html
   <div id="guard-loading">
     <div class="spin"></div>
     <div>Memuat materi…</div>
   </div>

   <div id="konten-utama" style="visibility:hidden">
     ... seluruh isi materi di sini ...
   </div><!-- /#konten-utama -->
   ```
   Overlay `#guard-loading` menutupi layar penuh (jadi siswa lihat spinner, bukan konten kelihatan sekilas), sementara `#konten-utama` disembunyikan lewat `visibility:hidden` sampai guard selesai mengecek.

   **c. Blok guard** — tepat sebelum `</body>`, setelah `</div><!-- /#konten-utama -->`:
   ```html
   <script>
     const MODUL_ID = "INF-XXX";
     // Jaring pengaman: kalau config.js/guard.js belum ada di folder yang sama,
     // atau pengecekan ke Google Sheet lambat/menggantung, materi tetap otomatis
     // ditampilkan — tidak nyangkut selamanya di layar "Memuat materi...".
     let __revealed = false;
     function __forceReveal(){
       if(__revealed) return;
       __revealed = true;
       const k = document.getElementById('konten-utama');
       if(k) k.style.visibility = 'visible';
       const l = document.getElementById('guard-loading');
       if(l) l.remove();
     }
     setTimeout(__forceReveal, 4000);
   </script>
   <script src="../config.js" onerror="__forceReveal()"></script>
   <script src="../guard.js" onerror="__forceReveal()"></script>
   ```
   Ganti `INF-XXX` dengan ID yang akan Anda pakai di Sheet. `onerror` pada dua tag `<script>` dan `setTimeout` 4 detik memastikan materi tidak pernah nyangkut kalau ada masalah jaringan/file — ini pelengkap fallback yang sudah ada di dalam `guard.js` sendiri (yang juga otomatis menutup overlay `#guard-loading` begitu status Sheet dipastikan, baik terkunci maupun terbuka).
2. Upload file itu ke folder `modules/` di GitHub (commit langsung lewat GitHub web, atau upload file).
3. Buka Google Sheet, tambahkan 1 baris baru di tab **Materi** dengan ID, judul, deskripsi, kategori, semester, icon, dan path file yang sama.
4. Centang kolom **Tampilkan** kapan pun materi itu siap dibuka untuk siswa.

Tidak ada lagi file JSON atau kode portal yang perlu diedit untuk menambah materi — semuanya lewat Sheet, kecuali file HTML materinya sendiri yang memang harus diupload.

## 6. Menerapkan form data siswa (NIS/Absen/Kelas/Kelompok) ke kuis lain

File `modules/kuis-jaringan-komputer.html` sudah jadi contoh lengkapnya. Untuk kuis baru yang Anda buat sendiri, cara tercepat:

1. Copy file `kuis-jaringan-komputer.html`, ganti nama sesuai kuis baru Anda.
2. Ganti `MODUL_ID` di bagian bawah file dengan ID kuis yang baru.
3. Ganti isi soal (elemen `#soal` dan pilihan `.opsi`), sesuaikan logika `jawab()` dan hitungan skor totalnya kalau soal lebih dari satu.
4. Bagian form data siswa (mode Individu/Kelompok, field Kelas/Nama/NIS/Absen/Anggota) dan fungsi `kirimHasil()` **tidak perlu diubah** — tinggal dipakai apa adanya karena sudah otomatis mengirim ke kolom yang sama di tab HasilKuis.

## Catatan soal `guard.js`

File `guard.js` di root repo **tidak perlu diedit** — cukup ikuti pola pemasangan di Langkah 5. Perilakunya:
- Kalau materi terkunci (`Tampilkan` tidak dicentang) → seluruh halaman diganti dengan pesan "Materi belum dibuka" + tautan kembali ke portal.
- Kalau materi terbuka → `#konten-utama` dibuat terlihat.
- Di kedua kondisi itu, overlay `#guard-loading` (spinner "Memuat materi…") otomatis dihapus begitu status dari Sheet dipastikan.
- Kalau fetch ke Sheet gagal (mis. offline), materi tetap ditampilkan apa adanya supaya tidak mengunci siswa karena masalah jaringan.

## Catatan jujur soal batasan

Ini situs statis (GitHub Pages), bukan server sungguhan. Pengecekan visibilitas berjalan lewat JavaScript di browser siswa — cukup efektif untuk kebutuhan kelas (materi hilang dari daftar, halaman terkunci walau diakses lewat link langsung), tapi bukan keamanan tingkat tinggi. Siswa yang sangat mahir teknis (mis. membaca kode sumber langsung) secara teori masih bisa melihat isi mentah file. Untuk kontrol akses materi kelas sehari-hari, ini sudah lebih dari cukup.
