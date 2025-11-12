# Basic Web Handling dengan Views di Pyramid

## Pendahuluan

Dalam aplikasi web, "view" adalah cara utama untuk menerima web request dan mengembalikan response. Sejauh ini, semua kode (view function, registrasi, routing, WSGI launcher) ditempatkan dalam satu file. Ini tidak ideal untuk aplikasi yang lebih besar.

Tutorial ini menunjukkan cara mengorganisir aplikasi dengan memindahkan views ke module terpisah dan menggunakan decorator untuk konfigurasi deklaratif.

## Konsep Views di Pyramid

**View** adalah function yang:
- Menerima request sebagai parameter
- Memproses request tersebut
- Mengembalikan response

### Sebelumnya (Semua dalam satu file)
```
tutorial/
  __init__.py  ← View function, routing, config, semua ada di sini
  tests.py
```

### Sekarang (Terorganisir)
```
tutorial/
  __init__.py  ← Hanya config dan routing
  views.py     ← Semua view functions
  tests.py     ← Tests untuk semua views
```

## Setup Project

### 1. Copy dari Step Sebelumnya
### 2. Refactor Kode

#### File: `tutorial/__init__.py` (Lebih Pendek)

**Perubahan utama**:
- Tidak ada view function di sini lagi
- `config.scan('.views')` mencari decorators di module views

#### File: `tutorial/views.py` (Baru)

```python
from pyramid.response import Response
from pyramid.view import view_config


# First view, available at http://localhost:6543/
@view_config(route_name='home')
def home(request):
    return Response('<body>Visit <a href="/howdy">hello</a></body>')


# /howdy
@view_config(route_name='hello')
def hello(request):
    return Response('<body>Go back <a href="/">home</a></body>')
```

### 3. Update Tests
### 4. Jalankan Tests

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

### 5. Jalankan Aplikasi

```bash
$VENV/bin/pserve development.ini --reload
```

Buka di browser:
- http://localhost:6543/ (home view)
- http://localhost:6543/howdy (hello view)

---

Output :
<img width="959" height="323" alt="image" src="https://github.com/user-attachments/assets/2e9e5937-70c9-4ecf-8a1e-7279fb8482e5" />

<img width="959" height="255" alt="image" src="https://github.com/user-attachments/assets/bf44567e-be80-47ce-aae2-81a0f578e313" />


## ANALISIS 

### Perubahan Struktur Kode

Kode sekarang lebih terorganisir dengan pemisahan concern:

| File | Responsibility |
|------|----------------|
| `__init__.py` | Application startup, routing configuration |
| `views.py` | View functions dan view registration |
| `tests.py` | Unit tests dan functional tests |

**Breakdown**:

1. **`config.add_route('home', '/')`**: 
   - Membuat route bernama 'home'
   - Map ke URL '/'
   - Route name akan digunakan di decorator

2. **`config.add_route('hello', '/howdy')`**:
   - Membuat route bernama 'hello'
   - Map ke URL '/howdy'

3. **`config.scan('.views')`**:
   - Scan module `views.py` untuk mencari decorators
   - Secara otomatis register semua view yang memiliki `@view_config`
   - Ini adalah "declarative configuration"

### View Configuration dengan Decorator

```python
@view_config(route_name='home')
def home(request):
    return Response('<body>Visit <a href="/howdy">hello</a></body>')
```

**Breakdown**:

- **`@view_config(route_name='home')`**: 
  - Decorator yang mendaftarkan view ke route 'home'
  - Equivalent dengan `config.add_view(home, route_name='home')`
  - Lebih deklaratif dan mudah dibaca

- **`def home(request)`**:
  - View function yang menerima request object
  - Return Response object dengan HTML

### Imperative vs Declarative Configuration

#### Imperative Configuration (Cara Lama)

```python
# Di __init__.py
def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.add_route('home', '/')
    config.add_view(home, route_name='home')  # Explicit
    return config.make_wsgi_app()
```

#### Declarative Configuration (Cara Baru)

```python
# Di views.py
@view_config(route_name='home')  # Decorator
def home(request):
    return Response('...')

# Di __init__.py
def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.add_route('home', '/')
    config.scan('.views')  # Auto-discover decorators
    return config.make_wsgi_app()
```

**Kedua cara menghasilkan konfigurasi yang sama**, hanya cara penulisannya berbeda. Declarative configuration lebih populer karena:
- View function dan konfigurasinya ada di satu tempat
- Lebih mudah dibaca dan di-maintain
- Lebih idiomatis dalam Pyramid

### Route Name vs URL vs View Name

Contoh dari kode:

