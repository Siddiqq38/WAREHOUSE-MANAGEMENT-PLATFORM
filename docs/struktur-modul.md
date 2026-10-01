# W.M.P — Model dan struktur modul (usulan)

Tiap modul punya halaman induk (hub) berisi submodul. Submodul berstatus **Tersedia** atau **Rencana**.

```
W.M.P
├─ People  (siapa)            people.html
│   ├─ Employee               ✔ people-employee.html (26 karyawan DC 38)
│   ├─ User & Role            login kode staf + PIN, hak akses
│   ├─ Shift & Rate           shift 1/2, rate, pola hari kerja/off
│   ├─ Attendance             presensi, pilih peran hari itu, JOTM, approval SPV/MGR
│   ├─ Lembur & SPL
│   ├─ Cuti & Ijin            wajib ada pengganti/hand over
│   ├─ Performance            dari jam eksekusi WOS
│   └─ Compliance & Coaching  temuan Checker per operator
│
├─ WMS  (apa, di mana, berapa)  wms.html
│   ├─ Master Item            ✔ wms-item.html (4.775 item)
│   ├─ Lokasi & Stok per Bin  ✔ wms-bin.html (1.972 bin)
│   ├─ Norma & Aturan         kapasitas, FIFO, penempatan
│   ├─ Movement Log           semua IN/OUT per bin per hari
│   ├─ Relokasi
│   ├─ Cycle Count & Opname
│   ├─ Adjustment & Transfer
│   ├─ Rekomendasi Lokasi     FIFO picking, bin put away
│   └─ Laporan & Audit Trail
│
├─ WOS  (siapa mengerjakan apa)  wos.html
│   ├─ Assignment, Daily Board
│   ├─ Inbound:  Receiving (GR), Put Away
│   ├─ Outbound: Picking, Checker & Verifikasi, Packing & Staging,
│   │            Load Plan / STTB / Pengiriman
│   └─ Monitoring Real-time, Operator Activity
│
└─ Integrasi & Dokumen            integrasi.html
    Upload SOQ, Upload Intransit, Lembar Transfer MOKA, STTB, SPL & Surat Jalan, Ekspor
```

## Aturan model data (agar modul saling bicara)
1. **Satu pergerakan stok = satu baris `stock_movements`** (IN, OUT, RELOKASI, ADJUSTMENT, OPNAME). Saldo bin adalah hasil penjumlahan pergerakan, bukan angka yang diketik. Ini yang membuat "stok selalu mengikuti kenyataan".
2. **WOS tidak mengubah stok langsung.** Eksekusi tugas (put away, picking, relokasi) menghasilkan pergerakan di WMS.
3. **Tugas selalu menunjuk Employee** (`task_assignments.employee_id`) dan menyimpan jam asli mulai/selesai.
4. **Attendance menentukan siapa yang bisa dapat tugas** (hadir + peran hari itu).
5. Semua tabel membawa `warehouse_id` (DC38), `created_by/at`, dan masuk `audit_log`.

## Sumber data WMS awal
Spreadsheet **WMS DCTSI 25.xlsx** (disimpan 28 Nov 2025):

| Sheet | Isi | Dipakai untuk |
|---|---|---|
| MASTER STORAGE | bin, item, bal, pcs, kapasitas maks, status (Free/Full/Over), slot (Utama/Cadangan) | Lokasi & Stok per Bin |
| MASTER ITEM 2 | kategori, nama item, area simpan | Master Item |
| MOVEMENT LOG | saldo awal + IN/OUT per bin per tanggal | Movement Log (belum diimpor) |
| LAYER | denah blok/rak (peta gudang) | Peta gudang (rencana) |

Data diekspor apa adanya ke `assets/data/` dan ditampilkan baca-saja. Catatan: arti satuan "Bal" dan "Kapasitas maks" perlu dikonfirmasi; kolom ditampilkan sesuai judul di spreadsheet.

## Urutan pengerjaan yang disarankan
1. People: Employee ✔, lalu User & Role + Attendance (syarat untuk WOS).
2. WMS: Master Item ✔, Bin & Stok ✔, lalu Movement Log (mode bayangan: bandingkan dengan spreadsheet).
3. WOS Outbound, lalu Inbound.
4. Integrasi (upload SOQ/intransit, MOKA, STTB) mengikuti kebutuhan WOS.

Penyimpanan bersama (database + login) belum tersambung; semua halaman saat ini statis.
