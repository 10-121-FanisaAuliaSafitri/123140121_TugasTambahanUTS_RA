07: Penanganan Web Dasar dengan Views

Struktur File
Struktur direktori untuk langkah ini (di dalam folder views/):

views/
├── setup.py           (Tidak berubah)
├── development.ini    (Tidak berubah)
└── tutorial/
    ├── __init__.py    (Dimodifikasi)
    ├── views.py       (File Baru)
    └── tests.py       (Dimodifikasi)

Kode
1. views/tutorial/__init__.py (Dimodifikasi)
File __init__.py kita menjadi jauh lebih ramping. Logika view telah dihapus. Sebagai gantinya, kita mendaftarkan route baru dan memanggil config.scan('.views'). Perintah scan ini akan secara otomatis menemukan dan mendaftarkan view yang memiliki decorator @view_config di modul views.py.

Python

from pyramid.config import Configurator

def main(global_config, **settings):
    """ This function returns a Pyramid WSGI application.
    """
    config = Configurator(settings=settings)
    
    # Mendaftarkan dua route
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    
    # Memindai modul .views untuk menemukan view
    config.scan('.views')
    
    return config.make_wsgi_app()
2. views/tutorial/views.py (File Baru)
Ini adalah modul baru yang berisi semua fungsi view kita. Perhatikan decorator @view_config di atas setiap fungsi. Decorator inilah yang menghubungkan route (yang didefinisikan dalam __init__.py) ke fungsi view yang sesuai.

Python

from pyramid.response import Response
from pyramid.view import view_config

# View pertama, terhubung ke route 'home' (URL: /)
@view_config(route_name='home')
def home(request):
    return Response('<body>Visit <a href="/howdy">hello</a></body>')

# View kedua, terhubung ke route 'hello' (URL: /howdy)
@view_config(route_name='hello')
def hello(request):
    return Response('<body>Go back <a href="/">home</a></body>')
3. views/tutorial/tests.py (Dimodifikasi)
File tes diperbarui untuk menguji kedua view (home dan hello), baik sebagai unit test maupun functional test.

Python

import unittest
from pyramid import testing

class TutorialViewTests(unittest.TestCase):
    def setUp(self):
        self.config = testing.setUp()

    def tearDown(self):
        testing.tearDown()

    def test_home(self):
        # Impor view dari modul .views
        from .views import home

        request = testing.DummyRequest()
        response = home(request)
        self.assertEqual(response.status_code, 200)
        self.assertIn(b'Visit', response.body)

    def test_hello(self):
        # Impor view dari modul .views
        from .views import hello

        request = testing.DummyRequest()
        response = hello(request)
        self.assertEqual(response.status_code, 200)
        self.assertIn(b'Go back', response.body)

class TutorialFunctionalTests(unittest.TestCase):
    def setUp(self):
        from tutorial import main
        app = main({})
        from webtest import TestApp
        self.testapp = TestApp(app)

    def test_home(self):
        # Tes URL root '/'
        res = self.testapp.get('/', status=200)
        self.assertIn(b'<body>Visit', res.body)

    def test_hello(self):
        # Tes URL '/howdy'
        res = self.testapp.get('/howdy', status=200)
        self.assertIn(b'<body>Go back', res.body)

Analisis
Pemisahan Logika
1. Kode View Dipindahkan: Kode view (home dan hello) telah dipindahkan keluar dari file startup aplikasi (tutorial/__init__.py) ke modul views.py mereka sendiri.
2. Registrasi View Dipindahkan: Registrasi view (penghubungan route ke view) sekarang dilakukan melalui decorator @view_config di dalam views.py.
3. Pemindaian (Scanning): File __init__.py sekarang menggunakan config.scan('.views') untuk secara otomatis menemukan dan mendaftarkan view tersebut saat startup.

Imperatif vs. Deklaratif
Langkah ini memperkenalkan dua gaya konfigurasi Pyramid:
1. Imperatif: Menggunakan metode seperti config.add_view langsung di dalam kode __init__.py (seperti pada langkah-langkah sebelumnya).
2. Deklaratif: Menggunakan decorator Python (seperti @view_config) tepat di atas fungsi view di modul views.py.

Kedua pendekatan ini menghasilkan konfigurasi akhir yang sama. Pilihan di antara keduanya biasanya hanya masalah selera dan gaya pengkodean. Gaya deklaratif seringkali lebih disukai karena menempatkan konfigurasi view tepat di samping kode view itu sendiri.

URL, Route, dan View
Langkah ini juga menunjukkan bahwa tiga hal berikut dapat memiliki nama yang berbeda:
1. URL yang dilihat pengguna (misalnya /howdy)
2. Nama Route (nama internal untuk memetakan URL ke view, misalnya hello)
3. Nama Fungsi View (nama fungsi Python, misalnya hello)