```python
# Di __init__.py
config.add_route('hello', '/howdy')

# Di views.py
@view_config(route_name='hello')
def hello(request):
    return Response('...')
```

| Komponen | Nilai | Penjelasan |
|----------|-------|------------|
| **Route Name** | `'hello'` | Internal identifier untuk route |
| **URL Pattern** | `'/howdy'` | URL yang user akses di browser |
| **View Function Name** | `hello` | Nama function di Python |

Ketiganya **bisa berbeda**! Contoh:

```python
# Route name: greeting
# URL: /say-hello
# View function: my_greeting_view
config.add_route('greeting', '/say-hello')

@view_config(route_name='greeting')
def my_greeting_view(request):
    return Response('Hello!')
```

Ini memberikan fleksibilitas dalam penamaan dan organisasi kode.

### Flow Request ke Response

1. **User akses URL**: http://localhost:6543/howdy
2. **Pyramid routing**: URL '/howdy' di-match dengan route 'hello'
3. **View lookup**: Pyramid cari view yang terdaftar untuk route 'hello'
4. **View execution**: Function `hello(request)` dipanggil
5. **Response returned**: Response object dikembalikan ke user

### Navigation antar Views

```python
# home view - link ke hello
def home(request):
    return Response('<body>Visit <a href="/howdy">hello</a></body>')

# hello view - link ke home
def hello(request):
    return Response('<body>Go back <a href="/">home</a></body>')
```

Kedua view saling terhubung dengan HTML links, memungkinkan navigasi bolak-balik.

### Testing Strategy

Test sekarang mencakup kedua views:

**Unit Tests**:
```python
def test_home(self):
    from .views import home
    request = testing.DummyRequest()
    response = home(request)
    self.assertEqual(response.status_code, 200)
    self.assertIn(b'Visit', response.body)
```

**Functional Tests**:
```python
def test_home(self):
    res = self.testapp.get('/', status=200)
    self.assertIn(b'<body>Visit', res.body)
```

Setiap view di-test dengan dua cara: unit test (terisolasi) dan functional test (end-to-end).

---

## EXTRA CREDIT

### 1. Apa Arti Dot (.) di `.views`?

**Pertanyaan**: Apa yang dimaksud dengan dot di `config.scan('.views')`?

**Jawaban**:

```python
config.scan('.views')
```

Dot (`.`) adalah **relative import** yang merujuk pada package saat ini.

#### Penjelasan Detail

**Struktur package**:
```
tutorial/
  __init__.py     ← Kode config.scan('.views') ada di sini
  views.py        ← Module yang akan di-scan
```

**`.views` berarti**:
- `.` = package saat ini (`tutorial`)
- `views` = module `views.py` di dalam package tersebut
- Lengkapnya: `tutorial.views`

#### Perbandingan Cara Scan

```python
# Cara 1: Relative import dengan dot
config.scan('.views')
# Scan: tutorial.views

# Cara 2: Absolute import tanpa dot
config.scan('tutorial.views')
# Scan: tutorial.views (sama saja)

# Cara 3: Scan seluruh package
config.scan('tutorial')
# Scan: semua module di dalam tutorial/

# Cara 4: Scan module saat ini
config.scan('.')
# Scan: tutorial/ (current package)
```

#### Mengapa Menggunakan Relative Import?

**Keuntungan**:
1. **Portability**: Jika nama package berubah, tidak perlu update kode
2. **Clarity**: Jelas bahwa views.py ada di package yang sama
3. **Convention**: Idiomatis dalam Pyramid

**Contoh masalah tanpa relative import**:
```python
# Hardcode package name
config.scan('tutorial.views')

# Jika rename package menjadi 'myapp', harus update:
config.scan('myapp.views')

# Dengan relative import, tidak perlu update:
config.scan('.views')  # Tetap bekerja!
```

#### Python Import System

Dot notation dalam Python import:
- `.` = current package
- `..` = parent package
- `...` = grandparent package

Contoh:
```python
# Structure:
# myapp/
#   __init__.py
#   views/
#     __init__.py
#     home.py
#     admin.py

# Di myapp/views/admin.py:
from .home import home_view       # Same directory
from ..models import User          # Parent package
```

### 2. Mengapa `assertIn` Lebih Baik dari `assertEqual` untuk Testing Response?

**Pertanyaan**: Mengapa `assertIn` mungkin pilihan yang lebih baik daripada `assertEqual` saat testing text dalam response?

**Jawaban**:

#### Perbandingan

```python
# assertIn - Check substring
self.assertIn(b'Visit', response.body)

# assertEqual - Check exact match
self.assertEqual(response.body, b'<body>Visit <a href="/howdy">hello</a></body>')
```

#### Alasan `assertIn` Lebih Baik

