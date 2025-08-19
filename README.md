# 📦 Dokumentasi Teknis API Order Kuota Gateway

Dokumen ini menyediakan spesifikasi teknis yang komprehensif untuk Antarmuka Pemrograman Aplikasi (API) Order Kuota Gateway. API ini dirancang untuk memfasilitasi pengelolaan pengguna, pembuatan faktur pembayaran QRIS, dan penerimaan notifikasi secara terprogram.

**URL Dasar (Base URL):** `https://gateway.rerechanstore.eu.org`

> **Pemberitahuan Penting:** Penggunaan API ini secara langsung dalam lingkungan produksi tidak direkomendasikan. Disarankan untuk memanfaatkan aplikasi resmi yang telah disediakan oleh pengembang guna menjamin stabilitas dan keamanan operasional.

---

## Daftar Isi

- [Dukungan Finansial](#-dukungan-finansial)
- [Prosedur Alur Penggunaan API](#-prosedur-alur-penggunaan-api)
- [Spesifikasi Titik Akhir (Endpoints) API](#-spesifikasi-titik-akhir-endpoints-api)
  - [1. Otentikasi Pengguna](#1-otentikasi-pengguna)
  - [2. Klaim OTP dan Akuisisi Token](#2-klaim-otp-dan-akuisisi-token)
  - [3. Permintaan Detail Pengguna](#3-permintaan-detail-pengguna)
  - [4. Pembaruan Data Pengguna](#4-pembaruan-data-pengguna)
  - [5. Permintaan Riwayat Transaksi (Mutasi)](#5-permintaan-riwayat-transaksi-mutasi)
  - [6. Pembuatan Faktur (Invoice)](#6-pembuatan-faktur-invoice)
  - [7. Permintaan Detail Faktur](#7-permintaan-detail-faktur)
  - [8. Generasi Gambar QRIS](#8-generasi-gambar-qris)
- [Mekanisme Panggilan Balik (Callback/Webhook)](#-mekanisme-panggilan-balik-callbackwebhook)
- [Versi Bahasa Inggris](#technical-documentation-order-kuota-gateway-api)

---

## 💸 Dukungan Finansial

Pengoperasian dan pemeliharaan infrastruktur server memerlukan sumber daya finansial yang berkelanjutan. Apabila layanan ini dinilai memberikan manfaat, dukungan dapat disalurkan melalui donasi via kode QRIS yang terlampir. Setiap bentuk kontribusi sangat dihargai.

![QRIS Donasi](https://raw.githubusercontent.com/FN-Rerechan02/FN-Rerechan02/main/photo_2025-05-14_09-23-18.jpg)

---

## 🚀 Prosedur Alur Penggunaan API

Untuk dapat mengakses sebagian besar titik akhir (endpoint), diperlukan proses otentikasi dua tahap sebagai berikut:

1.  **Otentikasi Kredensial Pengguna:** Mengirimkan `username` dan `password` untuk keperluan verifikasi identitas.
2.  **Akuisisi Token Otentikasi:** Mengirimkan `username` dan `otp` (One-Time Password) yang telah diterima untuk memperoleh **Bearer Token**.
3.  **Utilisasi Token:** Menyertakan Bearer Token yang telah diperoleh dalam *header* `Authorization` pada setiap permintaan ke titik akhir yang memerlukan otentikasi.

---

## 🔑 Spesifikasi Titik Akhir (Endpoints) API

Berikut adalah rincian mengenai titik akhir yang tersedia.

### 1. Otentikasi Pengguna

Titik akhir ini berfungsi untuk memverifikasi kredensial pengguna dan menginisiasi proses pengiriman OTP.

-   **Endpoint:** `POST /api/v2/users/auth`
-   **Request Body:**
    ```json
    {
      "username": "string",
      "password": "string"
    }
    ```
-   **Responses:**
    -   **Respon Sukses (200 OK)**: Mengindikasikan bahwa kredensial valid dan OTP telah dikirimkan.
        ```json
        {
          "status": null,
          "data": {
            "email": {
              "status": "Success",
              "data": {
                "email": "user***********@example.com"
              }
            }
          }
        }
        ```
    -   **Respon Kegagalan Kredensial (200 OK)**
        ```json
        {
           "status": null,
           "data": {
              "email": {
                 "status": "Error",
                 "data": {
                    "error": "Username or password Wrong"
                 }
              }
           }
        }
        ```

### 2. Klaim OTP dan Akuisisi Token

Titik akhir ini wajib diakses setelah otentikasi berhasil untuk mendapatkan Bearer Token.

-   **Endpoint:** `POST /api/v2/users/auth/otp`
-   **Request Body:**
    ```json
    {
       "username": "string",
       "otp": "string"
    }
    ```
-   **Responses:**
    -   **Respon Sukses (200 OK)**: Token akan diterima. Token ini harus disimpan dengan aman.
        ```json
        {
          "status": "OK",
          "data": {
            "token": "a1b2c3d4-e5f6-7890-g1h2-i3j4k5l6m7n8",
            "username": "contoh_user",
            "email": "user@example.com"
          }
        }
        ```
    -   **Respon Kegagalan OTP (400 Bad Request)**
        ```json
        {
           "status": "Error",
           "data": {
              "error": "OTP was wrong"
           }
        }
        ```

### 3. Permintaan Detail Pengguna

Mengambil data detail mengenai seorang pengguna.

-   **Endpoint:** `GET /api/v2/users/details/{username}`
-   **Headers:**
    -   `Authorization`: `Bearer <token>`
-   **Responses:**
    -   **Respon Sukses (200 OK)**
        ```json
        {
          "username": "contoh_user",
          "email": "user@example.com",
          "token": "a1b2c3d4-e5f6-7890-g1h2-i3j4k5l6m7n8",
          "qrcode": "data:image/png;base64,iVBO....5CYII=",
          "callback_url": "https://anda.com/webhook",
          "qris_string": "000201010...6304A0D7"
        }
        ```
    -   **Respon Pengguna Tidak Ditemukan (404 Not Found)**
        ```json
        {
          "timestamp": "YYYY-MM-DDTHH:MM:SS.sssssss",
          "status": 404,
          "error": "Not Found",
          "message": "Cannot invoke \"...User.getUsername()\" because \"user\" is null",
          "path": "/api/v2/users/details/nama_user_salah"
        }
        ```

### 4. Pembaruan Data Pengguna

Memperbarui data pengguna, seperti alamat surel dan URL panggilan balik.

-   **Endpoint:** `POST /api/v2/users/update`
-   **Headers:**
    -   `Authorization`: `Bearer <token>`
-   **Request Body:**
    ```json
    {
      "username": "string",
      "email": "string",
      "callback_url": "string"
    }
    ```
    > Parameter `username` bersifat **mandatori**. Parameter `email` dan `callback_url` bersifat opsional.
-   **Responses:**
    -   **Respon Sukses (200 OK)**
        ```json
        {
          "status": "OK",
          "data": {
            "username": "contoh_user",
            "email": "user_baru@example.com",
            "callback_url": "https://anda.com/webhook_baru"
          }
        }
        ```
    -   **Respon Parameter Tidak Valid (400 Bad Request)**
        ```json
        {
          "status": "Error",
          "data": {
            "email": "Invalid email"
          }
        }
        ```

### 5. Permintaan Riwayat Transaksi (Mutasi)

Mengambil riwayat transaksi pengguna dengan fungsionalitas paginasi.

-   **Endpoint:** `GET /api/v2/statements/{username}`
-   **Headers:**
    -   `Authorization`: `Bearer <token>`
-   **Query Parameters:**
    -   `page`: `integer` (Opsional, nilai default: `0`)
    -   `size`: `integer` (Opsional, nilai default: `10`)
-   **Responses:**
    -   **Respon Sukses (200 OK)**
        ```json
        {
          "page": 0,
          "size": 10,
          "data": [
            {
              "id": 162052951,
              "username": "contoh_user",
              "debet": 0,
              "kredit": 15000,
              "keterangan": "BANK / TR***********************",
              "status": "IN",
              "statement_status": "NOT_CLAIMED",
              "transfer_time": "YYYY-MM-DDTHH:MM:SS"
            }
          ],
          "total_content": 21,
          "total_pages": 3,
          "has_next": true,
          "has_previous": false,
          "total_data": 10
        }
        ```
    -   **Respon Pengguna Tidak Ditemukan (204 No Content)**
    -   **Respon Token Tidak Sah (401 Unauthorized)**
        ```json
        { "message": "Token Invalid" }
        ```

### 6. Pembuatan Faktur (Invoice)

Membuat sebuah faktur pembayaran QRIS baru.

-   **Endpoint:** `POST /api/v2/invoices/create`
-   **Headers:**
    -   `Authorization`: `Bearer <token>`
-   **Request Body:**
    ```json
    {
      "notes": "Pembayaran untuk layanan X",
      "amount": 50000,
      "expires_at": 3600
    }
    ```
    > `expires_at` merupakan durasi kedaluwarsa faktur dalam satuan **detik**.
-   **Responses:**
    -   **Respon Sukses (200 OK)**
        ```json
        {
          "username": "contoh_user",
          "amount": 50000,
          "status": "PENDING",
          "note": "Pembayaran untuk layanan X",
          "expires_at": 1755001134825,
          "created_at": "YYYY-MM-DDTHH:MM:SS.sssssssss",
          "qris_string": "000201010212...630495B0",
          "invoice_id": "inv_ad411c75-f6e3-49cc-b92a-195e9afd1a7b"
        }
        ```
    -   **Respon Token Tidak Sah (401 Unauthorized)**

### 7. Permintaan Detail Faktur

Mendapatkan data detail dari sebuah faktur berdasarkan ID-nya.

-   **Endpoint:** `GET /api/v2/invoices/details/{invoice_id}`
-   **Headers:**
    -   `Authorization`: `Bearer <token>`
-   **Responses:**
    -   **Respon Sukses (200 OK)**
        ```json
        {
          "username": "contoh_user",
          "amount": 50000,
          "status": "EXPIRED",
          "note": "Pembayaran untuk layanan X",
          "expires_at": 1754820766998,
          "created_at": "YYYY-MM-DDTHH:MM:SS.ssssss",
          "qris_string": "000201010212...630495B0",
          "invoice_id": "inv_6f467369-09d3-4d20-a70b-7ebcaed0bfb8"
        }
        ```
        > Status faktur dapat berupa: `PENDING`, `PAID`, atau `EXPIRED`.
    -   **Respon Faktur Tidak Ditemukan (404 Not Found)**
    -   **Respon Token Tidak Sah (401 Unauthorized)**

### 8. Generasi Gambar QRIS

Menghasilkan gambar QRIS dari sebuah faktur yang dapat dipindai.

-   **Endpoint:** `GET /api/v2/invoices/qris/{invoice_id}`
-   **Headers:**
    -   `Authorization`: `Bearer <token>`
    -   `Accept`: `image/png`
-   **Query Parameters:**
    -   `height`: `integer` (Opsional, nilai default: `1080`)
    -   `width`: `integer` (Opsional, nilai default: `1080`)
-   **Responses:**
    -   **Respon Sukses (200 OK)**: *Response body* akan berisi data biner dari gambar berformat PNG.
-   **Contoh cURL:**
    ```bash
    curl -X GET "https://gateway.rerechanstore.eu.org/api/v2/invoices/qris/c16f9d3c-c740-480e-90c4-41ccef101c53?height=1080&width=1080" \
    -H "Authorization: Bearer e2d307c5-ea01-484a-9bdd-face8d816088" \
    --output qris.png
    ```

---

## 📞 Mekanisme Panggilan Balik (Callback/Webhook)

Apabila `callback_url` telah dikonfigurasi, sistem akan secara otomatis mengirimkan permintaan `POST` ke URL tersebut setiap kali terjadi perubahan status pada sebuah faktur.

#### Struktur Data (Payload) - Faktur Terbayar (PAID)

```json
{
  "id": "wh_6434e9b7-7ba6-4160-b9c3-030529dda148",
  "status": "PAID",
  "amount": 50000,
  "note": "Pembayaran untuk layanan X",
  "created_at": "YYYY-MM-DDTHH:MM:SS.ssssss",
  "paid_at": 1754763948300
}
```

#### Struktur Data (Payload) - Faktur Kedaluwarsa (EXPIRED)

```json
{
  "id": "wh_6434e9b7-7ba6-4160-b9c3-030529dda148",
  "status": "EXPIRED",
  "amount": 50000,
  "note": "Pembayaran untuk layanan X",
  "created_at": "YYYY-MM-DDTHH:MM:SS.ssssss",
  "expired_at": 1754763948300
}
```

---
---
Credits : 
- Telegram @viandrabs
- Github @hikari-work
