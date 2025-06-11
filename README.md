Intinya sih, hubungan antara proyek kamu fitfromhome dan Supabase itu nyambung lewat sebuah **jembatan yang namanya API (Application Programming Interface)**.

Bayanginnya gini deh:
* **Proyek kamu** itu pelanggan di sebuah restoran.
* **Database PostgreSQL di Supabase** itu dapurnya.
* **API** itu pelayan yang tugasnya nyatet pesanan kamu, nganterin ke dapur, dan bawain makanan (data) balik ke kamu.

Pelanggan kan nggak bisa langsung nyelonong masuk ke dapur, nah proyek kamu juga gitu, nggak bisa langsung "ngobok-obok" database. Semuanya harus lewat perantara (API) yang udah ada aturannya.

Gini nih rincian teknis prosesnya:

### 1. Kunci dan Alamat Unik (API Keys & Project URL)

Waktu kamu bikin proyek di Supabase, kamu dikasih dua hal yang penting banget:
* **Project URL:** Ini alamat unik "restoran" Supabase kamu di internet. Setiap permintaan dari proyek kamu bakal ditujuin ke alamat ini.
* **API Keys:** Ini kayak "kartu identitas" yang kamu tunjukkin ke "pelayan" (API). Ada dua jenis utama:
    * `anon key` (kunci anonim): Kunci ini aman buat kamu pakai di sisi klien (browser/aplikasi mobile). Kunci ini cuma ngasih akses sesuai aturan *Row Level Security* (RLS) yang kamu atur di database. Contohnya, "pengguna A cuma boleh lihat datanya sendiri".
    * `service_role key`: Kunci ini super rahasia dan cuma boleh kamu pakai di sisi server (backend). Siapa pun yang punya kunci ini bisa ngelewatin semua aturan RLS dan punya akses penuh ke database kamu. **Jangan sampe bocorin kunci ini di kode frontend!**

### 2. "Pelayan" Otomatis: Auto-generated API

Nah, ini kerennya Supabase. Waktu kamu bikin tabel di database PostgreSQL, Supabase otomatis ngebangunin serangkaian "pelayan" API di atasnya.
* **Untuk data (CRUD):** Supabase pakai *engine* bernama **PostgREST**. *Engine* ini otomatis ngubah tabel database kamu jadi **RESTful API**. Jadi, pas kamu mau ambil data (`SELECT`), bikin data (`INSERT`), ubah (`UPDATE`), atau hapus (`DELETE`), proyek kamu sebenernya lagi ngelakuin permintaan HTTP (kayak `GET`, `POST`, `PATCH`, `DELETE`) ke URL API yang udah dibikinin PostgREST.
* **Untuk Realtime:** Supabase pakai *engine* beda buat fitur realtime. Proyek kamu bakal bikin koneksi terus-terusan pakai **WebSockets** ke layanan Realtime Supabase. Lewat koneksi ini, tiap ada perubahan di database (misalnya ada data baru), Supabase bakal langsung "teriak" dan ngirim info perubahan itu ke proyek kamu tanpa perlu kamu minta (di-*refresh*).

### 3. "Penerjemah" di Proyek Kamu: Supabase Client Library

Biar kamu nggak usah repot-repot nulis kode permintaan HTTP manual (yang lumayan ribet), Supabase nyediain **Client Library** (contohnya `supabase-js` untuk JavaScript, `supabase-py` untuk Python, dll).

Library ini tugasnya jadi "penerjemah" atau "pembantu" di kode kamu.

**Jadi, alur lengkapnya tuh kayak gini (contoh ambil data):**

1.  **Kamu Nulis Kode Simpel:** Di kode proyek kamu, kamu nulis kode yang gampang dibaca pakai client library, contohnya:
    ```javascript
    const { data, error } = await supabase
      .from('produk')
      .select('*')
    ```

2.  **Client Library Beraksi:** Di balik layar, `supabase-js` bakal:
    * Ngejalanin kode di atas.
    * Bikin permintaan HTTP `GET` yang lengkap.
    * Nargetin URL: `[Project URL kamu]/rest/v1/produk?select=*`
    * Masukin `anon key` kamu ke dalam *header* permintaan sebagai `apikey` buat otentikasi.

3.  **Supabase Nerima Permintaan:** Server Supabase nerima permintaan ini.
    * API Gateway-nya bakal ngecek `apikey` kamu buat mastiin permintaannya valid.
    * Permintaan ini diterusin ke *engine* PostgREST.
    * PostgREST nerjemahin permintaan API jadi kueri SQL: `SELECT * FROM produk;`
    * Kueri SQL ini dijalanin di database PostgreSQL kamu, sambil ngikutin aturan RLS yang berlaku buat `anon key`.

4.  **Data Dikirim Balik:**
    * Database-nya ngembaliin hasilnya ke PostgREST.
    * PostgREST ngubah hasilnya jadi format JSON (format data yang umum di web).
    * Supabase ngirim balik data JSON ini sebagai respons HTTP ke proyek kamu.

5.  **Proyek Kamu Nerima Data:** Client library (`supabase-js`) nerima respons JSON itu dan masukin isinya ke variabel `data` yang kamu bikin di awal.

Intinya, koneksi antara proyek kamu dan Supabase itu adalah **komunikasi API yang udah diatur**, yang dibantuin sama **Client Library** di sisi proyek kamu dan **API engine otomatis** di sisi Supabase, plus diamanin sama **API Keys** dan **Row Level Security**.
