# Functional Testing dengan WebTest di Pyramid

## Pendahuluan

Unit testing sangat bagus untuk menguji komponen individual, tetapi dalam aplikasi web, templating dan keseluruhan sistem juga perlu diuji. Functional testing memungkinkan pengujian end-to-end terhadap aplikasi web secara lengkap.

WebTest adalah package Python untuk functional testing yang mensimulasikan HTTP request penuh terhadap aplikasi WSGI, lalu menguji informasi dalam response-nya. Yang menarik, WebTest tidak perlu setup/teardown HTTP server yang sebenarnya, sehingga test tetap berjalan cepat dan cocok untuk TDD.

## Perbedaan Unit Testing vs Functional Testing

| Unit Testing | Functional Testing |
|--------------|-------------------|
| Menguji komponen individual (function/method) | Menguji aplikasi secara keseluruhan |
| Menggunakan fake request (DummyRequest) | Mensimulasikan HTTP request yang sesungguhnya |
| Lebih cepat dan terisolasi | Lebih lambat tapi lebih komprehensif |
| Tidak menguji template/view layer | Menguji semua layer termasuk template |

## Setup Project

### 1. Instalasi WebTest
Tambahkan `webtest` sebagai development dependency di `setup.py`

### 2. Install Dependencies

```bash
$VENV/bin/pip install -e ".[dev]"
```

## Menulis Functional Test
Extend file `tutorial/tests.py` dengan menambahkan functional test

### Menjalankan Test

```bash
$VENV/bin/pytest tutorial/tests.py -q
```

Output :
<img width="691" height="254" alt="image" src="https://github.com/user-attachments/assets/1e9d7fad-9a86-474e-ac69-9e5b818d6cae" />


## ANALISIS 

### Struktur Functional Test Class

```python
class TutorialFunctionalTests(unittest.TestCase):
```

Class ini terpisah dari unit test class untuk menjaga organisasi kode yang baik. Functional tests biasanya dipisahkan dari unit tests.

### setUp Method

```python
def setUp(self):
    from tutorial import main
    app = main({})
    from webtest import TestApp

    self.testapp = TestApp(app)
```

**Breakdown**:

1. **`from tutorial import main`**: Import factory function yang membuat aplikasi Pyramid
2. **`app = main({})`**: Membuat instance aplikasi dengan config kosong `{}`
3. **`from webtest import TestApp`**: Import TestApp dari WebTest
4. **`self.testapp = TestApp(app)`**: Wrap aplikasi Pyramid dengan TestApp untuk simulasi HTTP request

TestApp adalah objek yang memungkinkan simulasi HTTP request tanpa menjalankan server HTTP yang sebenarnya.

### Test Method

```python
def test_hello_world(self):
    res = self.testapp.get('/', status=200)
    self.assertIn(b'<h1>Hello World!</h1>', res.body)
```

**Breakdown**:

1. **`self.testapp.get('/', status=200)`**: 
   - Simulasi HTTP GET request ke path '/'
   - `status=200` mengecek bahwa response code adalah 200 (OK)
   - Mengembalikan response object
   
2. **`self.assertIn(b'<h1>Hello World!</h1>', res.body)`**:
   - Mengecek apakah HTML `<h1>Hello World!</h1>` ada di dalam response body
   - `res.body` berisi content HTML yang dikembalikan
   - `b'...'` adalah byte string (dijelaskan di Extra Credit)

### End-to-End Testing

Functional test ini adalah end-to-end testing karena:

1. Membuat aplikasi yang sesungguhnya dengan `main({})`
2. Mensimulasikan HTTP request yang sesungguhnya
3. Menguji seluruh stack: routing → view → template → response
4. Memverifikasi output HTML yang dikembalikan

Berbeda dengan unit test yang hanya menguji view function secara terisolasi, functional test menguji keseluruhan alur aplikasi.

### Kecepatan Eksekusi

Meskipun functional test lebih komprehensif, WebTest membuat test tetap cepat karena:

- Tidak perlu start/stop HTTP server
- Memanggil aplikasi WSGI secara langsung
- Masih cukup cepat untuk TDD (0.25 seconds untuk 2 tests)

---

## EXTRA CREDIT

### 1. Mengapa Functional Test Menggunakan `b''`?

**Pertanyaan**: Mengapa functional test menggunakan `b''` (byte string)?

**Jawaban**:

```python
self.assertIn(b'<h1>Hello World!</h1>', res.body)
```

Penggunaan `b''` diperlukan karena:

#### Alasan Teknis

1. **`res.body` adalah bytes, bukan string**:
   - HTTP response body dalam Python 3 dikembalikan sebagai bytes
   - Bytes adalah data mentah (raw data), bukan text yang sudah di-encode
   
2. **Perbandingan type harus sama**:
   - Untuk membandingkan atau mencari dalam bytes, harus menggunakan bytes
   - `b'hello'` adalah bytes, `'hello'` adalah string
   - Membandingkan bytes dengan string akan error atau return False

#### Contoh Perbandingan

```python
# BENAR: bytes dibandingkan dengan bytes
self.assertIn(b'<h1>Hello World!</h1>', res.body)  # ✓ Works

# SALAH: string dibandingkan dengan bytes
self.assertIn('<h1>Hello World!</h1>', res.body)   # ✗ Error atau False
```

#### Alternatif: Menggunakan res.text

Jika ingin bekerja dengan string, gunakan `res.text` yang sudah di-decode:

```python
# Menggunakan res.text (string)
res = self.testapp.get('/', status=200)
self.assertIn('<h1>Hello World!</h1>', res.text)  # ✓ Works, no b''

# Menggunakan res.body (bytes)
self.assertIn(b'<h1>Hello World!</h1>', res.body)  # ✓ Works, with b''
```

#### Waktu penggunaan yang tepat

| Gunakan `res.body` (bytes) | Gunakan `res.text` (string) |
|---------------------------|----------------------------|
| Ketika bekerja dengan binary data | Ketika bekerja dengan text/HTML |
| Saat butuh data mentah | Saat butuh compare string biasa |
| Untuk consistency dengan HTTP spec | Untuk readability yang lebih baik |

## Keuntungan Functional Testing dengan WebTest

1. **End-to-End Coverage**: Menguji seluruh stack aplikasi, bukan hanya komponen individual
2. **Template Testing**: Memastikan template render dengan benar dan menghasilkan HTML yang diharapkan
3. **Fast Execution**: Tidak perlu HTTP server yang sesungguhnya, sehingga test tetap cepat
4. **Terintegrasi dengan pytest**: Hasil test muncul dalam satu output bersama unit test
5. **Realistic Testing**: Mensimulasikan user interaction yang sesungguhnya
6. **Easy to Write**: Syntax sederhana dan intuitif untuk simulasi HTTP request


## Kesimpulan

Functional testing dengan WebTest melengkapi unit testing dengan memberikan coverage end-to-end terhadap aplikasi web. Kombinasi unit test (cepat dan terisolasi) dengan functional test (komprehensif dan realistic) memberikan confidence yang tinggi terhadap kualitas aplikasi.

WebTest memungkinkan penulisan test yang:
- Mensimulasikan user interaction yang sesungguhnya
- Menguji template dan HTML output
- Tetap cukup cepat untuk TDD
- Mudah dibaca dan di-maintain
