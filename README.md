# C Key Web (siap deploy Vercel)

Versi web dari bot **keybot (c key)**. Memakai data yang sama (`key.json` di GitHub), jadi key buatan web dan key buatan bot saling berlaku, termasuk untuk `license.js` di ubot.

## Dua kredensial yang berbeda
| Untuk | Format | Dibuat lewat bot | Dipakai di |
|---|---|---|---|
| Userbot Leo Brann | key `LEOX-xxxxx-...` | `/ckey` | `license.js` ubot |
| Login web ini | username + password `WEB-xxxxx-...` | `/cweb` | halaman login web |

Keduanya disimpan di `key.json` (key di bagian `keys`, login web di bagian `webs`), tapi tidak saling berlaku.

## Fitur
- **Login** (wajib) pakai username + password dari `/cweb`. Akun dicabut / expired -> sesi langsung berhenti.
- Akun biasa bertindak sebagai pembuatnya (role & kuota sama), akun `member` hanya Add Token & Cek Key.
- **C Key**: buat key ubot (`LEOX`) 7 / 30 / 90 hari, permanen, atau custom + catatan. Kuota per role sama dengan keybot.
- **Key Saya** + revoke, **Cek Key**.
- **Add Token**: simpan token Vercel / GitHub / bot Telegram / lainnya. Diverifikasi ke provider, disimpan **terenkripsi AES-256-GCM** di `tokens.json`, tampil disamarkan, per akun web.
- **Admin** (Admin & Developer): statistik, atur role, hapus role, tambah kuota (developer).

## Deploy ke Vercel
1. Upload folder ini ke repo GitHub baru (**PRIVATE** disarankan), atau pakai `vercel` CLI.
2. Vercel > Add New Project > pilih repo. Framework preset: **Other**. Tidak perlu build command.
3. Isi **Environment Variables** (lihat `.env.example`):
   | Nama | Isi |
   |---|---|
   | `GITHUB_TOKEN` | Fine-grained token, repo `leoganteng29/dbUbot`, **Contents: Read and write** (boleh sama dengan token keybot) |
   | `SESSION_SECRET` | acak panjang: `openssl rand -hex 32` |
   | `DEVELOPER_IDS` | ID Telegram developer, pisah koma (default `6146077243`) |
   | `GITHUB_REPO`, `GITHUB_BRANCH`, `GITHUB_FILE` | default `leoganteng29/dbUbot`, `main`, `key.json` |
   | `KEY_PREFIX` | default `LEOX` |
   | `BRAND`, `TIMEZONE_OFFSET`, `TIMEZONE_LABEL`, `DEFAULT_DAYS`, `SESSION_HOURS` | opsional |
   | `TOKEN_ENC_KEY` | opsional, kunci enkripsi token (kosong = pakai `SESSION_SECRET`) |
4. Deploy. Selesai.

## Login pertama
Di bot (akun developer) kirim `/cweb permanen leo`, simpan username & password yang muncul, lalu login di web.

## Keamanan
- **Tidak ada token/rahasia di kode.** Semua dari Environment Variables Vercel.
- Cookie sesi `HttpOnly; Secure; SameSite=Strict`, ditandatangani HMAC, 12 jam.
- Batas percobaan login (8 salah = kunci 5 menit per IP, best-effort per instance serverless).
- `tokens.json` ada di repo yang sama dengan `key.json`. Karena berisi token (walau terenkripsi), gunakan repo **private** untuk data. Kalau repo `dbUbot` public dan `license.js` membaca lewat raw URL, pakai `GITHUB_TOKENS_FILE` + repo private lain, atau jadikan repo private dan isi `READ_TOKEN` di `license.js`.
- Struktur data `key.json` tidak diubah, jadi keybot & ubot tetap kompatibel.

## Struktur
```
public/index.html     UI (login, C Key, Key Saya, Add Token, Cek Key, Admin)
api/login.js          POST login (akun dari /cweb)
api/logout.js         POST logout
api/me.js             GET info sesi + kuota
api/keys.js           GET list, POST buat/cek, DELETE revoke
api/tokens.js         GET list, POST tambah, DELETE hapus (Add Token)
api/admin.js          role, kuota, statistik
api/_lib/core.js      logika bersama (port keybot.js)
vercel.json           header keamanan + konfigurasi fungsi
```
