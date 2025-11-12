# Application Configuration dengan .ini Files

## Deskripsi

Tutorial ini menunjukkan cara menggunakan command `pserve` dari Pyramid dengan file konfigurasi `.ini` untuk menjalankan aplikasi dengan lebih sederhana dan lebih baik.

Pyramid memiliki konsep configuration yang terpisah dari code (first-class concept). Pendekatan ini bersifat opsional, tetapi membuat Pyramid berbeda dari framework Python lainnya.

### Cara Kerja

Pyramid memanfaatkan library **Setuptools** Python yang menetapkan konvensi untuk:
- Installing Python projects
- Menyediakan "entry points" 

**Entry point** digunakan Pyramid untuk mengetahui lokasi WSGI app.

## Objektif

- Memodifikasi `setup.py` dengan entry point yang menunjuk ke lokasi WSGI app
- Membuat aplikasi yang driven oleh file `.ini`
- Menjalankan aplikasi dengan command `pserve` dari Pyramid
- Memindahkan code ke `__init__.py` package

## Langkah-langkah

**1. Copy Project Sebelumnya**
**2. Update File `setup.py`**
**3. Install/Reinstall Project**
**4. Buat File Konfigurasi `.ini`**
**5. Refactor Startup Code ke `__init__.py`**
**6. Hapus File `app.py` yang Tidak Terpakai**

```bash
rm tutorial/app.py
```
**7. Jalankan Aplikasi**

```bash
$VENV/bin/pserve development.ini --reload
```

**8. Buka Browser**

Akses: `http://localhost:6543/`

## Analisis

### Alur Bootstrap Aplikasi

File `development.ini` dibaca oleh `pserve` untuk bootstrap aplikasi:

1. **pserve membaca `[app:main]`** → menemukan `use = egg:tutorial`
2. **Mencari entry point** → `setup.py` mendefinisikan entry point di lines 12-14 untuk `tutorial:main`
3. **Package `tutorial` memiliki function `main`** → ada di `__init__.py`
4. **Function main() dipanggil** → dengan values dari section tertentu di `.ini` file

### Penjelasan development.ini

#### Section [app:main]

```ini
[app:main]
use = egg:tutorial
```

- Memberitahu pserve untuk menggunakan package `tutorial`
- Menghubungkan ke entry point di `setup.py`

#### Section [server:main]

```ini
[server:main]
use = egg:waitress#main
listen = localhost:6543
```

- **WSGI server configuration** - memilih server (Waitress)
- **Port configuration** - listen di `localhost:6543`
- Waitress sudah diinstall via `setup.py` requires

### Penjelasan Entry Point

```python
entry_points={
    'paste.app_factory': [
        'main = tutorial:main'
    ],
}
```

Format: `nama_entry_point = package:function`

- `main` = nama entry point
- `tutorial` = nama package
- `main` (setelah `:`) = nama function di `__init__.py`

### Fungsi Lain File .ini

Selain bootstrap aplikasi, file `.ini` juga digunakan untuk:

#### 1. Konfigurasi WSGI Server

```ini
[server:main]
use = egg:waitress#main
listen = localhost:6543
```

Wire up server dan port number yang akan digunakan aplikasi.

#### 2. Konfigurasi Python Logging

Pyramid menggunakan Python standard logging. File `.ini` menyediakan configuration values untuk logging, yang menghasilkan console log output saat startup dan setiap request.

### Perubahan dari Step Sebelumnya

**Sebelumnya (package/app.py):**
```python
if __name__ == '__main__':
    with Configurator() as config:
        # ... configuration ...
        app = config.make_wsgi_app()
    serve(app, host='0.0.0.0', port=6543)
```

**Sekarang (ini/tutorial/__init__.py):**
```python
def main(global_config, **settings):
    config = Configurator(settings=settings)
    # ... configuration ...
    return config.make_wsgi_app()
```

**Alasannya:**
- Startup code dipindah ke `__init__.py` (common style di Pyramid)
- WSGI app bootstrapping dipisahkan dari module code
- Configuration dikelola oleh `.ini` file, bukan hardcoded

### Command pserve

```bash
$VENV/bin/pserve development.ini --reload
```

**Flag `--reload`:**
- Memantau filesystem untuk perubahan code
- Otomatis restart aplikasi saat ada perubahan
- Berlaku untuk: Python files, INI file, dll
- Sangat berguna selama development

