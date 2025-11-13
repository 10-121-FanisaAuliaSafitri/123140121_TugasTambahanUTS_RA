02: Python Packages for Pyramid Applications

Package/setup.py
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
)

Package/tutorial/__init__.py
# package
package/tutorial/app.py
Ini adalah kode aplikasi Anda dari langkah 01, sekarang dipindahkan ke dalam package tutorial. Isinya masih sama persis.

Python

from waitress import serve
from pyramid.config import Configurator
from pyramid.response import Response

def hello_world(request):
    print('Incoming request')
    return Response('<body><h1>Hello World!</h1></body>')

if __name__ == '__main__':
    with Configurator() as config:
        config.add_route('hello', '/')
        config.add_view(hello_world, route_name='hello')
        app = config.make_wsgi_app()
    serve(app, host='0.0.0.0', port=6543)

Analisis
1. Organisasi: Paket Python (Python packages) memberi kita unit pengembangan proyek yang terorganisir.
2. Fitur Proyek: Proyek Python (melalui file setup.py) memberikan kita fitur-fitur khusus saat paket diinstal. Dalam kasus ini, kita menginstalnya dalam mode pengembangan lokal (disebut juga local editable mode), yang ditandai dengan bendera -e.
3. Struktur: Dalam langkah ini, kita memiliki paket Python yang disebut tutorial. Di atas direktori tutorial ini, terdapat file setup.py yang menangani pengemasan (packaging) proyek.
4. Tidak Ada Perubahan Logika: Selain strukturisasi ini, tidak ada yang berubah dari aplikasi kita. Kita hanya mengubahnya menjadi paket Python dengan setup.py dan menginstalnya dalam mode pengembangan.
5. Catatan Peringatan: Cara kita menjalankan aplikasi saat ini (python tutorial/app.py) dianggap "agak aneh" (an odd duck). Ini bukan praktik umum dan hanya dilakukan untuk tujuan tutorial (menunjukkan cara kerja selangkah demi selangkah). Menjalankan modul Python di dalam paket secara langsung sebagai skrip umumnya adalah ide yang buruk.
