# 🫁 Lung Disease Detection System — Deep Learning (DenseNet-169)

A web-based system for diagnosing lung diseases from chest X-ray images and patient symptoms using a **DenseNet-169** deep learning model. The system integrates a **Laravel (PHP)** backend with a **Flask (Python)** microservice that runs the TensorFlow/Keras model.

---

## ✨ Features

- **User authentication** — register, login, logout using Laravel Breeze/Sanctum style.
- **X-ray image upload** — JPG/PNG up to 5MB with validation and preview.
- **Symptom selection** from a predefined list (batuk, sesak napas, nyeri dada, dll).
- **Multimodal diagnosis** combining image + symptoms via a trained DenseNet-169 model.
- **Real-time prediction** using a Flask API that returns probabilities for 11 lung conditions:
  - Cardiomegaly, Infiltration, Effusion, Fibrosis, Consolidation, Emphysema, Atelectasis, Bronchitis, Mass, Pneumothorax, Bronkiektasis.
- **Result display** with diagnosis (threshold ≥ 0.5) and probability scores.
- **User dashboard** with diagnosis history for logged-in users.
- **Data persistence** using MySQL/PostgreSQL via Eloquent ORM.
- **Test functional endpoint** to verify Flask API integration.

---

## 🧠 Model Architecture

| Attribute        | Detail                                                                 |
|------------------|------------------------------------------------------------------------|
| **Base model**   | DenseNet-169 pre-trained on ImageNet                                   |
| **Image input**  | Chest X-ray image (224×224 RGB)                                        |
| **Symptom input**| 13 binary features (e.g., Batuk, Sesak_napas, Demam, dll)             |
| **Output**       | 11-class softmax probabilities                                          |
| **Model file**   | `multimodal_medical_v3_auc0.7586_20250805_134028.h5` (AUC: 0.7586)   |
| **API format**   | Multipart form data with `xray_image` and `symptoms[]` fields          |

---

## 🛠️ Tech Stack

| Layer            | Technologies                                                                      |
|------------------|-----------------------------------------------------------------------------------|
| **Backend**      | Laravel 9 (PHP 8.0.2+), Eloquent ORM, Blade Templates                            |
| **Frontend**     | Vite, Axios, Bootstrap                                                            |
| **API Gateway**  | Laravel HTTP Client (Guzzle) → Flask                                              |
| **Deep Learning**| Python 3.8+, TensorFlow 2.x, Keras, Pillow, Flask                                |
| **Database**     | MySQL / PostgreSQL / SQLite (configurable via `.env`)                             |
| **Web Server**   | Laravel Artisan (dev) or Nginx/Apache (production)                                |

---

## 📋 Prerequisites

Pastikan dependensi berikut sudah terinstall:

- **PHP** >= 8.0.2 dengan ekstensi: `BCMath`, `Ctype`, `Fileinfo`, `JSON`, `Mbstring`, `OpenSSL`, `PDO`, `Tokenizer`, `XML`
- **Composer** — PHP dependency manager
- **Node.js** & **NPM** — untuk Vite asset compilation
- **Python** >= 3.8 dengan `pip`
- **Database** — MySQL 5.7+ / PostgreSQL 10+ / SQLite 3
- **Git** (opsional)

---

## 🚀 Instalasi

### 1. Clone atau ekstrak project

```bash
cd D:/backup/1/lung-diagnosis   # sesuaikan dengan root project Anda
```

### 2. Install dependensi PHP

```bash
composer install
```

### 3. Install dependensi JavaScript & build assets

```bash
npm install
npm run build   # atau npm run dev untuk development
```

### 4. Konfigurasi environment

Salin file environment dan generate application key:

```bash
cp .env.example .env
php artisan key:generate
```

Edit `.env` untuk mengkonfigurasi database:

```dotenv
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=lung_diagnosis
DB_USERNAME=root
DB_PASSWORD=
```

Sesuaikan juga `APP_NAME`, `APP_URL`, dan variabel lainnya.

### 5. Jalankan migrasi database

```bash
php artisan migrate
```

