# Spesifikasi data untuk Peta 3D FeNi Pomalaa

Untuk tim pembuat CMMS (Project 0). Istilah mengikuti BRD CMMS v1.0: status Andon (§6.7), level aset L6-L8 (§6.5),
Tf/T1/T3 (§6.13), FB dan LTB (§6.16).

Peta adalah **tampilan lokasi dari Andon**. Peta hanya menampilkan; tidak menghitung KPI dan tidak menulis ke CMMS.
Semua status dan angka harus berasal dari view yang sama dengan Andon, supaya sama persis di semua tampilan.

## 1. Bentuk data

Satu objek JSON berisi daftar Sistem (L6):

```json
{
  "source": "CMMS",
  "updated": "2026-11-17T08:00:00+08:00",
  "sample": false,
  "systems": [
    {
      "id": "aset-L6-0031",
      "system": "Rotary Kiln 3",
      "state": "MAINTENANCE OFF",
      "tf": "2026-11-17T10:05:00+08:00",
      "at": "2026-11-17T10:20:00+08:00",
      "cmOpen": 0,
      "fb": false,
      "ltb": false,
      "note": "",
      "subs": [
        { "name": "Drive System", "state": "MAINTENANCE OFF", "at": "2026-11-17T10:20:00+08:00" }
      ],
      "wo": [
        { "no": "WO-000123", "type": "CM", "title": "Main Drive Motor trip", "st": "Dikerjakan",
          "sub": "Drive System", "eq": "3-RK-M01", "prio": "P1", "wc": "242 ELM 3" }
      ]
    }
  ]
}
```

### Tingkat atas

| Field | Wajib | Keterangan |
|---|---|---|
| `systems` | ya | Daftar Sistem (L6). Sistem yang tidak dikirim dianggap RUNNING. |
| `updated` | disarankan | Waktu update terakhir (ISO 8601). Ditampilkan di peta, seperti di Andon (FR-AND-05). |
| `source` | tidak | Nama sumber yang ditampilkan. Bawaan `CMMS`. |
| `sample` | tidak | `true` bila data contoh; peta memberi label "DATA CONTOH". |
| `lines` | tidak | Angka kinerja per Line, lihat "Angka kinerja". |

### Per Sistem

| Field | Wajib | Keterangan |
|---|---|---|
| `system` | ya* | Nama Sistem di CMMS, mis. `Rotary Kiln 3`. Huruf besar/kecil dan spasi tidak berpengaruh. |
| `id` | ya* | ID atau tag aset di CMMS. Dipakai bila sudah didaftarkan di `FENI_CONFIG.assetMap` (lihat §3). |
| `state` | ya | Status Andon Sistem (tabel di bawah). |
| `at` | disarankan | Sejak kapan status ini berlaku. Untuk "durasi status". |
| `tf` | untuk BREAKDOWN dan MAINTENANCE OFF | Awal kegagalan (Tf). Downtime dihitung dari sini. |
| `cmOpen` | tidak | Jumlah WO CM terbuka yang belum dimulai (FR-AND-01c). Bila tidak dikirim, peta menghitung dari `wo`. |
| `fb`, `ltb` | tidak | Penanda Frequent Breakdown dan Long Time Breakdown yang aktif (FR-RCA-09). |
| `subs` | tidak | Status Sub-sistem (L7): `name`, `state`, `at`. Yang tidak dikirim dianggap RUNNING. |
| `wo` | tidak | WO yang masih terbuka pada Sistem ini dan turunannya. |
| `note` | tidak | Catatan singkat. |

\* Minimal salah satu dari `system` atau `id`.

### Status (`state`)

Persis enam status Andon. Urutan prioritas bila beberapa berlaku bersamaan ditentukan CMMS (FR-AND-01b); peta hanya menampilkan yang dikirim.

| Nilai | Arti | Warna di peta |
|---|---|---|
| `BREAKDOWN` | OFF karena kerusakan, pekerjaan belum dimulai (Tf sampai T1) | merah |
| `MAINTENANCE OFF` | OFF karena kerusakan, perbaikan berjalan (T1 sampai T3) | biru |
| `STANDBY` | OFF karena WO PM atau CMs sedang dikerjakan | kuning |
| `IDLE` | OFF karena alasan produksi | abu-abu |
| `MAINTENANCE ON` | Tetap jalan, ada WO yang sedang dikerjakan | hijau dengan tanda biru |
| `RUNNING` | Jalan, tidak ada pekerjaan yang berjalan | hijau |

