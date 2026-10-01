# W.M.P — Gambaran alur kerja (versi ringkas)

W.M.P adalah satu "papan kerja" bersama yang dilihat semua orang. Isinya tiga bagian yang saling bicara:

- **People** tahu siapa yang masuk kerja hari ini.
- **WMS** tahu barangnya apa, di rak mana, dan berapa banyak.
- **WOS** mengatur siapa mengerjakan apa di lapangan.

## Satu hari kerja Outbound
1. **Pagi: karyawan presensi.** Operator masuk dengan kode staf + PIN, presensi, dan memilih peran hari itu (mis. Picker). Sistem jadi tahu siapa saja picker yang ada.
2. **PPIC upload permintaan toko** (file Excel dengan header standar). Sistem membaca otomatis: outlet, item, dan jumlahnya.
3. **Sistem membuat rencana picking.** Barang dipecah menjadi koli lalu dibagi ke picker yang hadir, seimbang menurut beban. Untuk tiap barang, sistem menunjuk rak yang isinya paling lama masuk (FIFO). Admin hanya memeriksa dan menekan Rilis.
4. **Picker mengambil barang di tablet.** Di layar tampil daftar: rak, barang, jumlah. Picker mengisi jumlah yang diambil. Kalau rak ternyata kosong, ia menekan "stok tidak sesuai" dan sistem langsung menunjuk rak lain. Jam tiap langkah tercatat sendiri.
5. **Checker memeriksa koli di sistem.** Kalau ada kesalahan, dicatat sebagai temuan dan masuk Compliance, lalu koli kembali ke picker.
6. **Packing dan staging** terjadi otomatis setelah pemeriksaan selesai. Admin mendapat lembar transfer MOKA (daftar siap salin per outlet) dan menandai tiap baris yang sudah dimasukkan.
7. **Pengiriman:** admin mengatur Load Plan (tanggal kirim dan ekspedisi), mencetak STTB, lalu driver mencentang loading dan unloading.
8. **Stok di sistem berkurang otomatis** sesuai yang diambil, jadi saldo selalu mengikuti kenyataan.

## Alur Inbound (versi singkat)
Admin produksi upload data intransit. Tim GR menerima barang per batch tiga koli, menghitung (counting), lalu menyimpan. Sistem menyarankan rak untuk menyimpan dan menambah stok. Admin menyelesaikan data entry PO.

## Mengapa lebih efisien dibanding sekarang
| Sekarang | Dengan W.M.P |
|---|---|
| Admin memposting SOQ dan membagi baris manual | Dibagi otomatis |
| Admin mengetik lokasi dari master | Lokasi disarankan sistem (FIFO) |
| Jam selesai hanya perkiraan | Jam asli tercatat |
| Picking disalin ke MOKA satu per satu dari sheet | Daftar siap salin plus penanda per baris |
| Stok per rak di spreadsheet terpisah | Stok ikut bergerak otomatis |
| Data tersebar di browser tiap orang | Satu data bersama, dengan login |

## Yang tetap sama
MOKA tetap menjadi kasir dan pencatat transaksi ke toko. Yang berubah adalah pekerjaan di dalam gudang.

## Tiga hal yang perlu diingat
1. **Semua berjalan bertahap.** Awalnya W.M.P hanya "mengintip" data spreadsheet dan membandingkan saran rak dengan pilihan admin, tanpa mengganti cara kerja. Spreadsheet baru dimatikan setelah terbukti cocok.
2. **Kualitas saran rak bergantung pada kedisiplinan mencatat:** setiap penerimaan, relokasi, dan pengambilan harus masuk sistem.
3. **Aplikasi W.O.S yang sekarang tetap berjalan** sampai W.M.P siap menggantikannya.

Rincian teknis: [arsitektur-platform.md](arsitektur-platform.md).
