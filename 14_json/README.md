# Tutorial Pyramid: AJAX Development dengan JSON Renderer

## Pengenalan

Aplikasi web modern bukan hanya tentang render HTML. Halaman dinamis sekarang menggunakan JavaScript untuk update UI di browser dengan request data dari server dalam format JSON. Pyramid mendukung ini dengan JSON renderer.

Seperti yang sudah dipelajari di tutorial templating, deklarasi view bisa menentukan renderer. Output dari view kemudian diproses melalui renderer, yang generate dan return response. Sebelumnya menggunakan Chameleon renderer, lalu Jinja2 renderer.

Namun, renderer tidak terbatas pada template yang generate HTML saja. Pyramid menyediakan JSON renderer yang mengambil data Python, serialize menjadi JSON, dan melakukan fungsi lain seperti set content type. Bahkan bisa membuat custom renderer sendiri atau extend built-in renderer untuk kebutuhan aplikasi yang unik.

## Tujuan Tutorial

* Memahami cara kerja JSON renderer di Pyramid
* Membuat endpoint API yang return data JSON
* Menggunakan multiple decorator pada satu view method

## Langkah-langkah Pengerjaan

### 1. Menyalin Project Sebelumnya

Copy folder dari tutorial `view_classes` sebagai titik awal:

```bash
cd ..; cp -r view_classes json; cd json
$VENV/bin/pip install -e .
```

### 2. Menambahkan Route untuk JSON

Tambahkan route baru `hello_json` di file `json/tutorial/__init__.py`:

```python
from pyramid.config import Configurator


def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_chameleon')
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.add_route('hello_json', '/howdy.json')  # Route baru untuk JSON
    config.scan('.views')
    return config.make_wsgi_app()
```

### 3. Menambahkan Decorator untuk JSON di View

Edit `views.py` dan "stack" decorator tambahan pada method `hello`:

```python
from pyramid.view import (
    view_config,
    view_defaults
    )


@view_defaults(renderer='home.pt')
class TutorialViews:
    def __init__(self, request):
        self.request = request

    @view_config(route_name='home')
    def home(self):
        return {'name': 'Home View'}

    @view_config(route_name='hello')
    @view_config(route_name='hello_json', renderer='json')  # Decorator tambahan
    def hello(self):
        return {'name': 'Hello View'}
```

Perhatikan ada **dua decorator** `@view_config` pada method `hello`:
- Decorator pertama: untuk route `hello` dengan renderer default (home.pt)
- Decorator kedua: untuk route `hello_json` dengan renderer JSON

### 4. Menambahkan Functional Test

Tambahkan test baru di akhir file `json/tutorial/tests.py`:

```python
def test_hello_json(self):
    res = self.testapp.get('/howdy.json', status=200)
    self.assertIn(b'{"name": "Hello View"}', res.body)
    self.assertEqual(res.content_type, 'application/json')
```

### 5. Menjalankan Test

Jalankan test untuk memastikan semuanya berfungsi:

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

### 6. Menjalankan Aplikasi

Jalankan server development:

```bash
$VENV/bin/pserve development.ini --reload
```

### 7. Verifikasi di Browser

Buka http://localhost:6543/howdy.json di browser dan akan muncul response JSON:
<img width="959" height="277" alt="image" src="https://github.com/user-attachments/assets/276e1657-9c1f-42f9-a89c-6addfdeaf625" />

## Analisis: Bagaimana Cara Kerjanya?

### Data-Oriented View Layer

Sebelumnya sudah mengubah view function dan method untuk return data Python. Perubahan ke data-oriented view layer ini membuat penulisan test lebih mudah, memisahkan templating dari logika view.

### JSON Renderer di Pyramid

Karena Pyramid memiliki JSON renderer selain templating renderer, sangat mudah untuk return JSON. Dalam kasus ini, view yang sama tetap dipertahankan dan diatur agar return encoding JSON dari data view.

### Cara Implementasi

Implementasi dilakukan dengan:

1. **Menambahkan route** untuk mapping `/howdy.json` ke route name
2. **Menyediakan `@view_config`** yang mengasosiasikan route name tersebut dengan view yang sudah ada
3. **Override view defaults** dalam view config yang menyebutkan route `hello_json`, sehingga ketika route cocok, akan menggunakan JSON renderer bukan template renderer `home.pt`

### Multiple Decorators (Stacking)

Konsep penting di sini adalah **stacking decorators**. Satu method view bisa memiliki multiple decorator `@view_config`:

```python
@view_config(route_name='hello')              # Untuk HTML
@view_config(route_name='hello_json', renderer='json')  # Untuk JSON
def hello(self):
    return {'name': 'Hello View'}
```

Artinya method `hello()` bisa dipanggil melalui dua route berbeda:
- `/howdy` → return HTML (menggunakan template home.pt)
- `/howdy.json` → return JSON (menggunakan JSON renderer)

### Content Type Otomatis

JSON renderer tidak hanya serialize data ke JSON, tapi juga:
- Set `Content-Type: application/json` secara otomatis
- Handle encoding dengan benar
- Return response yang siap dikonsumsi oleh AJAX client

### Alternatif: View Predicates

Untuk aplikasi AJAX murni, bisa menggunakan route yang sama dengan memanfaatkan **view predicates** Pyramid untuk match pada header `Accept:` yang dikirim oleh implementasi AJAX modern. Ini membuat URL lebih clean tanpa perlu `.json` extension.

## Keterbatasan dan Solusi

### Python JSON Encoder

JSON renderer Pyramid menggunakan base Python JSON encoder, sehingga mewarisi kekuatan dan kelemahannya.

**Contoh masalah**: Python tidak bisa native JSON encode object DateTime.

**Solusi yang tersedia**:
- Extend JSON renderer dengan custom renderer
- Menggunakan custom JSON serializer
- Convert DateTime ke string sebelum return dari view
- Menggunakan library pihak ketiga seperti `simplejson`

### Custom JSON Renderer

Pyramid memungkinkan untuk membuat custom JSON renderer dengan logika khusus, misalnya:
- Handle DateTime secara otomatis
- Format Decimal dengan benar
- Serialize custom object
- Menambahkan metadata tambahan

---

## Kesimpulan

Tutorial ini menunjukkan cara mudah membuat JSON API di Pyramid:
1. Tambahkan route untuk endpoint JSON
2. Stack decorator `@view_config` dengan `renderer='json'`
3. View yang sama bisa melayani HTML dan JSON
4. JSON renderer otomatis handle serialization dan content type