Penulisan boleh dengan spasi, garis bawah, atau huruf kecil (`maintenance_off`).

### Work order

| Field | Keterangan |
|---|---|
| `no` | Maintenance ID / nomor WO |
| `type` | `CM`, `PM`, atau `CMs` |
| `title` | Deskripsi singkat |
| `st` | Status WO (mis. `Dibuat`, `Dikerjakan`). WO CM berstatus `Dibuat` dihitung sebagai "belum dimulai" bila `cmOpen` tidak dikirim. |
| `sub` | Nama Sub-sistem (L7). Bila Sub-sistem itu punya lokasi sendiri di peta, baris WO bisa diklik menuju ke sana. |
| `eq` | Tag Equipment (L8), mis. `3-RK-M01` |
| `prio` | Prioritas (P1, P2, ...) |
| `wc` | Work Center |
| `due` | Jatuh tempo (untuk PM/CMs) |
| `disc` | Disiplin: `Electrical` atau `Mechanical`. Instrument (243 INS) adalah departemen di bawah Electrical Maintenance Bureau, jadi WO instrument dikirim sebagai `Electrical` (nilai `Instrument` tetap diterima dan dihitung Electrical). Bila `disc` kosong, peta memakai biro dari work center. |
| `wc` | Work center atau pool piket yang mengerjakan, misalnya `242 ELM 3` atau `PIKET EMIP`. Peta mengenali Departemen dan Biro dari kode di nama (sesuai BRD §2.1.1: Departemen MM 202 MEC, MSP 204 PEW, MW 203 MWC, MP 201 PLA → Mechanical Maintenance Bureau; EM 242 ELM, IM 243 INS, EIP 241 PLAN → Electrical Maintenance Bureau) untuk filter work center. |

### Angka kinerja (`metrics`)

Peta **tidak menghitung** MTTR, MTBF, MA, PA, UA, dan EU. CMMS menghitungnya menurut aturan BRD §6.13 (penggabungan overlap, koreksi Tf/T3, model waktu), lalu mengirim angka jadi. Peta menampilkannya apa adanya di tab "Kinerja" pada kartu peralatan, dan per Line di kotak "Kondisi peralatan".

Per Sistem, tambahkan `metrics`. Isinya satu set, atau daftar set untuk beberapa periode:

```json
{
  "system": "Rotary Kiln 4",
  "state": "RUNNING",
  "metrics": [
    { "period": "7d",  "asOf": "2026-11-17T06:00:00+08:00", "mttr": 5.4, "mtbf": 71,  "ma": 91.2, "pa": 92.8, "ua": 96.4, "eu": 89.5 },
    { "period": "30d", "asOf": "2026-11-17T06:00:00+08:00", "mttr": 4.9, "mtbf": 104, "ma": 94.1, "pa": 95.0, "ua": 96.9, "eu": 92.1 }
  ]
}
```

| Field | Keterangan |
|---|---|
| `period` | `today` (Hari ini), `7d`, `30d`, `mtd` (Bulan berjalan). Teks lain ditampilkan apa adanya. Peta hanya menawarkan periode yang dikirim. |
| `asOf` | Waktu hitung (ISO 8601). Ditampilkan sebagai "per ...". |
| `mttr`, `mtbf` | Dalam jam. |
| `ma`, `pa`, `ua`, `eu` | Dalam persen (0 sampai 100). |
| `bdh`, `bdn` | Waktu Breakdown (jam) dan Jumlah Breakdown pada periode itu, sesuai BRD §6.13 (FR-BDT-01). |
| `resp`, `respIn` | Rata-rata response time (menit) dan persentase WO yang dalam target (FR-KPI-08). |
| `recov`, `recovIn` | Rata-rata recovery time (jam) dan persentase dalam target (FR-KPI-08). |

- Nilai boleh berupa teks, misalnya `"> 168"` untuk MTBF saat tidak ada breakdown pada periode itu (BRD §6.13.6). Teks ditampilkan apa adanya.
- Metrik yang tidak dikirim ditampilkan "—". Peta tidak memperkirakan angka.
- Bentuk lain yang juga diterima: `"metrics": { "7d": {...}, "30d": {...} }`.
- Periode bawaan yang dipilih peta adalah `30d` bila dikirim (BRD FR-AND-04). Urutan tombol: Hari ini, 7 hari, 30 hari, Bulan berjalan.

