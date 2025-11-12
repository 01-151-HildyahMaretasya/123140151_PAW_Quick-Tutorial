# Python Packages untuk Aplikasi Pyramid

## Deskripsi

Tutorial ini menjelaskan cara mengorganisir aplikasi "Hello World" menjadi Python package yang proper dengan struktur project yang lebih terorganisir menggunakan `setup.py`.

Python package adalah cara untuk mengorganisir kumpulan modul dan file menjadi satu unit yang terstruktur. Berikut cara kerjanya:

### Apa itu Python Package?

- **Package** = direktori yang memiliki file khusus bernama `__init__.py`
- Jika direktori ada di `sys.path` dan memiliki `__init__.py`, Python akan mengenalinya sebagai package
- Package bisa dibundle, diinstall, dan didistribusikan menggunakan `setup.py`

### Struktur Pengembangan

Dalam tutorial ini, struktur project mengikuti pola berikut:

1. **Project directory** - folder utama untuk setiap tutorial step
2. **setup.py** - file yang menginject fitur project machinery ke dalam direktori
3. **tutorial subdirectory** - dijadikan Python package dengan menambahkan `__init__.py`
4. **Development mode** - install project menggunakan `pip install -e .`

- Development dilakukan di dalam Python package
- Package tersebut adalah bagian dari sebuah project

## Objektif

- Membuat direktori Python "package" dengan `__init__.py`
- Setup Python "project" minimal dengan `setup.py`
- Install project dalam development mode

## Langkah-langkah

1. **Buat Directory untuk Tutorial**
2. **Buat File `setup.py`**
3. **Install Project dalam Development Mode**

```bash
$VENV/bin/pip install -e .
mkdir tutorial
```

**4. Buat File `__init__.py`**
**5. Buat File Aplikasi**
**6. Jalankan Aplikasi**

```bash
$VENV/bin/python tutorial/app.py
```

**7. Buka Browser**

Akses: `http://localhost:6543/`

## Analisis

### Python Package vs Python Project

**Python Package:**
- Memberikan unit pengembangan yang terorganisir
- Strukturnya menggunakan direktori dengan `__init__.py`

**Python Project:**
- Dikelola melalui `setup.py`
- Memberikan fitur khusus saat package diinstall
- Development mode (`-e .`) = editable mode yang memungkinkan perubahan code langsung terdeteksi tanpa reinstall

### Penjelasan setup.py

```python
requires = [
    'pyramid',
    'waitress',
]
```
List dependencies yang akan otomatis diinstall ketika menjalankan `pip install -e .`

```python
setup(
    name='tutorial',
    install_requires=requires,
)
```
Konfigurasi minimal untuk project dengan nama `tutorial` dan dependencies yang diperlukan.

### Nama Package: "tutorial"

Nama `tutorial` digunakan konsisten di setiap step untuk menghindari pengetikan ulang yang tidak perlu.

### Development Mode (Editable Mode)

Perintah `pip install -e .` menginstall project dalam mode editable:
- Perubahan code langsung aktif tanpa perlu reinstall
- Cocok untuk pengembangan
- Flag `-e` = editable
- `.` = direktori saat ini

## Perbandingan dengan Single-File

| Aspek | Single-File | Python Package |
|-------|-------------|----------------|
| Struktur | 1 file `app.py` | Directory dengan `__init__.py` |
| Dependencies | Manual install | Otomatis via `setup.py` |
| Skalabilitas | Terbatas | Mudah dikembangkan |
| Organisasi | Minimal | Terstruktur |
| Distribution | Sulit | Mudah di-share |

## Keuntungan Menggunakan Package

1. **Organisasi lebih baik** - File terpisah berdasarkan fungsi
2. **Dependency management** - Dependencies tercatat di `setup.py`
3. **Editable mode** - Perubahan langsung aktif tanpa reinstall
4. **Scalability** - Mudah menambah modul baru
5. **Distribution** - Bisa di-package dan di-share ke developer lain
