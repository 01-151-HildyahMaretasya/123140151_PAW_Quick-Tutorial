# Unit Testing dengan pytest di Pyramid

## Pendahuluan

Testing adalah bagian penting dalam pengembangan software. Seperti kata pepatah: "Kode yang tidak ditest adalah kode yang rusak." Tutorial ini menjelaskan cara menulis unit test untuk aplikasi Pyramid menggunakan pytest.

Unit testing adalah proses pengujian komponen-komponen kecil dari kode (biasanya function atau method) secara terpisah untuk memastikan mereka bekerja dengan benar. Setiap test harus bersifat independen dan tidak bergantung pada test lain.

## Setup Project

### 1. Instalasi pytest
### 2. Install Dependencies

```bash
$VENV/bin/pip install -e ".[dev]"
```

Perintah ini menginstall package dalam mode development beserta semua dependency yang ada di extras `[dev]`.

## Menulis Unit Test

Buat file `tutorial/tests.py`:

```python
import unittest

from pyramid import testing


class TutorialViewTests(unittest.TestCase):
    def setUp(self):
        self.config = testing.setUp()

    def tearDown(self):
        testing.tearDown()

    def test_hello_world(self):
        from tutorial import hello_world

        request = testing.DummyRequest()
        response = hello_world(request)
        self.assertEqual(response.status_code, 200)
```

### Menjalankan Test

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

### Output
<img width="688" height="343" alt="image" src="https://github.com/user-attachments/assets/7f18ad7b-8ced-4eb7-a8b1-76d64c35319b" />
<img width="685" height="359" alt="image" src="https://github.com/user-attachments/assets/d54253bd-191e-4a43-8bcb-a6cd1515965d" />
<img width="685" height="193" alt="image" src="https://github.com/user-attachments/assets/47fd4a9d-e3b0-4e33-91fe-dba595a519cc" />

## ANALISIS

Bagian ini menjelaskan bagaimana kode test bekerja dan konsep-konsep penting di baliknya.

### Struktur Test Class

```python
class TutorialViewTests(unittest.TestCase):
```

Class ini mewarisi dari `unittest.TestCase`, yang merupakan base class untuk semua unit test di Python.

### setUp dan tearDown

```python
def setUp(self):
    self.config = testing.setUp()

def tearDown(self):
    testing.tearDown()
```

- **setUp()**: Dijalankan sebelum setiap test method. Digunakan untuk menyiapkan lingkungan testing.
- **tearDown()**: Dijalankan setelah setiap test method. Digunakan untuk membersihkan lingkungan testing.

**Catatan**: Dalam contoh ini, `setUp()` dan `tearDown()` sebenarnya tidak diperlukan karena test tidak menggunakan object `config`. Method ini hanya diperlukan ketika test perlu menambahkan konfigurasi sebelum memanggil view.

### Test Method

```python
def test_hello_world(self):
    from tutorial import hello_world

    request = testing.DummyRequest()
    response = hello_world(request)
    self.assertEqual(response.status_code, 200)
```

**Mengapa import dilakukan di dalam method?**

Import dilakukan di dalam method test (bukan di atas file) untuk menjaga isolasi. Ini adalah prinsip penting dalam unit testing:

1. **Isolasi**: Setiap test harus independen dan tidak terpengaruh oleh test lain
2. **Menghindari side effects**: Import di level module bisa menyebabkan efek samping yang mempengaruhi test lain
3. **Unit testing principle**: Setiap test harus terisolasi dengan baik agar benar-benar menguji satu "unit" saja

**Proses testing**:

1. Import view function yang akan ditest
2. Buat fake request menggunakan `testing.DummyRequest()`
3. Panggil view dengan fake request tersebut
4. Assert bahwa response memiliki status code yang diharapkan (200 = OK)

### Assertion

```python
self.assertEqual(response.status_code, 200)
```

Method `assertEqual()` memeriksa apakah dua nilai sama. Jika tidak, test akan gagal dan pytest akan menampilkan error report.

---

## EXTRA CREDIT

Bagian ini berisi latihan tambahan dan eksplorasi lebih dalam tentang testing.

### 1. Test Response Code 404

**Latihan**: Ubah test untuk assert bahwa response status code seharusnya 404 (not found).

```python
def test_not_found(self):
    from tutorial import hello_world

    request = testing.DummyRequest()
    response = hello_world(request)
    self.assertEqual(response.status_code, 404)  # Akan gagal!
```