### Target KPI (`targets`)

Di tingkat atas. Ditetapkan Manajemen/Engineer di CMMS (FR-KPI-06); peta hanya memakainya untuk warna:

```json
"targets": { "ma": 92, "pa": 93, "ua": 90, "eu": 85, "mtbf": 100, "mttr": 6, "resp": 30, "recov": 8 }
```

- MA/PA/UA/EU/MTBF: hijau bila ≥ target, kuning bila kurang dari 5 poin di bawah target (MTBF: ≥ 80% target), merah di bawahnya. Selisih kuning bisa diubah lewat `FENI_CONFIG.perfWarnGap`.
- MTTR, Waktu/Jumlah BD: makin kecil makin baik; hijau ≤ target, kuning ≤ 125% target.
- Respons dan pemulihan (FR-KPI-08): hijau di bawah 80% target, kuning 80% sampai target, merah lewat target.
- Tanpa `targets`, kartu tidak diwarnai dan pilihan warna "Kinerja" di peta 3D memakai batas `FENI_CONFIG.perfColor`.

### RCA (`rca`)

Per Sistem, daftar RCA yang masih terbuka (FR-RCA-08/09):

```json
"rca": [ { "no": "RCA-0012", "trigger": "FB", "status": "DIKERJAKAN", "due": "2026-11-28", "wc": "242 ELM 3", "late": false } ]
```

`trigger`: `FB` atau `LTB`. RCA berstatus `DISETUJUI` tidak ditampilkan. Bila `late` tidak dikirim, peta menganggap RCA terlambat saat `due` sudah lewat.

### Geofence (`geofence`)

Per Sistem (FR-AST-03): `"geofence": { "r": 50, "lat": -4.1835, "lon": 121.6402 }`, radius dalam meter. Lingkaran digambar di peta saat kartu peralatan dibuka. Tanpa `lat`/`lon`, titik pusatnya posisi aset di peta. Agar `lat`/`lon` tepat di peta, isi `FENI_CONFIG.geoRef` dengan dua titik acuan `[{lat,lon,x,y},{lat,lon,x,y}]` (x, y = koordinat peta); peta sudah memakai dua titik acuan bawaan dari screenshot Google Earth berkisi koordinat (ketelitian ±30 m); isi `geoRef` sendiri bila punya titik ukur GPS yang lebih tepat.

Per Line, di tingkat atas, dengan isi `metrics` yang sama:

```json
"lines": [
  { "line": "Line 4", "metrics": [ { "period": "30d", "ma": 94.6, "pa": 95.5, "ua": 96.3, "eu": 92.0 } ] }
]
```

Tabel per Line di peta hanya menampilkan MA, PA, UA, dan EU. Angka per Line juga dihitung CMMS. Peta tidak merata-ratakan angka Sistem menjadi angka Line.

Angka kinerja dipakai di empat tempat: tab Ringkasan pada kartu peralatan, Diagram alir (satu angka pilihan di tiap blok), pilihan warna "Kinerja", dan Mode TV per Line.

Pilihan warna "Kinerja" mewarnai peralatan menurut MA, PA, UA, atau EU dengan dua batas. Bawaan peta adalah **nilai contoh** (hijau 92% atau lebih, kuning 85% sampai 92%, merah di bawah 85%) dan belum ditetapkan plant. Batas diatur lewat `FENI_CONFIG.perfColor`, misalnya `{ "good": 92, "warn": 85 }`.

### Yang dihitung peta sendiri

- **LTB**: bila `ltb` tidak dikirim, peta menandai LTB saat downtime sejak `tf` lebih dari 18 jam (bisa diubah lewat `FENI_CONFIG.ltbHours`). Sebaiknya `ltb` dikirim CMMS supaya aturannya satu sumber.
- **Perkiraan dampak**: saat sebuah Sistem OFF, peta menandai peralatan hilir yang *diperkirakan* ikut berhenti menurut alur proses. Ini bukan status CMMS dan selalu diberi label "perkiraan". Status sebenarnya (mis. IDLE) tetap dari catatan operator.

## 2. Cara mengirim data

Web CMMS memakai Next.js dengan login dan hak akses per peran, jadi cara yang disarankan adalah **B**: halaman web yang sudah login mengambil data, lalu mengirimkannya ke peta.

