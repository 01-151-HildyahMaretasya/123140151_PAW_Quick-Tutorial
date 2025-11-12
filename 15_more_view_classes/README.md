# Tutorial Pyramid: More With View Classes

## Pengenalan

Tutorial ini mengelompokkan views ke dalam sebuah class, sharing configuration, state, dan logic. Pendekatan ini membantu membangun aplikasi web yang lebih ambisius dengan struktur yang lebih terorganisir.

Sebagai bagian dari misinya untuk membantu membangun aplikasi web yang lebih ambisius, Pyramid menyediakan banyak fitur untuk views dan view classes.

Dokumentasi Pyramid membahas views sebagai Python "callable". Callable ini bisa berupa:
- Function biasa
- Object dengan method `__call__`
- Python class, dimana method pada class bisa di-decorate dengan `@view_config` untuk register class method sebagai view

### Evolusi Views

Awalnya, views adalah function sederhana yang berdiri sendiri. Namun seringkali views saling berhubungan: cara berbeda untuk melihat atau bekerja pada data yang sama, atau REST API yang handle multiple operations. Mengelompokkan ini sebagai view class lebih masuk akal:

**Keuntungan View Classes:**
- Mengelompokkan views yang related
- Memusatkan default configuration yang repetitive
- Share state dan helper functions
- Mengorganisir kode lebih baik

### View Predicates

Pyramid views memiliki **view predicates** yang menentukan view mana yang cocok dengan request, berdasarkan faktor seperti:
- Request method (GET, POST, dll)
- Form parameters
- Header information
- Dan banyak lagi

Predicates ini memberikan banyak fleksibilitas dalam routing.

## Tujuan Tutorial

* Mengelompokkan related views ke dalam view class
* Memusatkan konfigurasi dengan class-level `@view_defaults`
* Dispatch satu route/URL ke multiple views berdasarkan request data
* Share state dan logic antara views dan templates via view class

## Langkah-langkah Pengerjaan

### 1. Menyalin Project Sebelumnya
### 2. Menambahkan Replacement Patterns di Route
### 3. Membuat View Class dengan Multiple Views
### 4. Membuat Template Home
Buat template `more_view_classes/tutorial/home.pt`

### 5. Membuat Template Hello
Buat template `more_view_classes/tutorial/hello.pt`

### 6. Membuat Template Edit
Buat template `more_view_classes/tutorial/edit.pt`

### 7. Membuat Template Delete
Buat template `more_view_classes/tutorial/delete.pt`

### 8. Memperbarui Tests
Edit `more_view_classes/tutorial/tests.py`

### 9. Menjalankan Test

Jalankan test untuk memastikan semuanya berfungsi:

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

### 10. Menjalankan Aplikasi

Jalankan server development:

```bash
$VENV/bin/pserve development.ini --reload
```

### 11. Verifikasi di Browser

Buka http://localhost:6543/howdy/jane/doe di browser. Klik tombol **Save** dan **Delete**, lalu perhatikan output di console window.


## Analisis: Bagaimana Cara Kerjanya?

### Logical Grouping

Empat views dikelompokkan secara logis:

1. **Home view** tersedia di `http://localhost:6543/` dengan link yang bisa diklik ke hello view
2. **Hello view** di-return ketika mengakses `/howdy/jane/doe`. URL ini dipetakan ke route `hello` yang diset secara central menggunakan `@view_defaults`
3. **Edit view** di-return ketika form di-submit dengan method POST
4. **Delete view** di-return ketika klik tombol dengan `name="form.delete"`

### View Predicates untuk Dispatching

Tutorial ini menunjukkan cara menentukan view mana yang akan digunakan berdasarkan:

**1. HTTP Request Method**
```python
@view_config(request_method='POST', renderer='edit.pt')
```

**2. Parameter Information**
```python
@view_config(request_method='POST', request_param='form.delete', renderer='delete.pt')
```

### Centralized Configuration dengan @view_defaults

`@view_defaults` memusatkan konfigurasi di level class:

```python
@view_defaults(route_name='hello')
class TutorialViews:
    # ...
```

Semua views dalam class akan menggunakan `route_name='hello'` kecuali di-override:

```python
@view_config(route_name='home', renderer='home.pt')  # Override untuk home
def home(self):
    return {'page_title': 'Home View'}
```

### Sharing State dan Logic

View class memungkinkan sharing:

**1. State di `__init__`**
```python
def __init__(self, request):
    self.request = request
    self.view_name = 'TutorialViews'  # Shared state
```

**2. Computed Values dengan @property**
```python
@property
def full_name(self):
    first = self.request.matchdict['first']
    last = self.request.matchdict['last']
    return first + ' ' + last
```

State dan computed values ini tersedia di:
- View methods
- Templates (contoh: `${view.view_name}` dan `${view.full_name}`)

### Dynamic URL Generation

Beralih dari hardcoded URLs:

```html
<!-- Cara lama (hardcoded) -->
<a href="/howdy/jane/doe">Howdy</a>
```

Menjadi dynamic URL generation:

```html
<!-- Cara baru (flexible) -->
<a href="${request.route_url('hello', first='jane', last='doe')}">form</a>
```

**Keuntungan:**
- Fleksibel jika URL pattern berubah
- Tidak error-prone
- Mudah maintenance
- Type-safe dengan parameter

