# Tutorial 10: Menangani Web Requests dan Responses

## Deskripsi

Tutorial ini menjelaskan cara menangani HTTP requests dan responses di Pyramid Framework. Pyramid menggunakan library WebOb yang mature dan robust untuk memproses request dari client dan mengirim response kembali.

Aplikasi web bekerja dengan menerima request dari client (browser, mobile app) dan mengirimkan response kembali. Proses ini adalah inti dari web development, sehingga membutuhkan library yang handal.

Pyramid menggunakan **WebOb**, library Python yang sudah teruji dan banyak digunakan untuk:
- Membaca data dari request (URL, parameters, headers, cookies)
- Mengatur response (status code, headers, body, cookies)
- Menangani redirect dan HTTP exceptions

## Tujuan 

1. Memahami cara Pyramid menangani request dan response
2. Mengambil data dari request (URL parameters)
3. Mengubah informasi di response headers
4. Melakukan HTTP redirect

## Langkah-langkah Implementasi

### 1. Setup Proyek

```bash
cd ..
cp -r view_classes request_response
cd request_response
$VENV/bin/pip install -e .
```

### 2. Konfigurasi Routes
Sederhanakan file `request_response/tutorial/__init__.py`
**Penjelasan:**
- Route `home` (`/`) - Halaman utama yang akan redirect
- Route `plain` (`/plain`) - Halaman tujuan yang menampilkan plain text

### 3. Membuat Views
Edit file `request_response/tutorial/views.py`

#### Penjelasan Detail

**View `home()` - HTTP Redirect**

```python
@view_config(route_name='home')
def home(self):
    return HTTPFound(location='/plain')
```

- `HTTPFound` adalah HTTP 302 redirect
- Ketika user mengakses `/`, otomatis diarahkan ke `/plain`
- `location='/plain'` menentukan URL tujuan redirect

**View `plain()` - Membaca Request & Membuat Response**

```python
@view_config(route_name='plain')
def plain(self):
    # 1. Membaca parameter dari URL
    name = self.request.params.get('name', 'No Name Provided')
    
    # 2. Membuat body response
    body = 'URL %s with name: %s' % (self.request.url, name)
    
    # 3. Membuat Response object
    return Response(
        content_type='text/plain',
        body=body
    )
```

**Membaca Request:**
- `self.request.params` - Dictionary berisi query parameters dan form data
- `self.request.params.get('name', 'default')` - Ambil parameter `name`, jika tidak ada gunakan default
- `self.request.url` - URL lengkap yang sedang diakses

**Membuat Response:**
- `Response()` - Object response manual
- `content_type='text/plain'` - Set header Content-Type
- `body=body` - Isi response body

### 4. Update Tests
Edit file `request_response/tutorial/tests.py`

#### Penjelasan Tests

**Test Redirect:**
```python
def test_home(self):
    response = inst.home()
    self.assertEqual(response.status, '302 Found')
```
- Memastikan view `home()` mengembalikan HTTP 302

**Test Request Parameters:**
```python
# Simulasi request tanpa parameter
request = testing.DummyRequest()
response = inst.plain()
self.assertIn(b'No Name Provided', response.body)

# Simulasi request dengan parameter
request = testing.DummyRequest()
request.GET['name'] = 'Jane Doe'
response = inst.plain()
self.assertIn(b'Jane Doe', response.body)
```
- `request.GET['name']` mensimulasikan query parameter `?name=...`
- Memastikan default value bekerja
- Memastikan parameter yang dikirim diproses dengan benar

**Functional Tests:**
```python
res = self.testapp.get('/plain?name=Jane%20Doe', status=200)
```
- Test end-to-end dengan HTTP request sebenarnya
- `%20` adalah URL encoding untuk spasi
- `status=200` memastikan response sukses

### 5. Menjalankan Tests

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

### 6. Menjalankan Aplikasi

```bash
$VENV/bin/pserve development.ini --reload
```

**Coba akses:**
- http://localhost:6543/ → Otomatis redirect ke `/plain`
- http://localhost:6543/plain → Tampil: "URL ... with name: No Name Provided"
- http://localhost:6543/plain?name=alice → Tampil: "URL ... with name: alice"

**Output:**
<img width="1919" height="414" alt="Cuplikan layar 2025-11-12 214931" src="https://github.com/user-attachments/assets/0afed634-9e29-43f9-8d38-0b14886342a8" />


## Analisis

### Flow Aplikasi

1. **User mengakses `/`**
   - View `home()` dipanggil
   - Return `HTTPFound(location='/plain')`
   - Browser menerima HTTP 302 dan redirect ke `/plain`

