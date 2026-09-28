# Panduan EAS Build — Titik Huni (Frontend)

Langkah-langkah untuk membuat build aplikasi (APK/AAB/iOS) menggunakan [EAS Build](https://docs.expo.dev/build/introduction/).

---

## Persiapan (cukup sekali)

### 1. Install EAS CLI & login

```bash
npm install -g eas-cli
eas login
```

Belum punya akun? Daftar di https://expo.dev/signup.

### 2. Rapikan `app.json` sebelum build pertama

Beberapa nilai masih bawaan template:

- `"android.package": "com.anonymous.frontend"` → ganti ke ID unik, misalnya `com.ruangbumi.titikhuni`.
  > **Penting:** ID ini tidak bisa diubah setelah aplikasi terpasang atau terbit di Play Store.
- `"name": "frontend"` dan `"slug": "frontend"` → ganti ke nama aplikasi, misalnya `"Titik Huni"` dan `"titik-huni"`.
- Jika ingin build iOS, tambahkan:
  ```json
  "ios": { "bundleIdentifier": "com.ruangbumi.titikhuni", ... }
  ```

### 3. Hapus salah satu lockfile

Saat ini ada `yarn.lock` dan `package-lock.json` sekaligus, sehingga EAS bisa memakai package manager yang tidak diharapkan. Pilih satu (misalnya npm) lalu hapus `yarn.lock`.

### 4. Hubungkan project ke EAS

```bash
eas init
eas build:configure
```

- `eas init` menambahkan `projectId` ke `app.json`.
- `eas build:configure` membuat file `eas.json`.

### 5. Atur environment variable (penting)

File `.env` masuk `.gitignore`, sehingga **tidak ikut ter-upload** ke server EAS. Akibatnya `EXPO_PUBLIC_API_URL` akan kosong di hasil build. Pilih salah satu cara berikut:

**Opsi A — EAS env (disarankan):**

```bash
eas env:create --name EXPO_PUBLIC_API_URL --value https://api-kamu.com --environment preview --environment production --visibility plaintext
```

**Opsi B — lewat `eas.json`:** tambahkan `"env"` di tiap profil (lihat langkah 6).

> Pastikan URL API **bisa diakses publik** (HTTPS, bukan `localhost` atau `192.168.x.x`). Jika tidak, aplikasi di HP tidak bisa terhubung ke backend.

### 6. Sesuaikan `eas.json`

Agar profil `preview` menghasilkan APK yang bisa langsung dipasang di HP:

```json
{
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal",
      "android": { "buildType": "apk" },
      "environment": "preview"
    },
    "production": {
      "autoIncrement": true,
      "environment": "production"
    }
  }
}
```

Jika memakai **Opsi B** di langkah 5, tambahkan di dalam profil yang diperlukan:

```json
"env": { "EXPO_PUBLIC_API_URL": "https://api-kamu.com" }
```

---

## Build

### 7. Cek project (opsional, disarankan)

```bash
npx expo-doctor
npx expo install --check
```

### 8. Jalankan build

```bash
# APK untuk testing / dibagikan ke tim
eas build -p android --profile preview

# AAB untuk upload ke Play Store
eas build -p android --profile production

# iOS (perlu akun Apple Developer berbayar)
eas build -p ios --profile production
```

Pada build Android pertama, jawab **Yes** saat ditanya *"Generate a new Android Keystore?"*. EAS akan menyimpan keystore tersebut.

### 9. Download hasil build

Setelah build selesai (biasanya 10–30 menit di antrean gratis), terminal akan menampilkan link dan QR code. Hasilnya juga ada di https://expo.dev → project → **Builds**. Scan QR code di HP Android untuk memasang APK.

---

## Opsional: submit ke store

```bash
eas submit -p android   # perlu akun Google Play Console + service account key
eas submit -p ios
```

---

## Troubleshooting

Jika build gagal, buka halaman build di https://expo.dev lalu periksa log pada tahap yang error (biasanya *Install dependencies* atau *Run gradlew*).
