ini/setup.py
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

ini/development.ini
[app:main]
use = egg:tutorial

[server:main]
use = egg:waitress#main
listen = localhost:6543

ini/tutorial/__init__.py
from pyramid.config import Configurator
from pyramid.response import Response

def hello_world(request):
    return Response('<body><h1>Hello World!</h1></body>')

def main(global_config, **settings):
    """ This function returns a Pyramid WSGI application.
    """
    config = Configurator(settings=settings)
    config.add_route('hello', '/')
    config.add_view(hello_world, route_name='hello')
    return config.make_wsgi_app()

app.py
rm tutorial/app.py

Analisis:
Proses Startup Aplikasi
1. File development.ini sekarang dibaca oleh pserve dan berfungsi untuk melakukan bootstrap (proses awal) pada aplikasi Anda. Prosesnya berjalan sebagai berikut: pserve mencari bagian [app:main] di dalam file .ini dan menemukan perintah use = egg:tutorial.
2. File setup.py proyek telah mendefinisikan entry point (pada baris 10-13) untuk "main" proyek, yaitu tutorial:main.
3. Ini merujuk pada fungsi main yang ada di dalam file tutorial/__init__.py.
4. Fungsi main tersebut kemudian dijalankan, dengan nilai-nilai dari beberapa bagian file .ini diteruskan sebagai parameter.

Fungsi Lain dari File .ini
File .ini juga memiliki dua fungsi penting lainnya:
1. Mengonfigurasi Server WSGI:
    1. Bagian [server:main] digunakan untuk menentukan server WSGI mana yang akan digunakan (dalam hal ini waitress).
    2. Bagian ini juga mengatur konfigurasi server. Perintah listen = localhost:6543 memberitahu waitress untuk mendengarkan di host localhost pada port 6543.
2. Mengonfigurasi Logging Python:
    1. Pyramid menggunakan fitur logging standar Python.
    2. File .ini menyediakan konfigurasi ini, yang menghasilkan output log di konsol saat aplikasi dimulai dan setiap kali ada permintaan (request) masuk.

Struktur Kode dan Perintah
1. Pemindahan Kode: Kode startup aplikasi dipindahkan dari app.py (yang sekarang dihapus) ke file tutorial/__init__.py. Ini bukan keharusan, tetapi merupakan gaya yang umum (common style) dalam Pyramid.
2. Perintah pip install -e .: Perintah ini akan memeriksa setup.py untuk melihat paket apa saja yang dibutuhkan. Ia akan menginstal paket Anda dalam mode editable dan menginstal requirements lain (seperti waitress) yang belum ada di virtual environment.
3. Perintah pserve --reload: Opsi --reload sangat berguna selama pengembangan. Ini memberitahu pserve untuk mengawasi sistem file. Jika ada perubahan pada file yang relevan (seperti file Python atau file .ini), pserve akan secara otomatis me-restart aplikasi.