### A. Alamat data

Peta mengambil sendiri dari satu alamat tiap 60 detik. Cocok untuk file contoh atau endpoint tanpa login.

```html
<script>window.FENI_CONFIG = { dataUrl: "/api/peta/kondisi", refresh: 60, mode: "kerja" };</script>
```

Tanpa pengaturan, peta membaca `data/kondisi.json` di samping `index.html`.

### B. Peta ditanam di halaman CMMS (iframe)

```html
<iframe id="peta" src="/peta/index.html#kerja"></iframe>
<script>
  const peta = document.getElementById("peta");
  // kirim data kapan saja; kirim ulang tiap kali Andon diperbarui (mis. dari Supabase Realtime)
  peta.contentWindow.postMessage({ type: "feni:data", data: dataKondisi }, location.origin);
  // buka satu Sistem di peta
  peta.contentWindow.postMessage({ type: "feni:select", id: "Rotary Kiln 3" }, location.origin);
  // peta memberi tahu Sistem yang dibuka pengguna, mis. untuk membuka halaman aset
  addEventListener("message", (e) => {
    if (e.data && e.data.type === "feni:selected") console.log(e.data.id, e.data.name);
  });
</script>
```

Isi `FENI_CONFIG.parentOrigin` dengan origin web CMMS. Setelah diisi, peta hanya menerima pesan dari origin itu
dan baru mengirim pesan `feni:ready` dan `feni:selected` ke halaman induk.

| Pesan ke peta | Isi |
|---|---|
| `feni:data` | `data`: objek seperti §1 |
| `feni:select` | `id`: nama Sistem, ID aset, atau ID peta |
| `feni:clear` | menghapus data yang dikirim |

### C. Dipanggil dari skrip di halaman yang sama

```js
FENI.setData(dataKondisi);   // mengembalikan { mapped, unmapped }
FENI.select("Rotary Kiln 3");
FENI.clear();
```

## 3. Mencocokkan aset CMMS dengan peta

Peta mengenali nama Sistem. Bila CMMS mengirim ID atau tag, daftarkan padanannya sekali:

```html
<script>
  window.FENI_CONFIG = {
    assetMap: { "aset-L6-0031": "Rotary Kiln 3", "3-RK": "Rotary Kiln 3" }
  };
</script>
```

Sistem yang belum punya lokasi di peta tidak ditampilkan, tetapi dihitung dan disebutkan di panel "Kondisi peralatan".
Saat ini 28 dari 52 Main System pada tabel lama sudah punya lokasi; daftar ini perlu dicocokkan ulang dengan registri aset CMMS baru.

## 4. Tampilan dan hak akses

| Alamat | Tampilan |
|---|---|
| `index.html#kerja` | Tampilan kerja: panel kondisi terbuka, tombol presentasi disembunyikan |
| `index.html#pantau` | Layar pantau: berpindah otomatis antar Sistem yang tidak RUNNING atau punya WO terbuka |
| `index.html` | Lengkap, termasuk fungsi penjelasan proses |
| `index.html#presentasi` | Mode presentasi (tur proses → tur pemantauan maintenance → jalan) |
| `index.html#tur-maintenance` | Langsung memutar tur pemantauan maintenance (6 langkah, ±1 menit) |
| `index.html#denah` | Denah 2D (tampak atas di atas citra) |

Andon di BRD hanya baca dan hanya untuk Planner, Engineer, dan Manajemen (FR-AND-08). Peta tidak punya login sendiri,
jadi pembatasan itu dilakukan oleh halaman CMMS yang menanamnya.

## 4b. Riwayat per rentang tanggal

Panel "Riwayat maintenance" menampilkan aktivitas pada rentang tanggal yang dipilih (dari–sampai, atau satu hari). Bagian ini hanya muncul bila file riwayat tersedia.

