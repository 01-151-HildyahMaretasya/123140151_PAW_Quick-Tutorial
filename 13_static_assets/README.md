# Mengelola File Statis (CSS/JS/Images)

## Pengenalan

Web diperlukan juga static assets seperti CSS, JavaScript, dan gambar. Tutorial ini menunjukkan cara mengarahkan aplikasi web ke direktori dimana Pyramid akan melayani file-file statis tersebut.

## Tujuan Tutorial

* Mempublikasikan direktori static assets pada sebuah URL
* Menggunakan Pyramid untuk membantu generate URL ke file dalam direktori tersebut

## Langkah-langkah Pengerjaan

### 1. Menyalin Project Sebelumnya

Copy folder dari tutorial `view_classes` sebagai titik awal:

```bash
cd ..; cp -r view_classes static_assets; cd static_assets
$VENV/bin/pip install -e .
```

### 2. Menambahkan Static View di Konfigurasi

Tambahkan `config.add_static_view` di file `static_assets/tutorial/__init__.py`:

```python
from pyramid.config import Configurator


def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_chameleon')
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.add_static_view(name='static', path='tutorial:static')  # Baris baru
    config.scan('.views')
    return config.make_wsgi_app()
```

Baris `config.add_static_view` ini memberitahu Pyramid untuk melayani file statis dari folder `static` dalam package `tutorial`.

### 3. Menambahkan Link CSS di Template

Edit template `static_assets/tutorial/home.pt` dan tambahkan link CSS di bagian `<head>`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Quick Tutorial: ${name}</title>
    <link rel="stylesheet"
          href="${request.static_url('tutorial:static/app.css') }"/>
</head>
<body>
<h1>Hi ${name}</h1>
</body>
</html>
```

Perhatikan penggunaan `request.static_url()` untuk generate URL ke file CSS.

### 4. Membuat File CSS

Buat folder `static` dan file CSS baru di `static_assets/tutorial/static/app.css`:

```css
body {
    margin: 2em;
    font-family: sans-serif;
}
```

### 5. Menambahkan Functional Test

Tambahkan test untuk memastikan file statis dapat diakses dengan benar:

```python
def test_css(self):
    res = self.testapp.get('/static/app.css', status=200)
    self.assertIn(b'body', res.body)
```

### 6. Menjalankan Test

Jalankan test untuk memastikan semuanya berfungsi:

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

### 7. Menjalankan Aplikasi

Jalankan server development:

```bash
$VENV/bin/pserve development.ini --reload
```

### 8. Verifikasi di Browser

Buka http://localhost:6543/ di browser dan perhatikan perubahan font yang menunjukkan CSS sudah berhasil dimuat.


## Analisis

### Mapping Static Files

Aplikasi WSGI telah diubah untuk memetakan request ke `http://localhost:6543/static/` agar mengarah ke file dan direktori dalam folder `static` di dalam package `tutorial`. Direktori ini berisi file `app.css` yang sudah dibuat.

### Generate URL dengan Helper Pyramid

Di template, link ke CSS bisa saja di-hardcode menjadi `/static/app.css`. Namun, bagaimana jika nanti site dipindahkan ke `/somesite/static/`? Atau developer web mengubah struktur direktori?

Pyramid menyediakan helper yang memberikan fleksibilitas dalam generate URL:

```python
${request.static_url('tutorial:static/app.css')}
```

Kode ini cocok dengan `path='tutorial:static'` pada registrasi `config.add_static_view`. 

**Keuntungan menggunakan `request.static_url`:**
- Tetap sinkron dengan konfigurasi aplikasi
- Memberikan fleksibilitas refactoring di masa depan
- Otomatis menghasilkan URL lengkap yang benar
- Tidak perlu manual update link jika struktur berubah

### Cara Kerja Static View

1. `config.add_static_view(name='static', path='tutorial:static')` mendaftarkan route khusus
2. Parameter `name='static'` menentukan prefix URL (`/static/`)
3. Parameter `path='tutorial:static'` menunjuk ke folder `static` dalam package `tutorial`
4. Semua request ke `/static/*` akan dilayani dari folder tersebut

## Ekstra Kredit (Pertanyaan Lanjutan)

### Perbedaan `request.static_path` dan `request.static_url`

**Pertanyaan**: Ada juga API `request.static_path`. Apa perbedaannya dengan `request.static_url`?

**Jawaban**: 

**`request.static_url`**
- Menghasilkan URL absolut lengkap dengan scheme dan domain
- Contoh output: `http://localhost:6543/static/app.css`
- Cocok digunakan untuk:
  - Link ke resource yang mungkin diakses dari domain berbeda
  - Email yang berisi link ke assets
  - API response yang membutuhkan URL lengkap
  - CDN atau static file server terpisah

**`request.static_path`**
- Menghasilkan path relatif saja (tanpa scheme dan domain)
- Contoh output: `/static/app.css`
- Cocok digunakan untuk:
  - Link internal dalam satu aplikasi/domain yang sama
  - Lebih ringan karena string lebih pendek
  - Bekerja baik dengan relative URL di browser

**Perbandingan praktis:**

```python
# Di template
${request.static_url('tutorial:static/app.css')}
# Output: http://localhost:6543/static/app.css

${request.static_path('tutorial:static/app.css')}
# Output: /static/app.css
```

Untuk kebanyakan kasus website normal, `request.static_path` sudah cukup dan lebih efisien. Gunakan `request.static_url` hanya ketika benar-benar membutuhkan URL absolut.

---

## Kesimpulan

Tutorial ini menunjukkan cara mengelola static assets di Pyramid:
1. Daftarkan static view dengan `config.add_static_view`
2. Buat folder untuk menyimpan file statis
3. Gunakan `request.static_url()` atau `request.static_path()` untuk generate URL
4. Pyramid akan otomatis melayani file dari direktori tersebut