##### 1. **Flexibility (Fleksibilitas)**

```python
# assertIn: Fokus pada content yang penting
self.assertIn(b'Welcome', response.body)

# Test tetap pass meskipun ada perubahan kecil:
# ✓ "<h1>Welcome to our site</h1>"
# ✓ "<div>Welcome, user!</div>"
# ✓ "Welcome! Click here to continue"

# assertEqual: Harus exact match
self.assertEqual(response.body, b'<h1>Welcome</h1>')

# Test akan fail jika ada perubahan sedikit saja:
# ✗ "<h1>Welcome to our site</h1>"  - Berbeda!
# ✗ "<h1>Welcome!</h1>"             - Berbeda!
```

##### 2. **Maintainability (Kemudahan Maintenance)**

```python
# Scenario: Designer mengubah HTML

# Original HTML:
return Response('<body>Visit <a href="/howdy">hello</a></body>')

# Updated HTML (ditambah CSS class):
return Response('<body class="home">Visit <a href="/howdy">hello</a></body>')

# assertIn: Tetap pass ✓
self.assertIn(b'Visit', response.body)

# assertEqual: Fail ✗
self.assertEqual(response.body, b'<body>Visit <a href="/howdy">hello</a></body>')
```

##### 3. **Brittleness (Ketahanan terhadap Perubahan)**

Test dengan `assertEqual` lebih "brittle" (mudah rusak):

```python
# Test dengan assertEqual
def test_home_brittle(self):
    response = home(request)
    self.assertEqual(response.body, 
        b'<body>Visit <a href="/howdy">hello</a></body>')

# Perubahan apapun akan break test:
# - Tambah whitespace
# - Ubah atribut HTML
# - Tambah elemen lain
# - Reformat code

# Test dengan assertIn
def test_home_flexible(self):
    response = home(request)
    self.assertIn(b'Visit', response.body)
    self.assertIn(b'/howdy', response.body)

# Lebih robust terhadap perubahan
```

##### 4. **Focus on Important Content**

```python
# assertIn: Test yang penting saja
def test_home(self):
    response = home(request)
    # Check key content exists
    self.assertIn(b'Visit', response.body)
    self.assertIn(b'hello', response.body)
    self.assertIn(b'/howdy', response.body)

# assertEqual: Test everything
def test_home(self):
    response = home(request)
    # Must match EXACTLY
    self.assertEqual(response.body, 
        b'<body>Visit <a href="/howdy">hello</a></body>')
```

#### Kapan Menggunakan Mana?

| Gunakan `assertIn` | Gunakan `assertEqual` |
|-------------------|----------------------|
| Testing HTML content | Testing API response values |
| Checking important keywords | Testing exact status codes |
| Flexible content checks | Testing configuration values |
| User-facing messages | Testing data integrity |

#### Best Practices

```python
class TutorialViewTests(unittest.TestCase):
    def test_home_content(self):
        from .views import home
        request = testing.DummyRequest()
        response = home(request)
        
        # Good: Check status code exactly
        self.assertEqual(response.status_code, 200)
        
        # Good: Check important content exists
        self.assertIn(b'Visit', response.body)
        self.assertIn(b'hello', response.body)
        
        # Good: Check link exists
        self.assertIn(b'href="/howdy"', response.body)
        
        # Avoid: Too brittle
        # self.assertEqual(response.body, b'<exact html>')
    
    def test_api_endpoint(self):
        response = api_view(request)
        
        # Good: Exact match for API responses
        self.assertEqual(response.json_body, {
            'status': 'success',
            'count': 42
        })
```

#### Multiple Assertions untuk Coverage Lebih Baik

```python
def test_home_comprehensive(self):
    response = home(request)
    
    # Check status
    self.assertEqual(response.status_code, 200)
    
    # Check multiple important pieces
    self.assertIn(b'Visit', response.body)
    self.assertIn(b'<a href="/howdy">', response.body)
    self.assertIn(b'hello</a>', response.body)
    
    # Check content type
    self.assertEqual(response.content_type, 'text/html')
```

**Kesimpulan**: `assertIn` lebih baik untuk testing HTML content karena lebih flexible dan maintainable. Test fokus pada content yang penting tanpa terlalu bergantung pada detail implementasi yang mungkin berubah.

---

## Kesimpulan

Mengorganisir views dalam module terpisah dengan declarative configuration membuat aplikasi Pyramid lebih maintainable dan scalable. Penggunaan `@view_config` decorator memberikan cara yang lebih idiomatis dan mudah dibaca untuk mendaftarkan views, sementara `config.scan()` secara otomatis menemukan dan mendaftarkan semua views yang di-decorate.