- Alamat file: `FENI_CONFIG.historyUrl` (bawaan `data/riwayat.json`). Bentuk: `{ "source", "sample", "note", "records": [ ... ] }`.
- Satu record = satu aktivitas: `s`, `e` (mulai/selesai, `YYYY-MM-DDTHH:MM` waktu lokal), `h` (jam), `t` (`PM`/`CM`), `sys`, `so` (Sistem Off: true/false), `sub`, `bo` (Sub-sistem Off), `eq`, `wo`, `ker` (kerusakan), `ur` (uraian), `jn` (jenis pekerjaan), `ket` (`Selesai`/`Berlanjut`), `wc`, `line`, `mh` (man hour). **Jangan sertakan nama pelaksana.**
- Yang dihitung peta dari file ini hanyalah **perkiraan untuk tampilan contoh**: jumlah PM/CM, "Sistem OFF" (aktivitas CM dengan Sistem Off, digabung bila tumpang tindih), jam OFF, MTTR (rata-rata durasi CM) dan PA (100% − jam OFF ÷ jam rentang).
- **Untuk sambungan CMMS yang sebenarnya**, disarankan CMMS menyediakan perhitungan per rentang (misalnya `...?from=2026-09-01&to=2026-09-15`) dengan bentuk `metrics` yang sama seperti di atas, supaya aturan BRD §6.13 tetap satu sumber.
- Tombol "Warnai peta menurut riwayat ini" mewarnai Sistem menurut jam OFF / jumlah CM pada rentang terpilih (mode warna "Riwayat"), dan kartu alat menampilkan garis waktu OFF/CM/PM.
- Data riwayat asli tidak disimpan di repo publik ini.

## 5. Mode TV per Line

Papan satu Line untuk layar besar: peta di kiri dengan alur Line, daftar Sistem pada Line itu di kanan (status, durasi, LTB/FB, WO terbuka, angka kinerja). Hanya menampilkan data.

- Alamat: tambahkan `#tv-line4`, `#tv-line2`, atau `#tv-line3` di belakang alamat peta.
- Dari skrip: `FENI.tv('L4')`.
- Dari pengaturan: `FENI_CONFIG.tv = "L4"` membuka papan Line 4 saat peta dimuat.
- Rotasi otomatis antar Line (FR-AND-06): alamat `#tv-auto`, `FENI.tv('auto')`, `FENI_CONFIG.tv = "auto"`, atau tombol "Putar otomatis" di papan. Jeda bawaan 20 detik, diatur lewat `FENI_CONFIG.tvRotate` (detik).
- Sistem yang masuk satu Line ditentukan dari alur proses di peta, sehingga crane dan casting ikut tampil. Untuk memakai pembagian Line dari CMMS, isi `FENI_CONFIG.lineSystems`, misalnya `{ "L4": ["Rotary Dryer 4", "Rotary Kiln 4", "Electric Smelting Furnace 4"] }`. Nama boleh nama Sistem, ID peta, atau ID/tag aset yang ada di `assetMap`.

## 5a. Tautan ke halaman CMMS

Bila diisi, peta menampilkan tautan "Buka di CMMS ↗" pada WO dan RCA, serta tautan aset dan drawing di kartu alat (FR-AST-05, FR-AST-07). Tanpa pengaturan ini tautan tidak ditampilkan.

```js
window.FENI_CONFIG = {
  cmmsLinks: {
    wo:      "https://cmms.example/wo/{no}",          // {no} = nomor WO
    rca:     "https://cmms.example/rca/{no}",         // {no} = nomor RCA
    asset:   "https://cmms.example/asset?sys={system}", // {system} = nama Sistem CMMS, {id} = ID peta
    drawing: "https://cmms.example/drawing?sys={system}"
  }
};
```

## 5c. Pemberitahuan perubahan status

Peta membandingkan status setiap 2 detik. Bila sebuah Sistem masuk atau keluar dari BREAKDOWN, MAINTENANCE OFF, STANDBY atau IDLE, muncul pemberitahuan di bawah layar (klik untuk menuju Sistem), tercatat di "Perubahan status terakhir" (8 terakhir), dan layar TV berkedip untuk BREAKDOWN. Bunyi bisa dinyalakan per perangkat (tombol 🔔). Tidak perlu data tambahan; cukup status yang dikirim seperti biasa.

## 5b. Belum ada di peta (usulan berikutnya)

Kalibrasi `geoRef` bawaan: ±30 m (dari kisi koordinat Google Earth). Bisa diperbaiki dengan dua titik ukur GPS di lapangan.

## 6. Catatan

- Jangan menaruh data plant asli di repo publik. `data/kondisi.json` di repo ini adalah data contoh.
- Hasil pendataan dan edit layout di versi Claude disimpan di database artifact dan tidak ikut ke versi ini.
- Posisi dan ukuran di peta skematis; kaitan Sistem ke lokasi dicocokkan dari nama dan belum seluruhnya dicek di lapangan.