Ketika menjalankan pytest, akan muncul error report yang menunjukkan:
- Expected: 404
- Actual: 200
- Error report ini membantu memahami apa yang salah dalam test

### 2. Test dengan Error di View

**Latihan**: Kembalikan test seperti semula, lalu buat error di view (misalnya reference ke variable yang tidak ada).

Contoh error di view:
```python
def hello_world(request):
    x = undefined_variable  # Error!
    return Response('<h1>Hello World!</h1>')
```

Menjalankan test akan menampilkan error yang lebih mudah di-debug daripada harus reload browser berkali-kali. Test memberikan feedback cepat tentang apa yang rusak.

### 3. Mengubah Response Code di View

**Latihan**: Pelajari Pyramid Response objects dan cara mengubah response code.

Contoh mengubah response code:
```python
from pyramid.response import Response

def hello_world(request):
    response = Response('<h1>Hello World!</h1>')
    response.status_code = 404  # Atau response.status = '404 Not Found'
    return response
```

Test akan memvalidasi "kontrak" yang dijanjikan kode. Jika kode mengklaim mengembalikan 404, test memastikan hal itu benar-benar terjadi.

### 4. Test HTML Response Body

**Pertanyaan**: Bagaimana menambahkan assertion untuk test nilai HTML dari response body?

**Jawaban**: Gunakan assertion untuk memeriksa content body:

```python
def test_hello_world_content(self):
    from tutorial import hello_world

    request = testing.DummyRequest()
    response = hello_world(request)
    
    # Test HTML content (bytes)
    self.assertIn(b'<h1>Hello World!</h1>', response.body)
    
    # Atau test text content (string)
    self.assertIn('Hello World!', response.text)
    
    # Test substring
    self.assertTrue('Hello' in response.text)
```

**Catatan**: 
- `response.body` mengembalikan bytes, jadi perlu prefix `b` pada string
- `response.text` mengembalikan string yang sudah di-decode

### 5. Import di Dalam vs di Luar Method

**Pertanyaan**: Mengapa import `hello_world` view function dilakukan di dalam method `test_hello_world` dan bukan di top module?

**Jawaban**: 

Ini adalah best practice dalam unit testing untuk menjaga isolasi:

1. **Mencegah Side Effects**: Import di level module bisa menyebabkan efek samping (side effects) yang mempengaruhi test lain. Misalnya, jika module yang diimport melakukan inisialisasi global atau mengubah state.

2. **True Unit Testing**: Setiap test harus benar-benar terisolasi dan menguji satu "unit" saja. Import di dalam method memastikan setiap test dimulai dari kondisi yang bersih.

3. **Test Independence**: Test tidak boleh bergantung pada urutan eksekusi atau state dari test lain. Import di dalam method membantu mencapai independensi ini.

Contoh masalah jika import di top level:
```python
# BAD: Import di top level
from tutorial import hello_world

class TutorialViewTests(unittest.TestCase):
    def test_one(self):
        # Jika hello_world module mengubah global state...
        pass
    
    def test_two(self):
        # ...test ini mungkin terpengaruh oleh test_one
        pass
```

```python
# GOOD: Import di dalam method
class TutorialViewTests(unittest.TestCase):
    def test_one(self):
        from tutorial import hello_world
        # Test terisolasi
        pass
    
    def test_two(self):
        from tutorial import hello_world
        # Test ini independen dari test_one
        pass
```
---

## Keuntungan Unit Testing

1. **Feedback cepat**: Tidak perlu reload browser berulang kali untuk mengecek apakah kode bekerja
2. **Deteksi bug dini**: Menemukan error sebelum kode masuk ke production
3. **Dokumentasi**: Test berfungsi sebagai dokumentasi hidup tentang cara kerja kode
4. **Refactoring aman**: Bisa mengubah kode dengan percaya diri karena test akan menangkap error
5. **Kontrak kode**: Test memastikan kode memenuhi "kontrak" atau spesifikasi yang dijanjikan

## Kesimpulan

Unit testing dengan pytest membuat development lebih efisien dan kode lebih reliable. Meskipun tutorial ini tidak menggunakan pendekatan TDD (Test-Driven Development) secara ketat, menulis test tetap sangat penting untuk memastikan kualitas kode dan memudahkan maintenance di masa depan.
