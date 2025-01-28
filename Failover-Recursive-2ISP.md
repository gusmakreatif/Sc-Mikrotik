
# Konfigurasi Failover Recursive pada MikroTik dengan 2 ISP

Panduan ini memberikan langkah-langkah untuk mengkonfigurasi routing failover recursive pada MikroTik dengan dua ISP:

- **ISP 1 (Starlink)**: Gateway - `192.168.3.1`
- **ISP 2 (IndiHome)**: Gateway - `192.168.2.1`
- **Server DNS**: `1.1.1.1` dan `1.1.1.2`

## Prasyarat
- Router MikroTik dengan kedua ISP sudah terhubung dan dikonfigurasi.
- Akses ke router MikroTik melalui WinBox atau SSH.

## 1. Pengaturan Gateway Routes
### Konfigurasi Routing Recursive
Untuk mengatur routing recursive untuk failover:

```bash
# Konfigurasi Gateway Starlink dengan Recursive Check
/ip route
add dst-address=1.1.1.1 gateway=192.168.3.1 distance=1 check-gateway=ping comment="Gateway Starlink Recursive Check"
add dst-address=0.0.0.0/0 gateway=1.1.1.1 distance=1 comment="Default Route via Starlink"

# Konfigurasi Gateway IndiHome sebagai Backup
add dst-address=1.1.1.2 gateway=192.168.2.1 distance=2 check-gateway=ping comment="Gateway IndiHome Recursive Check"
add dst-address=0.0.0.0/0 gateway=1.1.1.2 distance=2 comment="Default Route via IndiHome (Backup)"
```

### Penjelasan
- **Check-Gateway**: Memastikan gateway aktif dengan melakukan ping ke DNS yang ditentukan.
- **Distance**: Menentukan prioritas rute; jarak lebih rendah (1) lebih diutamakan.

## 2. Pengaturan DNS
Konfigurasikan MikroTik untuk menggunakan server DNS yang ditentukan:

```bash
/ip dns
set servers=1.1.1.1,1.1.1.2 allow-remote-requests=yes
```

## 3. Pengaturan NAT Masquerade
Pastikan NAT masquerade dikonfigurasi dengan benar untuk kedua ISP:

```bash
/ip firewall nat
add chain=srcnat out-interface=<interface-starlink> action=masquerade comment="NAT Starlink"
add chain=srcnat out-interface=<interface-indihome> action=masquerade comment="NAT IndiHome"
```

Ganti `<interface-starlink>` dan `<interface-indihome>` dengan nama interface yang sesuai untuk Starlink dan IndiHome.

## 4. Pengaturan Netwatch untuk Optimasi Failover
Netwatch digunakan untuk memantau koneksi dan memastikan failover lebih responsif.

### Konfigurasi Netwatch
1. Masuk ke menu **Netwatch** di MikroTik.
2. Tambahkan dua entry untuk masing-masing ISP:

```bash
# Entry untuk memantau koneksi Starlink
/tool netwatch
add host=1.1.1.1 interval=10s timeout=3s up-script="/ip route enable [find comment=\"Default Route via Starlink\"]" down-script="/ip route disable [find comment=\"Default Route via Starlink\"]"

# Entry untuk memantau koneksi IndiHome
add host=1.1.1.2 interval=10s timeout=3s up-script="/ip route enable [find comment=\"Default Route via IndiHome (Backup)\"]" down-script="/ip route disable [find comment=\"Default Route via IndiHome (Backup)\"]"
```

### Penjelasan
- **host**: IP yang akan dipantau (gunakan DNS untuk setiap ISP).
- **interval**: Waktu jeda antara pengecekan koneksi (10 detik).
- **timeout**: Waktu tunggu sebelum dianggap gagal (3 detik).
- **up-script**: Mengaktifkan route jika koneksi kembali normal.
- **down-script**: Menonaktifkan route jika koneksi gagal.

## 5. Pengujian Failover
1. Putuskan koneksi ISP IndiHome dan uji konektivitas melalui Starlink.
2. Putuskan koneksi ISP Starlink dan pastikan lalu lintas beralih ke IndiHome.

## Catatan
- Pastikan kedua gateway ISP dapat dijangkau dan dikonfigurasi dengan benar.
- Fitur `check-gateway=ping` memastikan routing hanya beralih jika gateway utama tidak dapat dijangkau.
- Konfigurasi Netwatch membantu mengoptimalkan failover dengan memantau koneksi secara real-time.

Dengan mengikuti langkah-langkah ini, router MikroTik Anda akan otomatis beralih ke ISP cadangan (IndiHome) jika ISP utama (Starlink) tidak tersedia.
