01: Single-File Web Applications

from waitress import serve
from pyramid.config import Configurator
from pyramid.response import Response

def hello_world(request):
    print 'Incoming request' # <-- INI PERUBAHANNYA
    return Response('<body><h1>Hello World!</h1></body>')

if __name__ == '__main__':
    with Configurator() as config:
        config.add_route('hello', '/')
        config.add_view(hello_world, route_name='hello')
        app = config.make_wsgi_app()
    serve(app, host='0.0.0.0', port=6543)
    
Analisis Kode:
1. Baris 11: if __name__ == '__main__': adalah cara Python untuk menandakan, "Mulai eksekusi di sini saat file dijalankan langsung dari command line (baris perintah)," berbeda dengan saat file ini diimpor sebagai modul oleh file lain.
2. Baris 12-14: Baris ini menggunakan configurator Pyramid di dalam sebuah context manager. Tujuannya adalah untuk menghubungkan kode view (fungsi hello_world) ke route (rute) URL tertentu (dalam hal ini, /).
3. Baris 6-8: Ini adalah implementasi dari kode view. Fungsi ini bertugas menghasilkan response (respons) yang akan dikirim kembali ke browser.
4. Baris 15-17: Baris ini mempublikasikan aplikasi WSGI yang telah dibuat (disimpan dalam variabel app) menggunakan server HTTP (dalam contoh ini, waitress).