> **Catatan:** Migrasi bawaan hanya membuat tabel default Laravel. Anda perlu membuat migrasi tambahan untuk:
> - `diagnoses` (user_id, diagnosis_date, dll)
> - `diagnosis_labels` (diagnosis_id, label_name, label_score)
> - `diagnosis_symptoms` (diagnosis_id, symptom_name)
>
> Jika belum ada, jalankan `php artisan make:migration create_diagnoses_table` dan definisikan skemanya.

### 6. Setup environment Python (Flask API)

```bash
cd deep-learning-model

# Buat virtual environment (disarankan)
python -m venv venv
source venv/bin/activate      # Linux/Mac
venv\Scripts\activate         # Windows

# Install dependensi
pip install tensorflow flask pillow numpy
```

> Pastikan file model `multimodal_medical_v3_auc0.7586_20250805_134028.h5` berada di direktori yang sama dengan `predict.py`.

### 7. Buat storage link

```bash
php artisan storage:link
```

---

## ▶️ Menjalankan Aplikasi

Jalankan **dua service** secara bersamaan di terminal terpisah:

### Terminal 1 — Flask API (deep learning model)

```bash
cd deep-learning-model
python predict.py
```

Flask server akan berjalan di `http://127.0.0.1:5000`. Output yang diharapkan:

```
 * Running on http://0.0.0.0:5000
```

### Terminal 2 — Laravel development server

```bash
php artisan serve
```

Laravel app akan berjalan di `http://127.0.0.1:8000`.

### Buka di browser