2. **Browser mengakses `/plain`**
   - View `plain()` dipanggil
   - Membaca parameter `name` dari URL
   - Membuat response dengan Content-Type: text/plain
   - Menampilkan URL dan nilai parameter

3. **User mengakses `/plain?name=alice`**
   - Query parameter `name=alice` dibaca dari `request.params`
   - Response body menyertakan nilai "alice"

### Request Object

Request object menyediakan akses ke berbagai informasi:

```python
# URL Information
self.request.url              # Full URL
self.request.path             # Path saja (/plain)
self.request.host             # Hostname
self.request.scheme           # http atau https

# Parameters
self.request.params           # Query + Form data
self.request.GET              # Query parameters saja
self.request.POST             # Form data saja

# Headers
self.request.headers          # Request headers
self.request.cookies          # Cookies

# Body
self.request.body             # Raw body
self.request.json_body        # Parse JSON body

# Session & Auth
self.request.session          # Session data
self.request.authenticated_userid  # User ID
```

### Response Object

Response object mengatur data yang dikirim ke client:

```python
Response(
    body='Hello',                    # Response body
    status='200 OK',                 # HTTP status
    content_type='text/html',        # Content-Type header
    charset='UTF-8',                 # Character encoding
    headerlist=[                     # Custom headers
        ('X-Custom', 'Value')
    ]
)
```

### HTTP Redirects

Ada beberapa cara melakukan redirect di Pyramid:

**1. Return HTTPFound (Recommended)**
```python
return HTTPFound(location='/plain')
```

**2. Raise HTTPFound**
```python
raise HTTPFound(location='/plain')
```

**3. Response dengan status 302**
```python
from pyramid.response import Response
return Response(status=302, location='/plain')
```

### Jenis HTTP Redirects

- `HTTPFound` (302) - Temporary redirect, method bisa berubah
- `HTTPMovedPermanently` (301) - Permanent redirect
- `HTTPSeeOther` (303) - Redirect setelah POST, force GET
- `HTTPTemporaryRedirect` (307) - Temporary, method tetap sama

## Extra Credit: Return vs Raise HTTPFound

### Return HTTPFound

```python
def home(self):
    return HTTPFound(location='/plain')
```

**Karakteristik:**
- Normal flow control
- View selesai eksekusi normal
- Lebih eksplisit dan readable
- **Recommended untuk kasus umum**

### Raise HTTPFound

```python
def home(self):
    raise HTTPFound(location='/plain')
```

**Karakteristik:**
- Exception flow control
- Eksekusi view langsung berhenti
- Berguna untuk redirect di tengah proses
- Bisa digunakan di dalam helper functions

### Kapan Menggunakan Raise?

```python
def process_data(self):
    # Cek kondisi di tengah proses
    if not self.request.authenticated_userid:
        raise HTTPFound(location='/login')
    
    # Lanjut proses...
    data = expensive_operation()
    
    if data is None:
        raise HTTPFound(location='/error')
    
    return {'data': data}
```

**Keuntungan raise:**
- Tidak perlu `return` di multiple tempat
- Langsung stop eksekusi
- Mirip pattern exception handling

### Perbedaan Teknis

**Return:**
- Object dikembalikan sebagai response normal
- View method harus mengembalikan sesuatu
- Flow control menggunakan `return`

**Raise:**
- Exception ditangkap oleh Pyramid
- Dikonversi menjadi response
- Flow control menggunakan exception mechanism
- Bisa dipanggil dari helper function tanpa perlu propagate return value

**Kesimpulan:** Keduanya valid dan menghasilkan hasil yang sama. `return` lebih eksplisit untuk kasus sederhana, `raise` lebih fleksibel untuk logika kompleks.

## Konsep Penting

### 1. WebOb Integration

Pyramid menggunakan WebOb untuk request/response:
- Mature dan well-tested
- Standar di ekosistem Python web
- API yang konsisten dan powerful

### 2. Request Lifetime

Request object valid selama handling satu request:
```python
def __init__(self, request):
    self.request = request  # Store untuk digunakan di methods
```

### 3. Response Types

Pyramid mendukung berbagai response:
- `Response` - Manual response
- Dictionary - Otomatis dirender dengan template
- `HTTPException` - Redirect atau error response
- String - Converted to Response

## Kesimpulan

Tutorial ini menunjukkan fondasi penting dalam web development: handling requests dan responses. Pyramid dengan WebOb menyediakan API yang powerful dan mudah digunakan untuk:
- Membaca data dari request (parameters, headers, body)
- Membuat custom responses (headers, body, status)
- Melakukan redirects dengan berbagai metode
- Testing request/response dengan tools yang komprehensif