## Output
<img width="1715" height="526" alt="Cuplikan layar 2025-11-12 155806" src="https://github.com/user-attachments/assets/8c0f4792-8da1-4217-a59d-dfa67982644f" />

## Extra Credit 

### 1. Apakah bisa menghindari configuration dan .ini files dengan Python code saja?

**Ya, bisa.** Configuration dan `.ini` files bersifat opsional di Pyramid.

**Keuntungan menggunakan .ini:**
- Separation of concerns (config terpisah dari code)
- Mudah switch environment (development, staging, production)
- Logging configuration terpusat
- Tidak perlu rebuild/restart untuk config changes

**Alternatif tanpa .ini:**
```python
# Semua configuration dalam code
if __name__ == '__main__':
    settings = {
        'pyramid.reload_templates': True,
        'pyramid.debug_all': True,
    }
    config = Configurator(settings=settings)
    # ... setup routes, views ...
    app = config.make_wsgi_app()
    serve(app, host='localhost', port=6543)
```

### 2. Apakah bisa memiliki multiple .ini files? Mengapa?

**Ya, bisa.** Biasanya ada beberapa file `.ini` untuk environment berbeda:

```
project/
├── development.ini      # Development environment
├── production.ini       # Production environment
├── testing.ini          # Testing/CI environment
└── staging.ini          # Staging environment
```

**Alasan menggunakan multiple .ini:**

1. **Konfigurasi berbeda per environment:**
```ini
# development.ini
pyramid.reload_templates = true
pyramid.debug_all = true

# production.ini
pyramid.reload_templates = false
pyramid.debug_all = false
```

2. **Database connection berbeda:**
```ini
# development.ini
sqlalchemy.url = sqlite:///dev.db

# production.ini
sqlalchemy.url = postgresql://user:pass@prod-server/db
```

3. **Logging level berbeda:**
```ini
# development.ini
level = DEBUG

# production.ini
level = WARNING
```

4. **Port berbeda:**
```ini
# development.ini
listen = localhost:6543

# production.ini
listen = 0.0.0.0:80
```

**Cara penggunaan:**
```bash
# Development
pserve development.ini --reload

# Production
pserve production.ini
```

### 3. Mengapa entry point di setup.py tidak menyebut __init__.py saat declare tutorial:main?

```python
entry_points={
    'paste.app_factory': [
        'main = tutorial:main'  # Tidak ada __init__.py disini
    ],
}
```

**Alasannya:**

Python convention: ketika mengimport dari package, Python secara otomatis mencari di `__init__.py`.

**Yang terjadi:**
- `tutorial:main` berarti "cari function `main` di package `tutorial`"
- Python otomatis mencari di `tutorial/__init__.py`
- Ini sama dengan menulis: `from tutorial import main`

**Analogi:**
```python
# Kedua cara ini sama:
from tutorial import main
from tutorial.__init__ import main  # Redundant, tidak perlu
```

**File structure:**
```
tutorial/
└── __init__.py    # Python otomatis import dari sini
    def main():
        pass
```

### 4. Apa tujuan **settings? Apa arti tanda **?

```python
def main(global_config, **settings):
    config = Configurator(settings=settings)
```

**Tanda `**` (double asterisk) = keyword arguments (kwargs)**

**Fungsi:**
- Menangkap semua keyword arguments yang dikirim ke function
- Dikumpulkan dalam dictionary bernama `settings`
- Bisa berisi 0 atau banyak key-value pairs

**Contoh cara kerja:**
```python
def main(global_config, **settings):
    print(settings)

# Dipanggil dengan:
main({}, debug=True, reload=True, port=6543)

# Output:
# {'debug': True, 'reload': True, 'port': 6543}
```

**Dalam konteks Pyramid:**
- `**settings` menangkap semua configuration dari `.ini` file
- Settings ini kemudian dipass ke `Configurator(settings=settings)`
- Configurator menggunakan settings untuk konfigurasi aplikasi

**Contoh settings dari .ini:**
```ini
[app:main]
use = egg:tutorial
pyramid.reload_templates = true
pyramid.debug_all = true
my_custom_setting = some_value
```

Semua nilai ini akan masuk ke `**settings` dictionary.

**Keuntungan:**
- Fleksibel - bisa tambah settings baru tanpa ubah function signature
- Extensible - settings bisa berbeda per environment
- Clean - tidak perlu hardcode semua possible parameters

