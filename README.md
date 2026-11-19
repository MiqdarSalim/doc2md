# Doc2MD

Konverter **DOCX** dan **PDF** ke **Markdown** yang hemat token dan mudah dibaca AI. Satu file HTML, tanpa backend, tanpa build step. Semua diproses di browser, file tidak diunggah ke server mana pun.

## Fitur

| Area | DOCX | PDF |
|---|---|---|
| Heading | Dari style Word | Dari ukuran font relatif |
| Tebal, miring, coret | Ya | Tebal/miring dari nama font |
| Daftar | Bertingkat | Bullet dan bernomor |
| Tabel | Tabel Markdown (GFM) | Deteksi dari posisi kolom |
| Tautan, catatan kaki | Ya | Tidak |
| Gambar | Ekstrak, base64, atau abaikan | Ekstrak di posisi aslinya |
| OCR | Teks di dalam gambar (opsional) | Halaman scan + teks di gambar |
| Verifikasi | Perbandingan jumlah kata, peringatan elemen tak dikenal | Halaman kosong, tabel terdeteksi, halaman OCR |

Hemat token:
- Gambar diekstrak ke folder `images/` dan hanya direferensikan di `.md` (default), bukan base64.
- Header, footer, dan nomor halaman berulang dihapus (PDF).
- Spasi berlebih, baris kosong beruntun, dan marker daftar dirapikan.
- Ligatur dinormalisasi (NFKC) dan tanda hubung akhir baris digabung kembali.

Anti-halusinasi:
- Kata OCR berkeyakinan rendah ditandai `[?]`.
- Penanda halaman `<!-- hal. N -->` agar AI bisa mengutip halaman.
- Kartu hasil menampilkan prompt pendamping yang bisa disalin, sesuai penanda yang muncul.
- Laporan verifikasi per file.

## Tech stack

Murni HTML + CSS + JavaScript native (tanpa framework, bundler, atau Node.js). Library dimuat dari CDN dengan versi dipatok:

| Library | Versi | CDN | Fungsi |
|---|---|---|---|
| [mammoth.js](https://github.com/mwilliamson/mammoth.js) | 1.6.0 | cdnjs | DOCX ke HTML semantik, ekstraksi gambar |
| [pdf.js](https://github.com/mozilla/pdf.js) | 3.11.174 | cdnjs | Ekstraksi teks, posisi, font, gambar; render halaman untuk OCR |
| [Turndown](https://github.com/mixmark-io/turndown) | 7.1.2 | jsDelivr | HTML ke Markdown |
| turndown-plugin-gfm | 1.0.2 | jsDelivr | Tabel dan coret GFM |
| [Tesseract.js](https://github.com/naptha/tesseract.js) | 5.1.1 | jsDelivr | OCR (WebAssembly) |
| [JSZip](https://github.com/Stuk/jszip) | 3.10.1 | cdnjs | Paket `.md` + `images/` menjadi `.zip` |

Data bahasa OCR (`ind`, `eng`) diunduh otomatis oleh Tesseract.js saat pertama dipakai.

## Alur kerja

```
DOCX -> mammoth (HTML + gambar) -> rapikan DOM (tabel, heading) -> Turndown (+GFM) -> tidy -> .md
PDF  -> pdf.js (teks + posisi + font + gambar)
        -> halaman tanpa teks? -> render canvas -> Tesseract OCR
        -> susun baris -> deteksi heading / daftar / tabel / paragraf
        -> hapus header/footer -> sisipkan gambar sesuai posisi -> tidy -> .md
Hasil -> .md saja, atau .zip (.md + images/)
```

Catatan teknis PDF: pdf.js berjalan di main thread (worker disuntik lewat `pdf.worker.min.js` sebagai script biasa) supaya tidak bergantung pada Web Worker lintas origin.

## Deploy ke GitHub Pages

1. Simpan file sebagai `index.html` di repo.
2. Settings, Pages, pilih branch dan folder root.
3. Buka URL Pages. Tidak ada build step.

Bisa juga dibuka langsung dari disk (`file://`). Butuh internet saat pertama kali untuk memuat library CDN dan data bahasa OCR. Untuk penggunaan offline, unduh semua library dan data `.traineddata` ke repo dan ubah path-nya.

## Batasan

- PDF multi-kolom: urutan baca bisa tertukar.
- Tabel PDF kompleks (sel digabung, multi-baris): hasil bisa meleset.
- File `.doc` lama dan Google Docs: simpan sebagai `.docx` terlebih dahulu.
- Garis bawah dan warna teks tidak punya padanan di Markdown.
- Kotak teks, header/footer, dan komentar DOCX bisa tidak terbaca (dilaporkan sebagai peringatan).
- OCR lambat dan akurasinya bergantung kualitas scan (disarankan 300 dpi, halaman lurus).
- Bold/italic PDF bergantung pada nama font di file.

## Panduan akurasi

1. Periksa laporan di kartu hasil. Peringatan berarti perlu cek manual.
2. Salin prompt pendamping yang muncul ke prompt Anda.
3. Verifikasi angka, nama, dan tanggal penting dari hasil OCR atau tabel PDF ke dokumen asli.
4. Gunakan DOCX atau PDF digital bila ada, bukan scan.
5. Untuk gambar penting, kirim gambarnya langsung ke AI yang bisa melihat gambar.
6. Minta AI mengutip dan menjawab "tidak ada di dokumen" bila informasi tidak ditemukan.

## Struktur

```
index.html   # seluruh aplikasi (UI, logika, gaya)
README.md
```

## Lisensi

Kode aplikasi mengikuti lisensi repo Anda. Library pihak ketiga memiliki lisensinya masing-masing (umumnya MIT, BSD, atau Apache-2.0); periksa sebelum distribusi komersial.