### Matchdict untuk URL Parameters

Pyramid menyediakan `request.matchdict` untuk mengakses URL parameters:

```python
first = self.request.matchdict['first']  # Dari /howdy/{first}/{last}
last = self.request.matchdict['last']
```

## Ekstra Kredit (Pertanyaan Lanjutan)

### 1. Mengapa Template Bisa `${view.full_name}` Tanpa `()`?

**Pertanyaan**: Mengapa template bisa melakukan `${view.full_name}` dan tidak harus `${view.full_name()}`?

**Jawaban**: Karena `full_name` adalah Python **@property**, bukan method biasa. Property adalah cara untuk membuat method terlihat seperti attribute. Ketika mengakses property, Python otomatis memanggil method-nya tanpa perlu tanda kurung.

```python
@property
def full_name(self):
    # Ini method, tapi dipanggil seperti attribute
    return first + ' ' + last

# Di template
${view.full_name}  # Bukan ${view.full_name()}
```

Property memberikan computed value dengan interface yang bersih.

### 2. Mengapa Edit View Tidak Catch POST dari Delete?

**Pertanyaan**: Edit dan delete views keduanya menerima POST requests. Mengapa edit view configuration tidak catch POST yang digunakan oleh delete?

**Jawaban**: Karena **view predicates** yang lebih spesifik. Delete view memiliki predicate tambahan:

```python
# Edit view - hanya check method
@view_config(request_method='POST', renderer='edit.pt')

# Delete view - check method DAN parameter
@view_config(request_method='POST', request_param='form.delete', renderer='delete.pt')
```

Delete view memerlukan:
1. Method = POST
2. Parameter `form.delete` harus ada di request

Edit view hanya memerlukan method POST. Ketika tombol delete diklik, parameter `form.delete` ada, jadi delete view yang dipilih karena lebih spesifik. Ketika tombol save diklik, tidak ada parameter `form.delete`, jadi edit view yang cocok.

**Pyramid matching order**: View dengan predicates lebih spesifik akan dipilih terlebih dahulu.

### 3. Caching Computed Properties

**Pertanyaan**: Menggunakan Python `@property` pada `full_name`. Jika reference ini berkali-kali di template atau view code, akan re-compute setiap kali. Apakah Pyramid menyediakan sesuatu yang akan cache initial computation pada property?

**Jawaban**: Ya! Pyramid menyediakan **@reify** decorator dari `pyramid.decorator`:

```python
from pyramid.decorator import reify

class TutorialViews:
    @reify
    def full_name(self):
        # Hanya compute sekali, lalu di-cache
        first = self.request.matchdict['first']
        last = self.request.matchdict['last']
        return first + ' ' + last
```

**Perbedaan @property vs @reify:**
- `@property`: compute setiap kali diakses
- `@reify`: compute sekali saat pertama diakses, lalu hasil di-cache untuk akses berikutnya

Gunakan `@reify` untuk:
- Expensive computations
- Database queries
- Values yang tidak berubah selama request lifecycle

### 4. Multiple Routes untuk Satu View

**Pertanyaan**: Apakah bisa mengasosiasikan lebih dari satu route dengan view yang sama?

**Jawaban**: Ya, sangat bisa! Gunakan multiple `@view_config` decorators:

```python
@view_config(route_name='hello', renderer='hello.pt')
@view_config(route_name='howdy', renderer='hello.pt')
@view_config(route_name='greet', renderer='hello.pt')
def hello(self):
    return {'page_title': 'Hello View'}
```

Atau gunakan array di single decorator (jika didukung):

```python
# Alternative syntax (tergantung Pyramid version)
@view_config(route_name=['hello', 'howdy', 'greet'], renderer='hello.pt')
def hello(self):
    return {'page_title': 'Hello View'}
```

Ini berguna untuk:
- URL aliases
- Legacy URL support
- API versioning

### 5. Perbedaan `request.route_path` dan `request.route_url`

**Pertanyaan**: Ada juga `request.route_path` API. Bagaimana perbedaannya dengan `request.route_url`?

**Jawaban**:

**`request.route_url`**
- Generate URL absolut lengkap dengan scheme dan domain
- Contoh output: `http://localhost:6543/howdy/jane/doe`
- Cocok untuk:
  - External links
  - Email notifications
  - API responses
  - Redirect ke domain lain

**`request.route_path`**
- Generate path relatif saja (tanpa scheme dan domain)
- Contoh output: `/howdy/jane/doe`
- Cocok untuk:
  - Internal navigation
  - Form actions
  - AJAX calls
  - Lebih ringan dan efisien

**Contoh perbandingan:**

```python
# Di template atau view
request.route_url('hello', first='jane', last='doe')
# Output: http://localhost:6543/howdy/jane/doe

request.route_path('hello', first='jane', last='doe')
# Output: /howdy/jane/doe
```

Untuk navigasi internal dalam aplikasi yang sama, `route_path` lebih disarankan karena lebih portable dan efisien.

---

## Kesimpulan

Tutorial ini menunjukkan kekuatan view classes di Pyramid:
1. Grouping related views untuk organisasi lebih baik
2. Centralized configuration dengan `@view_defaults`
3. View predicates untuk sophisticated dispatching
4. Sharing state dan computed values
5. Dynamic URL generation untuk flexibility
