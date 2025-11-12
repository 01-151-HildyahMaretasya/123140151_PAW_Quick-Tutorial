# Easier Development dengan Debug Toolbar

## Deskripsi

Tutorial ini menunjukkan cara menggunakan `pyramid_debugtoolbar` add-on untuk error handling dan introspection selama development. Toolbar ini menyediakan berbagai tools debugging yang berguna langsung di browser.

Produktivitas dalam development dan debugging sangat penting. Setelah membahas template reloading dan `--reload` untuk application reloading, sekarang akan diperkenalkan tools yang lebih powerful.

### Apa itu pyramid_debugtoolbar?

**pyramid_debugtoolbar** adalah Pyramid add-on populer yang menyediakan berbagai tools development di browser, seperti:
- Performance monitoring
- Request/Response inspection
- Exception debugging dengan interactive traceback
- SQLAlchemy query profiling
- Template rendering info

Mengintegrasikan add-on ini juga mendemonstrasikan beberapa poin penting tentang configuration.

## Objektif

- Install dan enable toolbar untuk membantu development
- Memahami konsep Pyramid add-ons
- Menunjukkan cara konfigurasi add-on ke dalam aplikasi

## Langkah-langkah

### 1. Copy Project Sebelumnya
### 2. Update `setup.py` dengan Development Dependencies
### 3. Install Project dengan Extra Dependencies
### 4. Update File `development.ini`

```bash
$VENV/bin/pserve development.ini --reload
```
### 5. Buka Browser

Akses: `http://localhost:6543/`

###Analisis
Perhatikan toolbar yang muncul di sisi kanan browser.

### Pyramid Add-on

`pyramid_debugtoolbar` adalah:
1. **Python package** - tersedia di PyPI seperti ribuan package Python lainnya
2. **Pyramid add-on** - memerlukan konfigurasi khusus untuk di-include ke aplikasi

### Cara Mengintegrasikan Add-on

Ada **dua cara** untuk include add-on configuration:

#### 1. Imperative Configuration (di Python code)

```python
# tutorial/__init__.py
def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_debugtoolbar')  # Imperative
    # ... rest of configuration
```

#### 2. Declarative Configuration (di .ini file)

```ini
# development.ini
[app:main]
pyramid.includes =
    pyramid_debugtoolbar
```

Tutorial ini menggunakan cara ke-2 (declarative) karena lebih fleksibel.

### Setuptools Extras

**Apa itu Extras?**

Extras adalah fitur opsional atau recommended yang bisa diinstall dengan "extras specifier".

**Format dalam setup.py:**
```python
extras_require={
    'dev': dev_requires,      # Development extras
    'test': test_requires,    # Testing extras
    'prod': prod_requires,    # Production extras
}
```

**Format install:**
```bash
pip install -e ".[extra_name]"
```

**Contoh:**
- `pip install -e ".[dev]"` - Install dengan dev extras
- `pip install -e ".[test]"` - Install dengan test extras
- `pip install -e ".[dev,test]"` - Install multiple extras

### Cara Kerja Debug Toolbar

1. **Inject HTML/CSS** - Toolbar menambahkan HTML/CSS kecil sebelum closing tag `</body>`
2. **Display button** - Muncul button di sisi kanan browser
3. **Introspective access** - Click button untuk akses debugging info di tab baru
4. **Error display** - Jika aplikasi error, traceback bagus akan muncul di screen

### Enable/Disable Toolbar

**Disable toolbar:**
```ini
# development.ini
[app:main]
pyramid.includes =
#    pyramid_debugtoolbar    # Comment out
```

### Kapan Perlu Disable Toolbar?

- Saat testing client-side behavior yang kompleks
- Jika mengalami client-side weirdness yang tidak bisa dijelaskan
- Production environment (jangan pernah aktif di production!)

### Output
<img width="959" height="319" alt="image" src="https://github.com/user-attachments/assets/47a240ec-c9e9-4652-bd55-00637110db58" />

## Extra Credit 

### 1. Mengapa pyramid_debugtoolbar ditambahkan di dev_requires, bukan di requires?

**Alasan utama: Separation of concerns**

#### Dependencies yang Tepat

**requires (Production dependencies):**
```python
requires = [
    'pyramid',
    'waitress',
]
```
- Dependencies yang **harus** ada untuk menjalankan aplikasi
- Akan diinstall di semua environment (dev, staging, production)
- Aplikasi tidak bisa berjalan tanpa dependencies ini

**dev_requires (Development-only dependencies):**
```python
dev_requires = [
    'pyramid_debugtoolbar',
]
```
- Dependencies yang **hanya** diperlukan saat development
- Tidak akan diinstall di production
- Membantu development, tapi bukan requirement untuk aplikasi berjalan

#### Keuntungan Memisahkan Dependencies

**1. Security:**
- Debug toolbar mengekspos informasi sensitif (request data, database queries, dll)
- Tidak aman jika aktif di production
- Dengan extras, tidak akan terinstall di production

