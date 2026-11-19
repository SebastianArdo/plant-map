# Peta 3D FeNi Pomalaa

Peta 3D skematis plant FeNi Pomalaa dalam satu file HTML (three.js r128 dari CDN).
Bentuk, ukuran dan posisi adalah skematis; animasi proses adalah ilustrasi cara kerja, bukan data operasi.

## Isi

| File | Isi |
|---|---|
| `index.html` | Peta (kode dan data peta) |
| `data/kondisi.json` | Data kondisi dan WO. Yang ada di repo ini adalah **data contoh**. |
| `data/riwayat.json` | Riwayat maintenance (data dummy log aktivitas CMMS September 2026, tanpa nama pelaksana) |
| `img/` | Citra satelit Google Earth (plant 21/9/2023) untuk alas peta, beserta masker tepinya |
| `SPESIFIKASI-DATA.md` | Bentuk data dan cara menghubungkan ke website CMMS |

## Menjalankan

Peta harus dibuka lewat server web (bukan klik dua kali), karena membaca `data/kondisi.json`:

```
python -m http.server 8000
```

lalu buka `http://localhost:8000/`. Di GitHub Pages cukup aktifkan Pages untuk repo ini.
Tanpa file data, peta tetap jalan dan panel kondisi bertuliskan "Belum terhubung".

## Memperbarui data

Ganti isi `data/kondisi.json`, atau arahkan peta ke alamat lain lewat `FENI_CONFIG.dataUrl`.
Lihat `SPESIFIKASI-DATA.md`.

## Tampilan

`#kerja` tampilan kerja · `#pantau` layar pantau · `#presentasi` mode presentasi.

`#tv-line4` (juga `#tv-line2`, `#tv-line3`) Mode TV per Line · `#kondisi` panel kondisi.

## Citra satelit

Alas peta memakai screenshot Google Earth (tombol **Citra** di kanan untuk menyalakan/mematikan). Atribusi Google wajib tetap tampil. Sebelum repo dibuat publik, pastikan penggunaan citra ini sesuai ketentuan Google dan kebijakan perusahaan; jika ragu, simpan repo sebagai private atau hapus folder `img/` (peta tetap jalan tanpa citra).
