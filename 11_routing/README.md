# Tutorial 11: Dispatching URLs ke Views dengan Routing

## Deskripsi

Tutorial ini menjelaskan cara menggunakan routing di Pyramid untuk mencocokkan pola URL dengan view code. Pyramid routing memiliki fitur-fitur powerful untuk mengekstrak data dari URL dan menggunakannya di dalam views.

Aplikasi web modern membutuhkan desain URL yang sophisticated. URL tidak hanya sebagai alamat, tapi juga sebagai cara untuk mengirim data ke server. Contoh:
- `/users/123` - ID user adalah 123
- `/blog/2024/05/article-title` - Tahun, bulan, dan slug artikel
- `/products/laptop/macbook-pro` - Kategori dan nama produk

### Kenapa Routing Terpisah dari View?

Pyramid memisahkan pendaftaran route dan konfigurasi view menjadi dua langkah:

1. **Registrasi Route** - Mendefinisikan pola URL
2. **Konfigurasi View** - Menghubungkan route dengan view function/method

**Keuntungan:**
- **Explicit Ordering** - Urutan route jelas dan terkontrol
- **Flexibility** - Satu route bisa digunakan berbagai views
- **Testability** - Route dan view bisa ditest terpisah
- **Maintainability** - Mudah melihat semua routes di satu tempat

## Tujuan 

1. Mendefinisikan route dengan replacement pattern (URL parameters)
2. Mengekstrak data dari URL ke Python dictionary
3. Menggunakan data URL di dalam views
4. Testing routing dengan matchdict

## Langkah-langkah Implementasi

### 1. Setup Proyek
### 2. Konfigurasi Route dengan Replacement Pattern
Edit file `routing/tutorial/__init__.py`

#### Penjelasan Replacement Pattern

```python
config.add_route('home', '/howdy/{first}/{last}')
```

**Komponen URL:**
- `/howdy/` - Static part (tetap)
- `{first}` - Dynamic part pertama (variable)
- `{last}` - Dynamic part kedua (variable)

**Contoh Matching:**
| URL | first | last |
|-----|-------|------|
| `/howdy/amy/smith` | `amy` | `smith` |
| `/howdy/john/doe` | `john` | `doe` |
| `/howdy/jane/watson` | `jane` | `watson` |

### 3. Membuat View
Edit file `routing/tutorial/views.py`

#### Penjelasan Kode

**Mengakses URL Parameters:**
```python
first = self.request.matchdict['first']
last = self.request.matchdict['last']
```

- `request.matchdict` adalah dictionary yang berisi nilai dari replacement patterns
- Key dictionary sama dengan nama di curly braces `{first}` dan `{last}`
- Nilai diambil dari URL yang diakses user

**Return Data:**
```python
return {
    'name': 'Home View',
    'first': first,
    'last': last
}
```
Dictionary ini akan diteruskan ke template untuk di-render.

### 4. Membuat Template
Edit file `routing/tutorial/home.pt`

**Template Variables:**
- `${name}` - Dari dictionary key 'name'
- `${first}` - Dari dictionary key 'first' (dari URL)
- `${last}` - Dari dictionary key 'last' (dari URL)

### 5. Update Tests
Edit file `routing/tutorial/tests.py`

#### Penjelasan Testing

**Unit Test - Simulasi Matchdict:**
```python
request = testing.DummyRequest()
request.matchdict['first'] = 'First'
request.matchdict['last'] = 'Last'
```
- Membuat DummyRequest untuk testing
- Manually set matchdict untuk mensimulasi URL parameters
- Test view logic tanpa HTTP request sebenarnya

**Functional Test - Real URL:**
```python
res = self.testapp.get('/howdy/Jane/Doe', status=200)
self.assertIn(b'Jane', res.body)
self.assertIn(b'Doe', res.body)
```
- Request ke URL sebenarnya
- Pyramid router otomatis extract parameters
- Test end-to-end termasuk routing

### 6. Menjalankan Tests

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

### 7. Menjalankan Aplikasi

```bash
$VENV/bin/pserve development.ini --reload
```

**Coba akses:**
- http://localhost:6543/howdy/amy/smith
- http://localhost:6543/howdy/john/doe
- http://localhost:6543/howdy/alice/wonderland

**Output di browser:**
<img width="959" height="269" alt="image" src="https://github.com/user-attachments/assets/78978de8-62e0-43ba-b9a0-f5df67a307fe" />

## Analisis

### Cara Kerja Routing

1. **User mengakses URL:** `/howdy/amy/smith`

2. **Pyramid Router mencari route yang match:**
   ```python
   config.add_route('home', '/howdy/{first}/{last}')
   ```

3. **Extract parameters ke matchdict:**
   ```python
   request.matchdict = {
       'first': 'amy',
       'last': 'smith'
   }
   ```

4. **View mengakses matchdict:**
   ```python
   first = self.request.matchdict['first']  # 'amy'
   last = self.request.matchdict['last']    # 'smith'
   ```

5. **Data diteruskan ke template:**
   ```python
   return {'first': first, 'last': last}
   ```

### Request.matchdict

`request.matchdict` adalah dictionary yang berisi:
- Semua values dari replacement patterns di URL
- Tersedia di semua bagian aplikasi yang punya akses ke request
- Immutable setelah routing selesai

