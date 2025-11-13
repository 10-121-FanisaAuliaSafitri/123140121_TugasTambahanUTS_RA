03: Konfigurasi Aplikasi dengan File .ini
Langkah ini mengubah proyek kita dari skrip yang dijalankan langsung (python app.py) menjadi aplikasi yang dikonfigurasi dan dijalankan menggunakan pserve dan file development.ini.

Struktur File
Struktur direktori untuk langkah ini (di dalam folder ini/):

ini/
├── setup.py           (Dimodifikasi)
├── development.ini    (File Baru)
└── tutorial/
    ├── __init__.py    (Dimodifikasi)
    └── app.py         (Dihapus)
Kode
1. ini/setup.py
File setup.py dimodifikasi untuk menambahkan entry_points. Ini memberi tahu pserve (dan ekosistem setuptools) bahwa paket tutorial kita menyediakan "pabrik" aplikasi bernama main, yang dapat ditemukan di fungsi main di dalam paket tutorial (yaitu, di tutorial/__init__.py).

Python

from setuptools import setup

# List of dependencies installed via `pip install -e .`
# by virtue of the Setuptools `install_requires` value below.
requires = [
    'pyramid',
    'waitress',
]

setup(
    name='tutorial',
    install_requires=requires,
    # PERUBAHAN DIMULAI DI SINI
    entry_points={
        'paste.app_factory': [
            'main = tutorial:main'
        ],
    },
)
2. ini/development.ini (File Baru)
Ini adalah file konfigurasi yang akan dibaca oleh pserve.

[app:main] memberi tahu Pyramid untuk menggunakan aplikasi dari project tutorial (yang didefinisikan di entry_points dalam setup.py).

[server:main] memberi tahu pserve untuk menggunakan waitress sebagai server HTTP, yang mendengarkan di localhost pada port 6543.

Ini, TOML

[app:main]
use = egg:tutorial

[server:main]
use = egg:waitress#main
listen = localhost:6543
3. ini/tutorial/__init__.py (Dimodifikasi)
Logika startup aplikasi dari app.py lama sekarang dipindahkan ke sini, dibungkus dalam fungsi main yang sesuai dengan entry_points.

Python

from pyramid.config import Configurator
from pyramid.response import Response

def hello_world(request):
    """View function for the hello world page."""
    return Response('<body><h1>Hello World!</h1></body>')

def main(global_config, **settings):
    """ This function returns a Pyramid WSGI application.
    """
    config = Configurator(settings=settings)
    config.add_route('hello', '/')
    config.add_view(hello_world, route_name='hello')
    return config.make_wsgi_app()
4. Penghapusan tutorial/app.py
File tutorial/app.py dari langkah 02 sudah tidak diperlukan lagi dan harus dihapus. Logikanya telah dipindahkan ke tutorial/__init__.py.

Bash

# Jalankan ini di dalam direktori 'ini/'
rm tutorial/app.py

Analisis:
Proses Startup Aplikasi
File development.ini sekarang dibaca oleh pserve dan berfungsi untuk melakukan bootstrap (proses awal) pada aplikasi Anda. Prosesnya berjalan sebagai berikut:
1. pserve mencari bagian [app:main] di dalam file .ini dan menemukan perintah use = egg:tutorial.
2. File setup.py proyek telah mendefinisikan entry point (yaitu tutorial:main).
3. Ini merujuk pada fungsi main yang ada di dalam file tutorial/__init__.py.
4. Fungsi main tersebut kemudian dijalankan untuk membuat objek aplikasi WSGI.

Fungsi Lain dari File .ini
File .ini juga memiliki dua fungsi penting lainnya:
1. Mengonfigurasi Server WSGI:
    1. Bagian [server:main] digunakan untuk menentukan server WSGI mana yang akan digunakan (waitress).
    2. Bagian ini juga mengatur konfigurasi server. Perintah listen = localhost:6543 memberitahu waitress untuk mendengarkan di host localhost pada port 6543.
2. Mengonfigurasi Logging Python:
    1. Pyramid menggunakan fitur logging standar Python. File .ini adalah tempat standar untuk menyediakan konfigurasi logging ini (meskipun belum dikonfigurasi secara eksplisit di sini, ini adalah tempatnya).

Struktur Kode dan Perintah
1. Pemindahan Kode: Kode startup aplikasi dipindahkan dari app.py (yang sekarang dihapus) ke file tutorial/__init__.py. Ini adalah gaya yang umum (common style) dalam Pyramid.
2. Perintah pserve --reload: Opsi --reload sangat berguna selama pengembangan. Ini memberitahu pserve untuk mengawasi sistem file. Jika ada perubahan pada file Python atau .ini, pserve akan secara otomatis me-restart aplikasi.