👉 [http://127.0.0.1:8000](http://127.0.0.1:8000)

---

## 🧪 Alur Penggunaan

1. **Register** akun baru atau **login**.
2. Di halaman utama, **upload** gambar chest X-ray (JPG/PNG, maks 5MB).
3. Sistem mengarahkan ke halaman pemilihan gejala (`/symptom`).
4. **Pilih gejala** yang relevan (boleh lebih dari satu).
5. **Submit** — sistem akan:
   - Mengirim gambar + gejala ke Flask API (`/predict`)
   - Menerima probabilitas dan diagnosis dengan confidence ≥ 0.5
   - Menyimpan hasil ke database (jika sudah login)
   - Menampilkan hasil di halaman `/result`
6. Kunjungi **User Dashboard** (`/user`) untuk melihat riwayat diagnosis.

---

## 🔗 Daftar Route

| Method | URI               | Controller Method                        | Keterangan                         |
|--------|-------------------|------------------------------------------|------------------------------------|
| GET    | `/`               | `home` view                              | Halaman utama (form upload)        |
| GET    | `/login`          | `AuthController@showLogin`               | Halaman login                      |
| POST   | `/login`          | `AuthController@login`                   | Autentikasi pengguna               |
| GET    | `/register`       | `AuthController@showRegister`            | Halaman registrasi                 |
| POST   | `/register`       | `AuthController@register`                | Buat akun baru                     |
| POST   | `/upload`         | `DiagnosisController@uploadXray`         | Upload gambar X-ray                |
| GET    | `/symptom`        | `DiagnosisController@showSymptomForm`    | Form pemilihan gejala              |
| POST   | `/diagnosis`      | `DiagnosisController@processDiagnosis`   | Proses diagnosis                   |
| GET    | `/result`         | `DiagnosisController@showResult`         | Tampilkan hasil diagnosis          |
| GET    | `/user`           | `DiagnosisController@showUserDashboard`  | Dashboard & riwayat pengguna       |
| GET    | `/test-functional`| `DiagnosisController@testFunctional`     | Uji integrasi Flask API            |

---

## 🗂️ Struktur Project

```
lung-diagnosis/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── DiagnosisController.php   # Logika utama: memanggil Flask API
│   │   │   ├── AuthController.php
│   │   │   └── UserController.php
│   │   └── Kernel.php
│   └── Models/                           # Eloquent models (User, Diagnosis, dll)
├── config/
│   └── app.php                           # Konfigurasi app (timezone = UTC)
├── database/
│   └── migrations/                       # Schema migrations
├── deep-learning-model/                  # ⚠️ Python microservice
│   ├── predict.py                        # Flask API endpoint
│   ├── multimodal_medical_v3_auc0.7586_20250805_134028.h5
│   ├── evaluate_model.py                 # Script evaluasi (opsional)
│   └── test_images/                      # Sample gambar untuk testing
├── public/
│   └── storage/                          # Symlink ke storage/app/public/uploads
├── resources/
│   └── views/                            # Blade templates (home, symptom, result, user)
├── routes/
│   ├── web.php                           # Web routes
│   └── api.php                           # API routes (auth:sanctum)
├── storage/
│   └── app/public/uploads/               # Gambar X-ray yang diupload
├── .env                                  # Environment variables
├── composer.json                         # PHP dependencies
├── package.json                          # Node dependencies (Vite)
└── artisan                               # Laravel CLI
```

---

## ⚙️ Detail Konfigurasi

### Mapping Gejala (Form → Model)

`DiagnosisController` memetakan nama gejala dari form (Bahasa Indonesia) ke key yang diharapkan model:

| Form Field (Indonesia) | Model Key                   |
|------------------------|-----------------------------|
| `batuk`                | `Batuk`                     |
| `sesak_napas`          | `Sesak_napas`               |
| `nyeri_dada`           | `Nyeri_dada`                |
| `demam`                | `Demam`                     |
| `lemas`                | `Kelelahan_penurunan_bb`    |
| `batuk_darah`          | `Batuk_darah`               |
| `mual`                 | `Kembung_perut`             |
| ...dan lainnya         | (lihat `$symptomMapping`)   |

Mapping lengkap tersedia di `private $symptomMapping` dalam `DiagnosisController.php`.

### Flask API Endpoint

| Atribut           | Detail                                      |
|-------------------|---------------------------------------------|
| **URL**           | `http://127.0.0.1:5000/predict`             |
| **Method**        | `POST`                                      |
| **Content-Type**  | `multipart/form-data`                       |
| **Fields**        | `xray_image` (file), `symptoms[]` (array)   |

**Contoh response:**

```json
{
  "diagnosis": ["Cardiomegaly", "Effusion"],
  "probabilities": {
    "Cardiomegaly": 0.87,
    "Infiltration": 0.12,
    "Effusion": 0.73,
    "..."
  }
}
```

---

## 🧪 Testing Flask API Secara Mandiri

**Menggunakan curl:**

```bash
curl -X POST http://127.0.0.1:5000/predict \
  -F "xray_image=@/path/to/chest_xray.jpg" \
  -F "symptoms[]=Batuk" \
  -F "symptoms[]=Sesak_napas"
```

**Menggunakan endpoint bawaan Laravel:**

```
GET /test-functional
```

Endpoint ini mengirimkan sample image (`storage/app/public/sample_1.png`) dengan gejala yang telah ditentukan, kemudian menyimpan hasilnya ke `storage/app/public/test-functional-result.csv`.

---

## 🐛 Troubleshooting

### 1. Flask API tidak dapat diakses
- Pastikan Python server berjalan di port `5000`.
- Periksa firewall atau masalah CORS.
- URL Flask di `DiagnosisController.php` di-hardcode sebagai `http://127.0.0.1:5000/predict`. Ubah jika diperlukan.

### 2. File model tidak ditemukan
- Flask script mengharapkan file `multimodal_medical_v3_auc0.7586_20250805_134028.h5` di direktori yang sama.
- Verifikasi nama dan path file.

### 3. Gambar X-ray tidak tampil di halaman result
- Jalankan `php artisan storage:link` untuk membuat symbolic link dari `public/storage` ke `storage/app/public`.
- Pastikan file yang diupload ada di `storage/app/public/uploads/`.

### 4. Error database
- Jalankan `php artisan migrate` untuk membuat semua tabel yang diperlukan.
- Jika tabel diagnosis belum ada, buat secara manual via migrations.

### 5. Error "Gagal menerima respon dari model Python"
- Cek Laravel logs: `storage/logs/laravel.log`
- Verifikasi output Flask API (jalankan `python predict.py` dan amati console).

---

## 📄 Lisensi

Project ini dilisensikan di bawah **MIT License** (sesuai `composer.json`). Model deep learning bersifat proprietary dan digunakan untuk keperluan riset/demo.

---

## 👥 Kontributor

**Muhammad Rizky Sandyra** — dibangun sebagai bagian dari proyek riset deteksi penyakit paru-paru.

---

> **Last updated:** May 2026  
> **Laravel Version:** 9.x  
> **Python Version:** 3.8+  
> **Model Architecture:** DenseNet-169 (Multimodal)
