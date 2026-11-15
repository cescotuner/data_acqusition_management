# Tugas 1 - Akuisisi Data Akomodasi Pariwisata Kabupaten Gianyar (OpenStreetMap)

Mata kuliah BIFP-243 Akuisisi dan Manajemen Data, INSTIKI.
Pengambil: Francesco Nera Alfi Nanggor (NIM 2501010174).

## 1. Sumber data
- **Nama:** OpenStreetMap, diakses lewat Overpass API
- **URL:** https://overpass-api.de/api/interpreter
- **Penerbit:** OpenStreetMap contributors (data); layanan Overpass API
- **Alasan pemilihan:** API publik tanpa API key, berlisensi terbuka (ODbL), dan relevan dengan sektor pariwisata: sebaran akomodasi di Kabupaten Gianyar, salah satu pusat wisata budaya Bali. Sumber ini juga termasuk yang direkomendasikan pada brief tugas.

## 2. Tanggal dan jam akses
2026-11-16 02:02 WITA (sumber: `acquired_at` di `docs/provenance.json`).

## 3. Metode akuisisi
API (HTTP POST) ke `https://overpass-api.de/api/interpreter`, parameter `data` berisi query Overpass QL. Ada dua request:
1. Pencarian wilayah: relasi batas administrasi yang namanya memuat "Gianyar", dipilih yang `admin_level=5` (kabupaten). Hasilnya (nama dan ID relasi) tercatat di `provenance.json` (`region`).
2. Pengambilan data:

```
[out:json][timeout:60];
area(id:3600000000 + <ID relasi>)->.a;
nwr["tourism"~"^(hotel|guest_house|hostel|motel|apartment|resort|chalet)$"](area.a);
out center tags;
```
Query lengkap beserta ID relasi yang dipakai tersimpan di `provenance.json`.

Tidak memakai paginasi karena Overpass mengembalikan seluruh hasil dalam satu respons. Timeout 90 detik, jeda 1 detik sebelum setiap request, retry maksimal 4 kali dengan exponential backoff 1, 2, 4, 8 detik untuk timeout, koneksi putus, 429 (memakai `Retry-After` bila ada), dan 5xx. Status 4xx lain menghentikan program.

## 4. Batasan etika
- **Pemeriksaan robots.txt:** dilakukan dengan `RobotFileParser`; isi dan hasilnya disimpan di `docs/provenance.json` (`robots_check`) dan tercetak di notebook. Hasil: `can_fetch = False`. RobotFileParser mengembalikan can_fetch=False untuk User-Agent ini (isi robots.txt tersimpan di atas). Akses tetap dilakukan karena Overpass API adalah API publik resmi yang direkomendasikan pada brief tugas dan diatur oleh ketentuan penggunaannya sendiri (satu query, cek /api/status, jeda, User-Agent jujur). Keputusan ini dicatat terbuka di README butir 4. Sesuai brief, kepatuhan dibuktikan lewat robots.txt atau ketentuan API; untuk API publik resmi ini ketentuan API (poin berikut) menjadi acuan utama.
- **Ketentuan API Overpass:** tidak mengirim query berulang (satu query data, ditambah satu pencarian wilayah), mengecek slot lewat `/api/status` sebelum mengirim query, dan memakai retry dengan backoff.
- **Lisensi dan atribusi:** ODbL 1.0. Atribusi: "(c) OpenStreetMap contributors".
- **Jeda dan User-Agent:** minimal 1 detik antar-request. User-Agent mencantumkan nama, NIM, dan email pengambil.
- **Data yang sengaja tidak diambil:** nomor telepon, email, situs web dan akun media sosial, nama operator/pemilik, serta seluruh tag `contact:*`.
- Tidak ada API key.

## 5. Penjelasan kolom (`data/raw/osm_overpass_<YYYYMMDD>.csv`)
| Kolom | Keterangan |
|---|---|
| `osm_type` | Tipe objek OSM: node, way, atau relation |
| `osm_id` | ID objek di OSM |
| `name` | Nama akomodasi (bisa kosong) |
| `tourism` | Jenis akomodasi: hotel, guest_house, hostel, motel, apartment, resort, chalet |
| `stars` | Jumlah bintang, jika dicatat |
| `rooms` | Jumlah kamar, jika dicatat |
| `addr_street` | Nama jalan |
| `addr_city` | Kota/desa pada alamat |
| `addr_postcode` | Kode pos |
| `lat`, `lon` | Koordinat (untuk way/relation memakai titik tengah) |

Kolom yang kosong dibiarkan apa adanya karena data mentah tidak diubah; pembersihan dilakukan pada RTM Tugas 2.

**Jumlah data:** 905 record, 11 kolom (sumber: sel verifikasi notebook).

## 6. Cara menjalankan ulang
1. Buka `notebooks/akuisisi_osm_overpass.ipynb` di Jupyter atau Google Colab (Python 3.9+, paket `requests`).
2. Jalankan dari folder `notebooks/` (path `../data/raw` dan `../docs`). Di Colab, ubah `RAW_DIR` dan `DOCS_DIR` menjadi `data/raw` dan `docs`, lalu pindahkan hasilnya ke struktur folder ini.
3. Pilih **Restart & Run All**. Hasil: `data/raw/osm_overpass_<YYYYMMDD>.csv` dan `docs/provenance.json`.
4. Jika berkas hari itu sudah ada, notebook berhenti agar data mentah tidak tertimpa. Hapus berkas lama secara manual (di luar notebook) hanya jika pengambilan sebelumnya gagal.

Data (c) OpenStreetMap contributors, tersedia di bawah ODbL.
