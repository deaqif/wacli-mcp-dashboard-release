# wacli MCP Dashboard — Production Distribution

[![PHP Version](https://img.shields.io/badge/PHP-8.2%2B-blue?logo=php)](https://php.net)
[![Protocol](https://img.shields.io/badge/MCP-Model%20Context%20Protocol-emerald)](https://modelcontextprotocol.io)
[![WhatsApp](https://img.shields.io/badge/WhatsApp-Connected-25D366?logo=whatsapp&logoColor=white)](https://whatsapp.com)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Gerbang peribadi (*private self-hosted gateway*) untuk menghubungkan akaun WhatsApp anda terus kepada **Claude AI** melalui protokol **Model Context Protocol (MCP)**. Dilengkapi dengan **Papan Pemuka Web (Web Dashboard)** penuh untuk memantau status, menjadualkan mesej, had kuota harian (*rate limits*), dan log audit.

Direka khusus untuk boleh dipasang pada **Shared Hosting** (DirectAdmin, cPanel), VPS, Docker, mahupun pelayan tempatan (*localhost*).

---

## 📥 Muat Turun (Download)

Sila muat turun fail pengeluaran siap-pakai terkini di halaman **Releases**:

👉 **[Muat Turun Pakej Terkini (`wacli-mcp-dashboard-production.zip`)](../../releases/latest)**

Pakej `.zip` ini mengandungi:
- Frontend React binaan penuh (termasuk User Guide interaktif).
- Backend PHP API & enjin MCP Endpoint.
- Skrip diagnostik & pemasangan automatik untuk Shared Hosting.
- Fail migrasi pangkalan data SQLite bersih.

---

## ⚡ Ujian Keserasian Pelayan (Pre-flight Check)

Sebelum memasang pada akaun hosting anda, jalankan arahan 1-baris ini dalam Terminal SSH hosting untuk mengesahkan keserasian pelayan secara automatik:

```bash
curl -fsSL https://raw.githubusercontent.com/deaqif/wacli-mcp-dashboard/main/deploy/shared-hosting/check-host.sh | sh
```

Skrip ini akan menyemak:
- Seni bina CPU & versi glibc (menentukan sama ada binary `wacli` sedia ada disokong terus).
- Keupayaan kompilasi Go / GCC (jika glibc lama).
- Versi PHP CLI (>= 8.2) dan status sambungan (`pdo_sqlite`, `openssl`, `mbstring`, `json`).
- Fungsi PHP `proc_open` (memastikan tidak disekat dalam `disable_functions`).
- Sambungan rangkaian keluar port 443 ke WhatsApp (`web.whatsapp.com`).

---

## 🚀 Panduan Pemasangan Pantas (Shared Hosting)

### 1. Ekstrak Pakej
Muat naik `wacli-mcp-dashboard-production.zip` ke folder aplikasi anda (contoh: `~/apps/wacli-dashboard` atau `~/domains/wa.example.com/app`) dan ekstrak:
```bash
unzip wacli-mcp-dashboard-production.zip
```

### 2. Pautkan Document Root Web
Pautkan `public_html` domain anda ke folder `backend/public`:
```bash
# Contoh untuk DirectAdmin / cPanel
rm -rf ~/domains/wa.example.com/public_html
ln -s ~/domains/wa.example.com/app/backend/public ~/domains/wa.example.com/public_html
```

### 3. Sediakan Konfigurasi `.env`
Salin templat konfigurasi dan sesuaikan nilai:
```bash
cp .env.example backend/.env
nano backend/.env
```
Tetapkan nilai wajib:
* `APP_URL`: Domain anda (contoh: `https://wa.example.com`).
* `ADMIN_USERNAME`: Username untuk log masuk dashboard.
* `ADMIN_PASSWORD_HASH`: Jana hash kata laluan menggunakan:
  ```bash
  php backend/bin/console.php hash-password
  ```
* `MCP_TOKEN`: Token rahsia unik (32 aksara) untuk sambungan Claude.
* `WACLI_STORE_DIR`: Folder stor sesi wacli (contoh: `/home/<username>/.local/state/wacli`).

### 4. Pasang Pangkalan Data
Jalankan migrasi pangkalan data SQLite:
```bash
php backend/bin/console.php migrate
```

### 5. Pasang & Pautkan wacli (WhatsApp Pairing)
Bina atau muat turun binary `wacli`:
```bash
sh deploy/shared-hosting/wacli-build.sh
```
Pautkan telefon WhatsApp anda dengan mengimbas kod QR di terminal:
```bash
~/bin/wacli auth
```
*(Imbas QR menggunakan WhatsApp di telefon → Linked Devices. Tekan `Ctrl + C` selepas status `Authenticated` keluar).*

### 6. Tetapkan Cron Job (Untuk Mesej Berjadual)
Di panel DirectAdmin / cPanel, tambah satu Cron Job setiap minit:
```bash
* * * * * /usr/local/bin/php /home/<username>/domains/wa.example.com/app/backend/bin/scheduler.php >> /dev/null 2>&1
```

---

## 🤖 Cara Sambungkan ke Claude AI

1. Log masuk ke [Claude.ai](https://claude.ai) → **Settings** → **Connectors**.
2. Tekan **Add custom connector**:
   * **Name**: `WhatsApp`
   * **MCP server URL**: `https://wa.example.com/mcp`
   * **Authentication**: Pilih **`No sign-in`**
3. Di bahagian **Request headers**, tekan **+ Add header**:
   * **Header Name**: `Authorization`
   * **Header Value**: `Bearer TOKEN_MCP_ANDA`
4. Tekan **Save / Connect**.
5. Buka perbualan baharu di Claude, pilih penyambung WhatsApp, dan arahkan Claude:
   > *"Hantar mesej WhatsApp ke +60123456789 beritahu mesyuarat bermula jam 3 petang."*

---

## 📋 Keperluan Sistem (System Requirements)

* **Sistem Operasi**: Linux x86_64 / amd64 atau aarch64 / arm64.
* **PHP**: Versi 8.2 atau 8.3+.
* **Sambungan PHP**: `pdo_sqlite`, `openssl`, `mbstring`, `json`, `posix` (disyorkan).
* **Fungsi PHP**: `proc_open` mesti dibenarkan (tidak tersenarai dalam `disable_functions`).
* **Rangkaian**: Port keluar HTTPS (443) terbuka ke `web.whatsapp.com`.
* **Akaun WhatsApp**: Telefon pintar aktif untuk imbasan pautan peranti (*Linked Devices*).

---

## 🛡️ Keselamatan & Privasi

* **Tiada Pihak Ketiga**: Mesej WhatsApp dihantar terus dari pelayan peribadi anda ke pelayan rasmi WhatsApp tanpa melalui gateway pihak ketiga atau pelayan awam.
* **Had Keselamatan (Guardrails)**: Kawalan had penghantaran harian (*daily cap*), had penghantaran per minit (*rate limit*), dan senarai penerima dibenarkan (*allowlist*) boleh diubah suai melalui menu Settings di Dashboard.
* **Privasi Nombor**: Nombor telefon dikaburkan secara automatik dalam paparan dashboard dan log audit.

---

## 📄 Lesen

Dilesenkan di bawah [Lesen MIT](LICENSE).

*Penafian: Projek ini tidak bergabung, dibiayai, atau disokong oleh WhatsApp Inc. atau Meta Platforms, Inc.*
