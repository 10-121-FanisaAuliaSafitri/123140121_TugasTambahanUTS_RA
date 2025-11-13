06: Functional Testing dengan WebTest

Struktur File
Struktur direktori untuk langkah ini (di dalam folder functional_testing/):

functional_testing/
├── setup.py           (Dimodifikasi)
├── development.ini    (Tidak berubah)
└── tutorial/
    ├── __init__.py    (Tidak berubah)
    └── tests.py       (Dimodifikasi, tes baru ditambahkan)

Kode
1. functional_testing/setup.py
File setup.py dimodifikasi untuk menambahkan webtest ke dalam daftar dev_requires.

Python

from setuptools import setup

# List of dependencies installed via `pip install -e .`
# by virtue of the Setuptools `install_requires` value below.
requires = [
    'pyramid',
    'waitress',
]

# List of dependencies installed via `pip install -e ".[dev]"`
# by virtue of the Setuptools `extras_require` value in the Python
# dictionary below.
dev_requires = [
    'pyramid_debugtoolbar',
    'pytest',
    'webtest',  # <-- DEPENDENSI BARU DITAMBAHKAN
]

setup(
    name='tutorial',
    install_requires=requires,
    extras_require={
        'dev': dev_requires,
    },
    entry_points={
        'paste.app_factory': [
            'main = tutorial:main'
        ],
    },
)
2. functional_testing/tutorial/tests.py (Dimodifikasi)
File tes kita sekarang diperluas untuk menyertakan class tes fungsional baru (TutorialFunctionalTests) di samping unit test yang sudah ada.

Python

import unittest
from pyramid import testing

# --- UNIT TESTS (DARI LANGKAH 05) ---
class TutorialViewTests(unittest.TestCase):
    def setUp(self):
        self.config = testing.setUp()

    def tearDown(self):
        testing.tearDown()

    def test_hello_world(self):
        from tutorial import hello_world # Menguji fungsi hello_world

        request = testing.DummyRequest()
        response = hello_world(request)
        self.assertEqual(response.status_code, 200)

# --- FUNCTIONAL TESTS (BARU DI LANGKAH 06) ---
class TutorialFunctionalTests(unittest.TestCase):
    def setUp(self):
        # Impor 'main' dari __init__.py
        from tutorial import main
        # Buat aplikasi WSGI yang sesungguhnya
        app = main({})
        from webtest import TestApp
        
        # Bungkus aplikasi dengan WebTest
        self.testapp = TestApp(app)

    def test_hello_world(self):
        # Simulasikan request GET ke '/'
        res = self.testapp.get('/', status=200)
        
        # Periksa apakah body respons mengandung HTML yang diharapkan
        # 'b' menandakan 'bytes string'
        self.assertIn(b'<h1>Hello World!</h1>', res.body)
Analisis
Pengujian End-to-End
Kita sekarang telah memiliki jenis pengujian end-to-end (menyeluruh) yang dicari. WebTest memungkinkan kita untuk dengan mudah memperluas pendekatan pengujian berbasis pytest yang sudah ada, dengan menambahkan tes fungsional.

Hasil Gabungan
Tes-tes fungsional baru ini dilaporkan dalam output yang sama (bersamaan dengan unit test) saat kita menjalankan pytest.

Cepat dan Efisien
Poin pentingnya, tes-tes baru ini tidak hanya mencakup templating (memeriksa konten HTML aktual yang dilihat pengguna), tetapi juga tidak meningkatkan waktu eksekusi tes secara drastis. Ini karena WebTest mensimulasikan request secara internal tanpa perlu menjalankan server HTTP yang sesungguhnya.

Perbedaan Utama: Unit vs. Functional Test
1. Unit Test (TutorialViewTests): Menguji "unit" kode (fungsi hello_world) secara terisolasi. Kita membuat DummyRequest dan memanggil fungsi itu secara langsung. Kita tidak tahu apakah routing atau templating berfungsi.
2. Functional Test (TutorialFunctionalTests): Menguji aplikasi secara keseluruhan. Di setUp, kita membuat instance aplikasi WSGI yang sesungguhnya (via main({})) dan membungkusnya dengan TestApp. Tes ini (self.testapp.get('/')) mensimulasikan request HTTP penuh dan memeriksa HTML (res.body) yang dikembalikan.
