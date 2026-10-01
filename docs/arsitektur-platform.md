# W.M.P — Warehouse Management Platform
Dokumen arsitektur dan rencana kerja (versi 0.1, rancangan untuk disepakati).

## 1. Tujuan
Satu aplikasi web terpusat untuk gudang **DC 38 Tigalapan**: mulai dari pengelolaan SDM, pencatatan transaksi, sampai inventori, sehingga spreadsheet tidak lagi menjadi sumber data. MOKA tetap menjadi POS untuk transaksi ke toko.

Struktur mengikuti konsep "Warehouse Management Platform":

```
                    W.M.P (Warehouse Management Platform)
        ┌───────────────────┴───────────────────┐
      WMS                                      WOS
 (apa, di mana, berapa)                  (siapa, kapan, bagaimana)
 data induk, stok, lokasi,               tugas operator, eksekusi
 aturan, jejak audit                     di lapangan (tablet)
        └───────────────────┬───────────────────┘
                     PEOPLE / EMPLOYEE
        (karyawan, peran, shift, presensi, penugasan, performance)

   + lapisan lintas modul: INTEGRASI & DOKUMEN
     (upload PPIC, upload intransit, lembar transfer MOKA, STTB, SPL, ekspor)
```

Aliran data:
- **People → WOS:** siapa yang hadir dan berperan apa menentukan siapa yang mendapat tugas.
- **WMS → WOS:** apa yang harus diambil dan dari lokasi mana (FIFO), apa yang harus disimpan dan di mana.
- **WOS → WMS:** setiap eksekusi (terima, simpan, ambil, kirim) menulis pergerakan stok dengan jam asli.
- **Semua modul → Audit Trail.**

## 1a. Peran W.M.P: pusat pencatatan dan aktivitas gudang
W.M.P bukan sekadar portal web. W.M.P adalah **sistem pencatatan utama (system of record)** sekaligus **pusat aktivitas** gudang DC 38. WMS dan WOS saling terhubung melalui People/Employee, sehingga setiap kegiatan gudang tercatat lengkap: apa yang terjadi, siapa operatornya, dan kapan.

**Yang dicatat di modul WMS (transaksi dan aktivitas gudang):**
- Seluruh transaksi **inbound** (intransit, GR, counting, selisih, put away) dan **outbound** (SOQ, picking, VP, packing, loading, STTB).
- **Relokasi** antar bin (membawa tanggal masuk asli untuk FIFO).
- **Cycle count** dan stock opname, beserta selisih dan penyesuaiannya.
- Adjustment, transfer, dan kegiatan gudang lain; setiap perubahan stok menjadi `stock_movements` dengan jam asli.

**Yang dicatat di modul WOS (penugasan dan eksekusi):**
- **Pemberian assignment/task per operator**: siapa mendapat tugas apa, dari siapa, kapan diberikan, dan perubahan atau pemindahan tugasnya.
- **Eksekusi operasional**: jam mulai dan selesai, hasil (qty aktual, selisih, temuan), serta operator yang mengerjakan.

**Penghubung:** setiap tugas WOS merujuk ke Employee (People) dan ke dokumen/pergerakan stok WMS, sehingga satu kejadian dapat ditelusuri dari tugas, operator, sampai perubahan stok. Semua tercatat di Audit Trail.

## 2. Nama dan modul
| Modul | Isi |
|---|---|
| **People** | Employee, User & Role, Shift (Rate 1 dan 2), Attendance (termasuk lembur dan SPL), cuti dan Change Off, Assignment, Performance, Compliance, Coaching |
| **WMS** | Master Data (SKU/item, lokasi/bin, outlet, norma), Inventory, Stock Movement, Receiving dan Putaway (perencanaan), Picking (perencanaan: alokasi, FIFO), Transfer, Stock Adjustment, Stock Opname, Reporting, Audit Trail |
| **WOS** | Task Operator, Receiving Execution, Putaway Execution, Picking, Packing, Loading, Scanning (nanti), Operator Activity, Real-time Monitoring |
| **Integrasi & Dokumen** | Upload SOQ PPIC, upload intransit, lembar transfer MOKA (salin per baris + penanda), STTB, SPL, surat jalan, ekspor CSV |

## 3. Pengguna dan peran
| Peran | Akses utama |
|---|---|
| PPIC | upload SOQ, melihat status permintaan |
| Admin Produksi | upload data intransit/produksi, melihat status |
| Admin Outbound | meninjau SOQ, rilis pembagian picking, lembar transfer MOKA, STTB, Load Plan |
| Admin Inbound | intransit, GR, put away, data entry PO |
| Picker, Checker, Packer, Driver, Tim GR | tugas masing-masing di tablet; Checker mencatat temuan |
| HRD | data karyawan, presensi, cuti, SPL (penerima) |
| Leader, Head of Department | pengajuan dan pengetahuan SPL |
| Supervisor | pantau semua, ubah tim dan penugasan, koreksi |
| Owner | akses penuh, pengaturan, audit |

Login: **kode staf + PIN** (6 angka). Employee (data SDM) terpisah dari User (akun) dan Role (hak akses); satu orang bisa memegang beberapa peran, dan peran harian dipilih saat presensi (mis. Picker).

## 4. Kesiapan multi-gudang (rancangan saja)
Saat ini hanya **DC 38**. Agar toko atau gudang lain bisa bergabung kelak tanpa merombak data, setiap tabel transaksi dan master membawa `warehouse_id` (bawaan: DC38). Transfer antar gudang dan integrasi dengan departemen toko baru dirancang setelah mereka setuju; tidak dibangun sekarang.

