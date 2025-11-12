# Tutorial 09: Mengorganisir Views dengan View Classes

## Deskripsi

Tutorial ini menjelaskan cara mengubah view functions menjadi view classes di Pyramid Framework. Dengan view classes, beberapa view yang berhubungan dapat dikelompokkan dalam satu class, membuat kode lebih terorganisir dan mudah dipelihara.

Pada tutorial sebelumnya, semua view dibuat sebagai fungsi terpisah. Namun dalam aplikasi nyata, seringkali beberapa view saling berkaitan karena:
- Bekerja dengan data yang sama
- Merupakan bagian dari REST API yang menangani berbagai operasi
- Berbagi konfigurasi atau helper functions yang sama

### Keuntungan View Classes

1. **Mengelompokkan Views** - View yang berhubungan diorganisir dalam satu tempat
2. **Sentralisasi Konfigurasi** - Konfigurasi berulang bisa dipindah ke class level
3. **Berbagi State dan Helper** - Mudah berbagi data dan fungsi helper antar views

## Tujuan 

1. Mengelompokkan views yang berkaitan ke dalam view class
2. Memusatkan konfigurasi menggunakan `@view_defaults` di class level
3. Memahami pola instansiasi view class

## Langkah-langkah Implementasi

### 1. Setup Proyek
### 2. Membuat View Class

#### Penjelasan Kode

**`@view_defaults(renderer='home.pt')`**
- Decorator di level class
- Mengatur default renderer untuk semua method dalam class
- Tidak perlu menulis `renderer='home.pt'` di setiap `@view_config`

**`__init__(self, request)`**
- Constructor yang menerima request object
- Request disimpan sebagai instance variable (`self.request`)
- Pyramid otomatis memanggil constructor ini dan menyuntikkan request

**Method `home()` dan `hello()`**
- Sebelumnya: function dengan parameter `request`
- Sekarang: method class tanpa parameter (menggunakan `self.request`)
- Mengembalikan dictionary seperti sebelumnya

### 3. Update Unit Tests
Edit file `view_classes/tutorial/tests.py`

#### Perubahan pada Testing

**Sebelum (View Functions):**
```python
from .views import home
response = home(request)
```

**Sesudah (View Classes):**
```python
from .views import TutorialViews
inst = TutorialViews(request)
response = inst.home()
```

**Pola Testing View Class:**
1. Import view class
2. Buat instance dengan DummyRequest
3. Panggil method yang akan ditest
4. Assert hasilnya

### 4. Menjalankan Test

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

### 5. Menjalankan Aplikasi

```bash
$VENV/bin/pserve development.ini --reload
```

Buka browser dan akses:
- http://localhost:6543/
- http://localhost:6543/howdy

**Output yang diharapkan:**


## Analisis

### Perubahan Utama

Tutorial ini tidak menambahkan fitur baru, hanya mengubah struktur kode untuk mempermudah maintenance. Berikut perbandingannya:

#### Sebelum: View Functions

```python
@view_config(route_name='home', renderer='home.pt')
def home(request):
    return {'name': 'Home View'}

@view_config(route_name='hello', renderer='home.pt')
def hello(request):
    return {'name': 'Hello View'}
```

**Masalah:**
- Konfigurasi `renderer='home.pt'` berulang
- Views terpisah meski logikanya berkaitan
- Sulit berbagi state atau helper functions

#### Sesudah: View Class

```python
@view_defaults(renderer='home.pt')
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

**Keuntungan:**
- Konfigurasi renderer hanya sekali di `@view_defaults`
- Views dikelompokkan secara logis
- Mudah menambahkan shared state atau methods

### Konsep Penting

#### 1. View Defaults

`@view_defaults` mengatur konfigurasi default untuk semua method dalam class:
- `renderer` - Template default
- `permission` - Izin akses default
- `context` - Context default
- Dan decorator lainnya

#### 2. Dependency Injection

Pyramid otomatis menyuntikkan `request` ke constructor:
```python
def __init__(self, request):
    self.request = request
```

Ini memungkinkan semua method mengakses request tanpa parameter tambahan.

#### 3. Instance Variables

State dapat disimpan di instance variables dan diakses di semua methods:
```python
def __init__(self, request):
    self.request = request
    self.user = request.authenticated_userid
    self.db = request.db

def home(self):
    # Bisa akses self.user dan self.db
    pass
```

### Kapan Menggunakan View Classes?

**Gunakan View Class ketika:**
- Beberapa views bekerja dengan data yang sama
- Membuat REST API (GET, POST, PUT, DELETE untuk satu resource)
- Perlu berbagi helper functions atau computed properties
- Banyak konfigurasi yang berulang

**Tetap gunakan View Functions ketika:**
- View berdiri sendiri dan tidak berkaitan
- Logika sangat sederhana
- Tidak ada konfigurasi yang berulang

##  Catatan

1. **Naming Convention** - Beri nama class dengan suffix `Views` atau `Controller`
2. **Satu Tanggung Jawab** - Satu view class untuk satu resource atau fitur
3. **Constructor Ringan** - Jangan lakukan operasi berat di `__init__`
4. **Testing** - Selalu instantiate class dengan DummyRequest saat testing
5. **Documentation** - Dokumentasikan class untuk menjelaskan tanggung jawabnya

## Kesimpulan

View classes menyediakan cara yang lebih terorganisir untuk mengelola views yang berkaitan. Dengan `@view_defaults`, konfigurasi berulang bisa dipusatkan di satu tempat. Meski tutorial ini hanya melakukan konversi dasar, konsep ini sangat powerful untuk aplikasi yang lebih kompleks.
