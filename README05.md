05: Unit Testing dengan pytest
Struktur File
Struktur direktori untuk langkah ini (di dalam folder unit_testing/):

unit_testing/
├── setup.py           (Dimodifikasi)
├── development.ini    (Tidak berubah)
└── tutorial/
    ├── __init__.py    (Tidak berubah)
    └── tests.py       (File Baru)

Kode
1. unit_testing/setup.py
File setup.py dimodifikasi untuk menambahkan pytest ke dalam daftar dev_requires. Ini membuatnya menjadi dependensi yang hanya diinstal saat kita menjalankan pip install -e ".[dev]".

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
    'pytest',  # <-- DEPENDENSI BARU DITAMBAHKAN
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
2. unit_testing/tutorial/tests.py (File Baru)
Ini adalah file tes baru kita. File ini berisi test case yang menggunakan modul unittest bawaan Python dan helper dari pyramid.testing.

Python

import unittest
from pyramid import testing

class TutorialViewTests(unittest.TestCase):
    def setUp(self):
        # Menyiapkan lingkungan testing Pyramid minimal
        self.config = testing.setUp()

    def tearDown(self):
        # Membersihkan setelah tes selesai
        testing.tearDown()

    def test_hello_world(self):
        # Impor view yang ingin kita tes
        # (Saat ini, 'hello_world' ada di tutorial/__init__.py)
        from tutorial import hello_world 

        # Buat request palsu (dummy) untuk dikirim ke view
        request = testing.DummyRequest()
        
        # Panggil view secara langsung
        response = hello_world(request)
        
        # Periksa apakah respons memiliki status code 200 OK
        self.assertEqual(response.status_code, 200)

Analisis
Pengenalan Unit Test
1. File tests.py kita mengimpor framework unit testing standar bawaan Python (modul unittest).
2. Untuk mempermudah penulisan tes yang berorientasi Pyramid, Pyramid menyediakan beberapa helper (fungsi pembantu) dalam modul pyramid.testing. Fungsi-fungsi ini kita gunakan dalam metode setUp (persiapan tes) dan tearDown (pembersihan setelah tes).
3. Tes yang kita tulis (test_hello_world) mengimpor view (hello_world), membuat request palsu (DummyRequest), dan memeriksa apakah view tersebut mengembalikan respons seperti yang diharapkan.

Isolasi Unit Tes
1. Tes test_hello_world adalah contoh unit test sederhana.
2. Perhatikan bahwa kita mengimpor view (from tutorial import hello_world) di dalam metode tes itu sendiri, bukan di bagian atas file (level modul) seperti kode Python pada umumnya.
3. Alasannya adalah untuk menjaga isolasi setiap "unit" tes. Melakukan impor di bagian atas file dapat menimbulkan efek samping yang mungkin merusak tes lain. Prinsip unit testing adalah mengisolasi setiap tes semaksimal mungkin.

Alur Tes dan setUp/tearDown
1. Tes ini membuat sebuah request web palsu, kemudian memanggil view kita dengan request tersebut.
2. Setelah itu, kita menguji status code HTTP pada respons untuk memastikan nilainya sesuai dengan ekspektasi kita (dalam hal ini, 200).

Catatan Penting: Penggunaan pyramid.testing.setUp() dan pyramid.testing.tearDown() dalam contoh khusus ini sebenarnya tidak diperlukan. Fungsi-fungsi tersebut hanya benar-benar dibutuhkan jika tes Anda perlu menggunakan objek config (yang merupakan Configurator) untuk menambahkan sesuatu ke status konfigurasi (misalnya, mendaftarkan route atau view lain) sebelum pengujian dijalankan.