## 5. Model data inti (garis besar)
- **employees, users, roles, shifts, attendance, overtime_spl, leave** — People.
- **items** (kode SKU, nama item unik, kategori, UoM, status aktif) — kunci operasional sementara tetap nama item; kode SKU dipetakan dari master.
- **locations** (kode pendek `A1.1`, kode panjang, zona, tipe PICKING/BUFFER/STORAGE, jenis rak, kapasitas, status, barcode).
- **stock_layers** (SKU, lokasi, tanggal masuk, qty) — dasar **FIFO**; relokasi membawa tanggal asli.
- **stock_movements** (jenis: OPENING, INBOUND, OUTBOUND, RELOKASI, ADJUSTMENT; no dokumen; dari/ke lokasi; qty; user; jam asli).
- **soq_orders / soq_lines** (hasil upload PPIC: outlet, item, qty, jenis REGULAR/LONJAKAN/CRITICAL).
- **pick_tasks** (koli, picker, baris, lokasi, qty, ACT, jam mulai dan selesai), **verifications** dan **findings** (Compliance).
- **shipments** (packing/staging, Load Plan, ekspedisi, tanggal kirim, loading, unloading, STTB).
- **inbound** (intransit, batch GR, counting, selisih, QC sampling, put away).
- **task_assignments** (tugas, jenis, operator/employee, pemberi tugas, waktu diberikan, status, riwayat pindah tugas) dan **task_executions** (jam mulai/selesai, qty aktual, hasil, rujukan ke dokumen dan `stock_movements`).
- **cycle_counts** (jadwal, lokasi/SKU, qty sistem vs hitung, selisih, penyesuaian).
- **audit_log** (siapa, apa, kapan, nilai sebelum dan sesudah).

## 6. Aturan operasional yang disepakati
- **Pembagian picking otomatis:** SOQ yang diimpor dibagikan ke operator yang sudah presensi dengan peran Picker; seimbang menurut norma waktu; CRITICAL didahulukan; admin dapat meninjau dan mengubah sebelum rilis. Sisa koli diambil picker lewat "ambil koli berikutnya".
- **Rekomendasi lokasi:** FIFO menurut tanggal masuk paling lama. Tombol "stok bin tidak sesuai" di tablet menyarankan bin berikutnya dan mencatat selisih.
- **Template SOQ PPIC (standar header):** `TANGGAL | KODE OUTLET | KATEGORI | NAMA ITEM | QTY | JENIS | KETERANGAN`. Kelipatan dan pembulatan tetap milik PPIC; W.M.P menerima angka akhir.
- **MOKA:** tanpa integrasi API (belum diketahui ada atau tidak). Lembar transfer siap-salin dengan penanda "sudah masuk MOKA" per baris.
- **Inbound:** data berasal dari intransit; produksi di luar cakupan awal. QC hanya sampling.
- **Penomoran dokumen otomatis dan tidak dapat diubah:** `STTB-YYYYMM-NNNN`, `SPL-YYYYMM-NNNN`, dibuat di server.

## 7. Posisi aplikasi W.O.S yang berjalan sekarang
| Bagian | Sudah ada di W.O.S | Menjadi |
|---|---|---|
| People | Employee master, Attendance, Lembur dan SPL, cuti, tim dan rate, Compliance, performance, rekap operator | modul People |
| WOS | assignment, jadwal picking, VP, packing, Load Plan, loading, GR per batch, STTB, monitoring | modul WOS |
| WMS | hanya master ringan | **dibangun baru** (inti: stok, lokasi, pergerakan, FIFO, opname, audit) |

Aplikasi W.O.S tetap berjalan di alamatnya sekarang selama masa migrasi; logika yang sudah teruji dipindahkan fitur demi fitur.

## 8. Fase kerja
| Fase | Isi |
|---|---|
| 0. Fondasi | repositori dan proyek Vercel baru, proyek Supabase, login kode staf + PIN, Employee dan User & Role, kerangka modul dan tampilan tablet |
| 1. People dan WMS inti (mode bayangan) | pindahkan data People ke database; impor master SKU, lokasi, saldo, dan riwayat dari spreadsheet WMS; hitung stok per bin dan lapisan FIFO; bandingkan rekomendasi sistem dengan pilihan admin selama beberapa hari tanpa mengganti cara kerja |
| 2. WOS Outbound | upload SOQ, pembagian otomatis, picking di tablet, VP, temuan Compliance, lembar transfer MOKA, packing, Load Plan, STTB |
| 3. WOS Inbound | upload intransit, GR batch, counting dan selisih, put away dengan rekomendasi lokasi, data entry |
| 4. WMS lengkap | transfer, adjustment, opname, laporan, audit trail penuh |
| 5. Barcode | label lokasi dan SKU, scan kamera atau scanner |

## 9. Teknologi (usulan)
- Hosting: **Vercel**, proyek terpisah dari W.O.S.
- Data, login, dan realtime: **Supabase** (Postgres, Auth, Row Level Security, Realtime).
- Aplikasi: satu aplikasi web modular (**Vite + TypeScript**, React) yang dapat dipasang sebagai PWA di tablet, dengan tampilan layar sentuh besar untuk operator.
- Hak akses dijaga di database (RLS), bukan hanya di tampilan.

## 10. Keputusan yang masih terbuka
1. Daftar field pasti template SOQ dan siapa yang menjaganya.
2. Aturan FIFO: bin picking dulu lalu isi ulang dari buffer, atau murni tanggal masuk di semua bin.
3. Aturan pembentukan koli (pcs maksimal, 1 atau 2 koli per trolley).
4. Alur pindah tugas bila picker berhalangan.
5. Pemilik perawatan master SKU dan lokasi.
6. Jumlah tablet dan kondisi Wi-Fi gudang; tanggal target mulai paralel.
7. Ada tidaknya API MOKA (opsional, tidak menghalangi).