**2. Performance:**
- Debug toolbar menambah overhead
- Inject HTML/CSS ke setiap response
- Production tidak memerlukan overhead ini

**3. Size & Installation Time:**
- Mengurangi dependencies yang harus diinstall di production
- Production deployment lebih cepat
- Docker images lebih kecil

**4. Clean Production Environment:**
```bash
# Development
pip install -e ".[dev]"    # Install semua tools

# Production
pip install -e .           # Hanya install yang necessary
```

#### Contoh Dependencies Lain

```python
setup(
    name='tutorial',
    install_requires=[
        'pyramid',          # Production
        'waitress',         # Production
        'sqlalchemy',       # Production
    ],
    extras_require={
        'dev': [
            'pyramid_debugtoolbar',  # Development only
            'pyramid_ipython',       # Development only
        ],
        'test': [
            'pytest',                # Testing only
            'webtest',              # Testing only
            'pytest-cov',           # Testing only
        ],
    },
)
```

### 2. Apa yang terjadi saat introduce bug dan explore interactive debugger?

#### Introduce Bug

**Original code:**
```python
def hello_world(request):
    return Response('<body><h1>Hello World!</h1></body>')
```

**Buggy code:**
```python
def hello_world(request):
    return xResponse('<body><h1>Hello World!</h1></body>')
    #      ^ Bug: xResponse tidak ada
```

#### Yang Terjadi Saat Visit http://localhost:6543/

**1. Nice Traceback Display:**
```
NameError: name 'xResponse' is not defined

Traceback (most recent call last):
  File ".../tutorial/__init__.py", line 7, in hello_world
    return xResponse('<body><h1>Hello World!</h1></body>')
NameError: name 'xResponse' is not defined
```

**2. Interactive Console:**

Di baris paling bawah traceback, ada icon "screen" atau "console" di sebelah kanan.

**Click icon tersebut** untuk membuka interactive Python console di context error tersebut.

#### Apa yang Bisa Dilakukan di Interactive Console?

**Inspect Variables:**
```python
>>> request
<Request GET http://localhost:6543/>

>>> Response
<class 'pyramid.response.Response'>

>>> request.method
'GET'

>>> request.path
'/'

>>> request.url
'http://localhost:6543/'
```

**Explore Request Object:**
```python
>>> request.headers
{'Host': 'localhost:6543', 'User-Agent': '...', ...}

>>> request.params
<MultiDict []>

>>> request.cookies
{}

>>> dir(request)
['GET', 'POST', 'accept', 'cookies', 'headers', ...]
```

**Test Fixes:**
```python
>>> # Test apakah Response bekerja
>>> response = Response('<h1>Test</h1>')
>>> response
<Response 200 OK>

>>> response.text
'<h1>Test</h1>'
```

**Explore Globals:**
```python
>>> globals()
{
    'Response': <class 'pyramid.response.Response'>,
    'Configurator': <class 'pyramid.config.Configurator'>,
    'hello_world': <function hello_world at 0x...>,
    ...
}

>>> locals()
{
    'request': <Request GET http://localhost:6543/>,
}
```

**Execute Python Code:**
```python
>>> import sys
>>> sys.version
'3.10.0 (default, ...'

>>> 2 + 2
4

>>> [i**2 for i in range(5)]
[0, 1, 4, 9, 16]
```

#### Fitur Lain yang Bisa Ditemukan

**1. Stack Frames:**
- Click pada setiap line di traceback
- Explore variables di context berbeda
- Lihat source code di setiap frame

**2. Source Code View:**
- Highlight line yang error
- Lihat context di sekitar error
- Copy code langsung dari traceback

**3. Request Details:**
- HTTP headers lengkap
- Query parameters
- POST data
- Cookies
- Session info

**4. Performance Metrics:**
- Execution time
- Memory usage
- Number of SQL queries (jika pakai SQLAlchemy)

**5. Template Info:**
- Template yang di-render
- Template variables
- Rendering time

#### Workflow Debugging yang Efektif

1. **Error terjadi** → Traceback muncul otomatis
2. **Inspect variables** → Gunakan interactive console
3. **Test fixes** → Coba solusi langsung di console
4. **Fix code** → Edit file Python
5. **Auto-reload** → `--reload` restart aplikasi otomatis
6. **Test lagi** → Refresh browser

#### Tips Interactive Console

- **Tab completion** - Press Tab untuk autocomplete
- **History** - Arrow up/down untuk command history
- **Multi-line** - Shift+Enter untuk multi-line input
- **Help** - `help(object)` untuk documentation
- **Dir** - `dir(object)` untuk list attributes/methods

## Keuntungan Debug Toolbar

| Fitur | Tanpa Toolbar | Dengan Toolbar |
|-------|--------------|----------------|
| Error Display | Generic 500 error | Nice traceback + console |
| Variable Inspect | Print statements | Interactive console |
| Performance | Guess | Metrics displayed |
| SQL Queries | Log files | Real-time display |
| Request Data | Manual logging | One-click view |
| Debug Speed | Slow | Fast & efficient |
