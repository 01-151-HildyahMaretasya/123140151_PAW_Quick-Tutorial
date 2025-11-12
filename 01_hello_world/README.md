# Single-File Web Applications 

## Deskripsi

Tutorial ini menunjukkan cara membuat aplikasi web Pyramid paling sederhana menggunakan single-file module. 

**Microframeworks** adalah framework dengan overhead mental yang rendah, melakukan sedikit hal sehingga hanya perlu fokus pada aplikasi sendiri.

Pyramid istimewa karena dapat bertindak sebagai **single-file module microframework** sekaligus memiliki kemampuan untuk scale ke aplikasi besar.

### Konsep Penting

- **WSGI (Web Server Gateway Interface)**: Standard Python yang mendefinisikan bagaimana aplikasi web Python terhubung ke server, menerima request, dan mengembalikan response
- **MVC (Model-View-Controller)**: Pattern aplikasi di mana data dalam model memiliki view yang memediasi interaksi dengan sistem luar

## Objektif

- Menjalankan aplikasi web Pyramid sesederhana mungkin
- Memahami dasar WSGI apps, requests, views, dan responses
- Membuat foundation yang jelas untuk menambah kompleksitas aplikasi

## Langkah-langkah

## 1. Setup Directory
## 2. Buat File `app.py`
## 3. Jalankan Aplikasi

```bash
$VENV/bin/python app.py
```

## 4. Buka Browser

Akses: `http://localhost:6543/`

## Analisis Kode

### Line 11: `if __name__ == '__main__':`
Ini adalah cara Python mengatakan "Mulai dari sini ketika dijalankan dari command line", bukan ketika module ini diimpor oleh file lain.

### Lines 12-14: Configurator
```python
with Configurator() as config:
    config.add_route('hello', '/')
    config.add_view(hello_world, route_name='hello')
```
Menggunakan Pyramid's configurator dalam context manager untuk menghubungkan view code ke URL route tertentu. Configurator adalah central role dalam Pyramid development.

### Lines 6-8: View Function
```python
def hello_world(request):
    print('Incoming request')
    return Response('<body><h1>Hello World!</h1></body>')
```
Implementasi view code yang menghasilkan response HTTP. View menerima `request` object dan mengembalikan `Response` object.

### Lines 15-17: WSGI Server
```python
app = config.make_wsgi_app()
serve(app, host='0.0.0.0', port=6543)
```
Mempublikasikan WSGI app menggunakan HTTP server (Waitress). `make_wsgi_app()` membuat aplikasi yang kompatibel dengan standard WSGI.

## Konsep Kunci

**Application Configuration** adalah ide sentral dalam Pyramid - membangun aplikasi dari komponen yang loosely-coupled. Konsep ini akan sering digunakan dalam tutorial selanjutnya.

## Output :
<img width="766" height="290" alt="image" src="https://github.com/user-attachments/assets/00f12357-8eb1-42e6-9605-770c57fc9a94" />

## Extra Credit

### 1. Mengapa menggunakan `print('Incoming request')` bukan `print 'Incoming request'`?

Karena code ini ditulis untuk **Python 3**. Di Python 3, `print` adalah fungsi sehingga memerlukan tanda kurung `()`. Di Python 2, `print` adalah statement dan tidak memerlukan kurung.

```python
# Python 3 (correct)
print('Incoming request')

# Python 2 (legacy)
print 'Incoming request'
```

### 2. Apa yang terjadi jika mengembalikan string HTML atau sequence of integers?

**String HTML:**
```python
def hello_world(request):
    return '<h1>Hello</h1>'  # Error!
```
Akan menghasilkan **error** karena Pyramid view harus mengembalikan object Response, bukan string biasa.

**Sequence of Integers:**
```python
def hello_world(request):
    return [1, 2, 3]  # Error!
```
Juga akan menghasilkan **error** karena return value harus berupa Response object atau object yang compatible dengan WSGI.

**Solusi yang benar:**
```python
return Response('<h1>Hello</h1>')
```

### 3. Apa yang terjadi dengan kode invalid seperti `print xyz`?

Ketika menambahkan kode invalid:
```python
def hello_world(request):
    print xyz  # Invalid!
    return Response('<body><h1>Hello World!</h1></body>')
```

Dan merestart server, lalu reload browser:
- **Exception akan muncul di console** tempat menjalankan `python app.py`
- Browser akan menampilkan error page atau timeout
- Console akan menunjukkan traceback lengkap dengan detail error (`NameError: name 'xyz' is not defined`)

Ini sangat berguna untuk debugging karena dapat melihat langsung error yang terjadi.

### 4. GI dalam WSGI adalah "Gateway Interface" - dimodelkan dari standard web apa?

**CGI (Common Gateway Interface)**

WSGI dimodelkan dari CGI, yang merupakan standard lama untuk menjalankan program eksternal dari web server. WSGI adalah versi Python-specific dan lebih modern dari konsep gateway interface ini.

**Perbedaannya:**
- **CGI**: Meluncurkan proses baru untuk setiap request (lambat)
- **WSGI**: Aplikasi tetap berjalan dan hanya memanggil fungsi Python (cepat dan efisien)

## Dependencies

Pastikan telah menginstall:
```bash
pip install pyramid waitress
```

## Port Default

Aplikasi berjalan di 
- `http://localhost:6543`
- IP address kompute di network (jika firewall mengizinkan)