**Contoh:**
```python
# Route: /blog/{year}/{month}/{slug}
# URL: /blog/2024/05/my-article

request.matchdict = {
    'year': '2024',
    'month': '05',
    'slug': 'my-article'
}
```

### Replacement Pattern Types

#### 1. Simple Pattern
```python
config.add_route('user', '/users/{id}')
# Matches: /users/123, /users/abc
```

#### 2. Multiple Patterns
```python
config.add_route('blog', '/blog/{year}/{month}/{day}')
# Matches: /blog/2024/05/15
```

#### 3. Pattern dengan Regex
```python
config.add_route('user', '/users/{id:\d+}')
# Matches: /users/123 (hanya digit)
# Not match: /users/abc
```

#### 4. Optional Patterns
```python
config.add_route('page', '/page/{num:\d+}')
config.add_route('page_default', '/page')
# /page/5 → num = 5
# /page → route berbeda
```

#### 5. Catch-All Pattern
```python
config.add_route('files', '/files/*subpath')
# /files/docs/readme.txt → subpath = ('docs', 'readme.txt')
```

## Extra Credit: Akses /howdy Tanpa Parameters

### Pertanyaan
Apa yang terjadi jika mengakses http://localhost:6543/howdy tanpa `/first/last`?

### Jawaban

**Error yang terjadi:**
```
404 Not Found
```

**Kenapa 404?**
Route didefinisikan dengan **required parameters**:
```python
config.add_route('home', '/howdy/{first}/{last}')
```

URL `/howdy` tidak match pattern ini karena:
- Pattern butuh 3 segments: `/howdy/`, `{first}`, `{last}`
- URL hanya punya 1 segment: `/howdy`
- Pyramid tidak menemukan route yang cocok
- Return 404 Not Found

### Solusi: Membuat Route Optional

**Opsi 1: Route Terpisah**
```python
config.add_route('home_full', '/howdy/{first}/{last}')
config.add_route('home_partial', '/howdy')
```

**Opsi 2: Default Values di View**
```python
config.add_route('home', '/howdy/{first}/{last}')
config.add_route('home_default', '/howdy')

@view_config(route_name='home')
@view_config(route_name='home_default')
def home(self):
    first = self.request.matchdict.get('first', 'Guest')
    last = self.request.matchdict.get('last', '')
    return {'first': first, 'last': last}
```

**Opsi 3: Custom Predicates**
```python
config.add_route('home', '/howdy/*traverse')
```

### Expected vs Actual Result

**Expected (oleh pemula):**
- Mungkin mengharapkan default values atau empty strings
- Atau redirect ke form untuk input nama

**Actual (behavior Pyramid):**
- 404 Not Found karena pattern tidak match
- Pyramid explicit: jika route tidak match, return 404
- Tidak ada "guessing" atau fallback otomatis

**Design Philosophy:**
- Explicit is better than implicit
- Developer harus explicit tentang URL patterns
- Mencegah bugs dari route matching yang ambigu

## Konsep Penting

### 1. Route Ordering

Routes di-check berdasarkan urutan pendaftaran:
```python
config.add_route('specific', '/users/admin')
config.add_route('general', '/users/{id}')
# /users/admin → match 'specific'
# /users/123 → match 'general'
```

**PENTING:** Order matters! Jika dibalik:
```python
config.add_route('general', '/users/{id}')
config.add_route('specific', '/users/admin')
# /users/admin → match 'general' dengan id='admin'
# Route 'specific' tidak akan pernah tercapai!
```

### 2. Matchdict vs Params

**Matchdict** - Dari URL path:
```python
# Route: /users/{id}
# URL: /users/123
request.matchdict['id']  # '123'
```

**Params** - Dari query string:
```python
# URL: /users?id=123
request.params['id']  # '123'
```

Keduanya berbeda sumber dan use case!

### 3. Route Names

Route names digunakan untuk:
```python
# Generate URLs
request.route_url('home', first='john', last='doe')
# Returns: '/howdy/john/doe'

# Reverse routing di template
${request.route_url('home', first='jane', last='smith')}
```

Keuntungan: Jika URL pattern berubah, tidak perlu update semua link manual.

## Use Cases Routing

### 1. RESTful APIs
```python
config.add_route('users_list', '/api/users')
config.add_route('user_detail', '/api/users/{id}')
config.add_route('user_posts', '/api/users/{id}/posts')
```

### 2. Blog/CMS
```python
config.add_route('blog_home', '/blog')
config.add_route('blog_category', '/blog/category/{slug}')
config.add_route('blog_post', '/blog/{year}/{month}/{slug}')
```

### 3. E-commerce
```python
config.add_route('products', '/products')
config.add_route('category', '/products/category/{slug}')
config.add_route('product', '/products/{category}/{id}')
```

### 4. User Profiles
```python
config.add_route('profile', '/u/{username}')
config.add_route('profile_posts', '/u/{username}/posts')
config.add_route('profile_followers', '/u/{username}/followers')
```

## Kesimpulan

Routing adalah fondasi penting dalam web development. Pyramid routing menyediakan:
- **Flexibility** - Berbagai pattern types (simple, regex, catch-all)
- **Explicitness** - Urutan route jelas, no guessing
- **Power** - Extract data dari URL dengan mudah
- **Testability** - Mudah di-test dengan matchdict
