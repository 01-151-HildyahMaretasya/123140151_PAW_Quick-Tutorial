# Jinja2

## Deskripsi

Tutorial ini menunjukkan cara menggunakan sistem template Jinja2 di aplikasi Pyramid. Jinja2 adalah sistem template populer yang juga dipakai oleh framework Flask dan terinspirasi dari template Django.

## Tujuan Tutorial

* Membuktikan bahwa Pyramid mendukung berbagai sistem template
* Belajar cara memasang add-on (plugin) di Pyramid

## Langkah-langkah Pengerjaan

### 1. Menyalin Project Sebelumnya

Pertama, copy folder dari tutorial sebelumnya (`view_classes`) sebagai titik awal:

```bash
cd ..; cp -r view_classes jinja2; cd jinja2
```

### 2. Menambahkan Dependency di `setup.py`

Tambahkan `pyramid_jinja2` ke daftar package yang dibutuhkan project:

```python
requires = [
    'pyramid',
    'pyramid_chameleon',
    'pyramid_jinja2',  # Baris baru ini
    'waitress',
]
```

File `setup.py` ini seperti daftar belanja - berisi semua library yang dibutuhkan project.

### 3. Install Dependency

Jalankan perintah install untuk mengunduh dan memasang `pyramid_jinja2`:

```bash
$VENV/bin/pip install -e .
```

### 4. Mengaktifkan Jinja2 di Konfigurasi

Edit file `tutorial/__init__.py` dan tambahkan `config.include('pyramid_jinja2')`:

```python
def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_jinja2')  # Baris ini mengaktifkan Jinja2
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.scan('.views')
    return config.make_wsgi_app()
```

### 5. Mengubah Renderer di Views

Edit `tutorial/views.py`, ubah renderer dari `.pt` menjadi `.jinja2`:

```python
@view_defaults(renderer='home.jinja2')  # Ganti ekstensi file
class TutorialViews:
    def __init__(self, request):
        self.request = request

    @view_config(route_name='home')
    def home(self):
        return {'name': 'Home View'}

    @view_config(route_name='hello')
    def hello(self):
        return {'name': 'Hello View'}
```

### 6. Membuat Template Jinja2

Buat file baru `tutorial/home.jinja2`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Quick Tutorial: {{ name }}</title>
</head>
<body>
<h1>Hi {{ name }}</h1>
</body>
</html>
```

Perhatikan sintaks `{{ name }}` - ini cara Jinja2 menampilkan variabel.

### 7. Menjalankan Test

Pastikan semua test masih berjalan dengan baik:

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

### 8. Menjalankan Aplikasi

Jalankan server development:

```bash
$VENV/bin/pserve development.ini --reload
```

### 9. Buka di Browser

Akses http://localhost:6543/ untuk melihat hasilnya.
<img width="959" height="244" alt="image" src="https://github.com/user-attachments/assets/67664b35-0719-4e15-8a46-d12e11da4fbc" />

## Analisis: Bagaimana Cara Kerjanya?

### Proses Memasang Add-on Pyramid

Memasang add-on di Pyramid sangat sederhana:

1. **Install package**: Gunakan pip untuk menginstall package add-on (dalam hal ini `pyramid_jinja2`) ke virtual environment Python
2. **Aktifkan di configurator**: Beritahu Pyramid untuk menjalankan kode setup dari add-on dengan `config.include()`

Ketika memanggil `config.include('pyramid_jinja2')`, Pyramid akan menjalankan kode setup yang memberitahu sistem bahwa sekarang ada "renderer" baru yang bisa digunakan untuk file dengan ekstensi `.jinja2`.

### Perubahan Minimal di Kode

Yang menarik adalah kode view hampir tidak berubah! Hanya perlu:
- Mengganti ekstensi file renderer dari `.pt` (Chameleon) menjadi `.jinja2`
- Sintaks template Chameleon dan Jinja2 untuk menampilkan variabel sederhana ternyata sangat mirip

Ini membuktikan bahwa Pyramid benar-benar fleksibel dan tidak memaksa menggunakan satu template engine tertentu.

## Ekstra Kredit (Pertanyaan Lanjutan)

### 1. Cara Lain Mendeklarasikan Dependency

**Pertanyaan**: Project sekarang bergantung pada `pyramid_jinja2`. Instalasi dilakukan secara manual. Apa cara lain untuk membuat asosiasi ini?

**Jawaban**: Sebenarnya dengan menambahkan `pyramid_jinja2` ke daftar `requires` di `setup.py`, sudah membuat asosiasi otomatis. Jadi ketika orang lain menginstall project dengan `pip install -e .`, mereka akan otomatis mendapatkan `pyramid_jinja2` juga. Tidak perlu install manual satu per satu.

Cara lain yang bisa dilakukan:
- Menggunakan file `requirements.txt` dan menjalankan `pip install -r requirements.txt`
- Menggunakan dependency manager modern seperti Poetry atau Pipenv

### 2. Cara Lain Menginclude Konfigurasi

**Pertanyaan**: Penggunaan `config.include()` merupakan konfigurasi imperatif untuk memuat konfigurasi `pyramid_jinja2`. Apa cara lain untuk memasukkannya ke config?

**Jawaban**: Cara alternatif adalah menggunakan **konfigurasi deklaratif** melalui file `.ini`. Bisa menambahkan setting di file `development.ini`:

```ini
[app:main]
pyramid.includes = pyramid_jinja2
```

Dengan cara ini, tidak perlu menulis `config.include('pyramid_jinja2')` di kode Python. Pyramid akan otomatis memuat add-on tersebut saat aplikasi start berdasarkan setting di file konfigurasi.

Keuntungan pendekatan deklaratif:
- Konfigurasi terpisah dari kode
- Lebih mudah mengubah setting tanpa edit kode
- Bisa punya konfigurasi berbeda untuk development vs production

---

## Kesimpulan

Tutorial ini menunjukkan betapa mudahnya mengganti sistem template di Pyramid. Hanya perlu:
1. Install package add-on
2. Include di configurator
3. Ubah ekstensi file renderer
