# Setup Firestore untuk Kita Berkah Teknik (Admin & Teknisi)

Portal **admin** dan **teknisi** membutuhkan **Firebase Firestore** dengan dua collection:
- `orders` — daftar pesanan/jadwal service
- `technicians` — daftar teknisi (untuk login portal teknisi)

> ⚠️ **Catatan penting:** Portal admin & teknisi memakai **password plaintext** di data Firestore (sama dengan pola di proyek aslanteknik). Untuk produksi sesungguhnya, sangat disarankan memakai **Firebase Authentication** + **Firestore Security Rules** yang proper. Ini **versi v1 minimal** — untuk internal/internal demo.

---

## 1. Buat project Firebase

1. Buka https://console.firebase.google.com
2. Klik **Add project** → beri nama (mis. `kita-berkah-teknik`) → ikuti langkah.
3. Di project tersebut, buka **Build → Firestore Database** → klik **Create database** → pilih mode **production** atau **test** (untuk cepat: *test mode* selama 30 hari; untuk permanen pakai rules di bawah).
4. Lokasi database: pilih terdekat (mis. `asia-southeast2` / Jakarta).

---

## 2. Salin konfigurasi web

1. Di project Firebase → **Project settings (⚙️) → Your apps → Web app (</>)**.
2. Daftarkan aplikasi, beri nama mis. `admin-teknisi`.
3. Firebase SDK config akan tampil seperti ini:
```js
const firebaseConfig = {
  apiKey: "AIza...",
  authDomain: "kita-berkah-teknik.firebaseapp.com",
  projectId: "kita-berkah-teknik",
  storageBucket: "kita-berkah-teknik.firebasestorage.app",
  messagingSenderId: "1234567890",
  appId: "1:1234567890:web:abcdef"
};
```
4. **Isi nilai tersebut ke dalam `tset` file berikut** (ganti nilai placeholder `API_KEY_ANDA`, `PROJECT_ID`, `SENDER_ID`, `APP_ID`):
   - `admin/index.html`
   - `teknisi/index.html`

---

## 3. Atur Security Rules

Buka **Firestore Database → Rules**, lalu tempel rules berikut. (Ganti `[PROJECT_ID]` — untuk demo/internal boleh kosongkan kunci admin.)

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Pisahkan baca/tulis berdasarkan task. Untuk v1 internal, buka read/write
    // sedangkan login dicek di client (pola sederhana).
    // ⚠️ GANTI dengan auth rules jika sudah pakai Firebase Auth.
    match /orders/{id} {
      allow read, write: if true;
    }
    match /technicians/{id} {
      allow read, write: if true;
    }
  }
}
```

> **Keamanan lebih baik (tanpa auth):** biarkan rules terbuka hanya untuk durasi pengembangan/demo. Untuk produksi, aktifkan Firebase Auth & tulis ulang rules dengan `request.auth`.

---

## 4. Seed data awal (jalankan di konsol Firestore)

Buka **Firestore Database → Data**, buat collection `technicians`, lalu tambahkan dokumen contoh:

**Field dokumen teknisi:**
| Field | Nilai contoh |
|---|---|
| `name` | `Budi` |
| `tech_id` | `T001` |
| `phone` | `08123456789` |
| `password` | `rahasia123` |
| `is_active` | `true` |
| `created_at` | *(server timestamp)* |

Collection `orders` contoh:
| Field | Nilai contoh |
|---|---|
| `customer_name` | `Andi` |
| `phone` | `081298765432` |
| `address` | `Jl. Contoh No.1, Jakarta` |
| `service_type` | `Cuci AC 1 PK` |
| `services_text` | `Cuci AC 1/2–1 PK x2` |
| `scheduled_date` | `2026-02-10` |
| `assigned_to` | `T001` |
| `task_status` | `belum` |

---

## 5. Alur kerja

- **Admin:** buka `/admin/` → login (default `admin` / password ada di bagian atas `admin/index.html`, ubah sesuai keinginan) → tambah/edit/hapus pesanan + kelola teknisi.
- **Teknisi:** buka `/teknisi/` → login pakai `tech_id` + `password` (dikelola admin) → lihat pesanan yang di-assign → tandai **Selesai**.

---

## 6. Catatan produksi (langkah berikutnya)

- Gunakan **Firebase Authentication** untuk login admin/teknisi yang aman.
- Terapkan **Firestore Security Rules** berbasis `request.auth.uid` (bukan open rules).
- Tidak boleh ada password plaintext di database publik — gunakan hash (mis. Firebase Functions + bcrypt).
