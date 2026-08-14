# Setup Firebase Auth untuk Admin & Teknisi

Login admin & teknisi sekarang memakai **Firebase Authentication (email/password)**, bukan lagi
username/password hardcoded. Ini **langkah yang wajib kamu lakukan di Firebase Console**
sebelum login bisa jalan.

**File yang sudah di-update di repo:**
- `admin/index.html` → login email/password via Firebase Auth + verifikasi `admin_users`
- `teknisi/index.html` → login email/password via Firebase Auth + cocokkan dengan doc teknisi (field `email`)

---

## 1. Aktifkan provider Email/Password

1. Buka **Firebase Console** → project **`kitaberkahteknik-aa1f5`**
2. Menu kiri: **Build → Authentication → Sign-in method**
3. Klik **Email/Password** → aktifkan (**Enable**) → **Save**

---

## 2. Buat user admin (di Firebase Auth)

1. Tab **Authentication → Users → Add user**
2. Isi email & password admin (mis. `admin@kitaberkahteknik.com` + password kuat)
3. **Catat UID** user admin tersebut (klik user → salin UID)

---

## 3. Tandai user admin (collection `admin_users`)

Aplikasi mengecek akses admin lewat collection `admin_users` (doc id = UID user, field `role`).

1. Buka **Firestore Database → Data**
2. Buat collection **`admin_users`**
3. Tambah dokumen dengan **doc ID = UID admin** (dari langkah 2)
4. Field: `role` = `"admin"`, `name` = `"Admin"`

---

## 4. Buat user teknisi (Firebase Auth)

Untuk setiap teknisi:
1. **Authentication → Add user** → buat akun dengan **email** yang sama dgn field `email` di doc teknisi (mis. `budi@kitaberkahteknik.com`)
2. **Firestore → Data → collection `technicians`** → cari/buat doc teknisi, pastikan field **`email`** persis sama dengan email akun Auth.

> Semua teknisi yang sudah ada di collection `technicians` harus ditambah field `email`
> supaya bisa login. (Admin bisa menambahkan field ini lewat console.)

---

## 5. Perketat Security Rules (aman untuk publik)

Tempel rules ini di **Firestore → Rules → Publish**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Baca harga & ketersediaan boleh publik (untuk web utama)
    match /service_prices/{id} {
      allow read: if true;
      allow write: if isAdmin();
    }
    match /availability/{date} {
      allow read: if true;
      allow write: if isAdmin();
    }

    // Order: dipakai admin & teknisi (authenticated)
    match /orders/{id} {
      allow read, write: if request.auth != null;
    }

    // Teknisi bisa membaca profil sendiri; admin kelola semua
    match /technicians/{id} {
      allow read: if request.auth != null;
      allow create: if isAdmin();
      allow update: if isAdmin() || resource.data.email == request.auth.token.email;
      allow delete: if isAdmin();
    }

    // Hanya admin yg dilihat lewat admin_users
    match /admin_users/{uid} {
      allow read: if request.auth != null && request.auth.uid == uid;
      allow write: if isAdmin();
    }

    function isAdmin() {
      return request.auth != null &&
        get(/databases/$(database)/documents/admin_users/$(request.auth.uid)).data.role == 'admin';
    }
  }
}
```

> ⚠️ **Catatan:** rules dengan `get()` membutuhkan **index/izin baca `admin_users`** untuk
> user terautentikasi. Jika muncul error permission saat akses, simpan rules dgn versi lebih
> terbuka sementara (bagian orders/technicians pakai `request.auth != null`, untuk `service_prices`
> & `availability` read `if true`), lalu kencangkan admin-only pada tulis.

---

## 6. Alur Login Setelah Setup

- **Admin**: `/admin/` → login dengan email & password Auth → aplikasi cek `admin_users` (by UID) → masuk dashboard
- **Teknisi**: `/teknisi/` → login email & password Auth → aplikasi cocokkan `technicians.email` → masuk dashboard

---

## ⚠️ Keamanan (penting untuk klien)
- Password tidak lagi hardcoded — semua login lewat Firebase Auth (aman).
- Rules disarankan dibatasi (authenticated untuk data internal, publik hanya utk harga/availability).
- **Jangan** gunakan rules `if true` pada `orders`/`technicians` di produksi.
