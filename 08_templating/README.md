# Tutorial 08: Membuat HTML dengan Templating di Pyramid

## Deskripsi

Tutorial ini menjelaskan cara menggunakan sistem templating untuk menghasilkan HTML di Pyramid Framework. Dengan templating, kita tidak perlu lagi menulis HTML langsung di dalam kode Python, melainkan memisahkan logika program dan tampilan.

Pada tutorial sebelumnya, HTML dibuat langsung di dalam kode Python. Cara ini kurang efisien dan sulit dipelihara. Framework web modern menggunakan sistem templating yang memisahkan:
- **Logika program** (Python code)
- **Tampilan** (HTML template)

Pyramid mendukung berbagai template engine seperti Jinja2, Mako, dan Chameleon. Tutorial ini menggunakan **pyramid_chameleon**.

## Tujuan Pembelajaran

1. Mengaktifkan add-on `pyramid_chameleon` di proyek Pyramid
2. Membuat file template HTML terpisah
3. Menghubungkan template sebagai "renderer" untuk view
4. Mengubah view agar hanya mengembalikan data (bukan HTML)

## Langkah-langkah Instalasi

### 1. Membuat Proyek Baru
### 2. Menambahkan Dependency
### 3. Install Dependencies

```bash
$VENV/bin/pip install -e .
```

### 4. Konfigurasi Template Engine
Edit `tutorial/__init__.py` untuk mengaktifkan pyramid_chameleon

### 5. Membuat View
File `tutorial/views.py` sekarang lebih sederhana

- Tidak ada lagi kode HTML di view
- View hanya mengembalikan dictionary berisi data
- Parameter `renderer='home.pt'` menghubungkan view ke template

### 6. Membuat Template
Buat file `tutorial/home.pt`

**Sintaks Template:**
- `${name}` akan diganti dengan nilai dari dictionary yang dikembalikan view
- Template menggunakan sintaks Chameleon

### 7. Konfigurasi Development
Edit `development.ini` untuk auto-reload template

## Testing

### Unit Test

File `tutorial/tests.py` fokus pada pengujian data:

```python
def test_home(self):
    from .views import home
    request = testing.DummyRequest()
    response = home(request)
    # View sekarang mengembalikan data
    self.assertEqual('Home View', response['name'])
```

### Menjalankan Test

Buka browser dan akses:
- http://localhost:6543/
- http://localhost:6543/howdy



## Analisis

### Keuntungan Menggunakan Templating

1. **Pemisahan Concern (Separation of Concerns)**
   - Kode Python hanya menangani logika bisnis
   - Template hanya menangani tampilan
   - Lebih mudah dipelihara dan dikembangkan

2. **Reusability**
   - Satu template bisa digunakan oleh banyak view
   - Contoh: `home.pt` digunakan untuk route `home` dan `hello`

3. **Testing Lebih Mudah**
   - Unit test fokus pada kontrak data
   - Tidak perlu mengecek HTML string yang rumit
   - Test menjadi lebih sederhana dan robust

4. **Fleksibilitas**
   - Pyramid tidak memaksakan satu template engine
   - Developer bebas memilih: Chameleon, Jinja2, atau Mako
   - Mudah mengganti template engine tanpa ubah view code

5. **Developer Experience**
   - Auto-reload template saat development
   - Tidak perlu restart server setiap ubah tampilan
   - Proses development lebih cepat

### Konsep Penting

**Renderer:**
- Parameter yang menghubungkan view dengan template
- Pyramid secara otomatis meneruskan data ke template
- Data yang dikembalikan view (dictionary) menjadi variabel di template

**Data Contract:**
- View hanya perlu mengembalikan dictionary
- Template menerima dictionary tersebut
- Kontrak yang jelas antara view dan template

## Kesimpulan

Templating membuat kode lebih bersih, terstruktur, dan mudah dipelihara. View fokus pada logika dan data, sementara template fokus pada presentasi. Ini adalah praktik standar dalam pengembangan web modern.