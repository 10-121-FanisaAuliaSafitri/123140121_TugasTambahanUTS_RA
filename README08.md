08: HTML Generation With Templating

1. Let's begin by using the previous package as a starting point for a new project:

cd ..; cp -r views templating; cd templating
2. This step depends on pyramid_chameleon, so add it as a dependency in templating/setup.py:

from setuptools import setup

# List of dependencies installed via `pip install -e .`
# by virtue of the Setuptools `install_requires` value below.
requires = [
    'pyramid',
    'pyramid_chameleon',
    'waitress',
]

# List of dependencies installed via `pip install -e ".[dev]"`
# by virtue of the Setuptools `extras_require` value in the Python
# dictionary below.
dev_requires = [
    'pyramid_debugtoolbar',
    'pytest',
    'webtest',
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
3. Now we can activate the development-mode distribution:

$VENV/bin/pip install -e .
4. We need to connect pyramid_chameleon as a renderer by making a call in the setup of templating/tutorial/__init__.py:

from pyramid.config import Configurator


def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_chameleon')
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.scan('.views')
    return config.make_wsgi_app()
5. Our templating/tutorial/views.py no longer has HTML in it:

from pyramid.view import view_config


# First view, available at http://localhost:6543/
@view_config(route_name='home', renderer='home.pt')
def home(request):
    return {'name': 'Home View'}


# /howdy
@view_config(route_name='hello', renderer='home.pt')
def hello(request):
    return {'name': 'Hello View'}
6. Instead we have templating/tutorial/home.pt as a template:

<!DOCTYPE html>
<html lang="en">
<head>
    <title>Quick Tutorial: ${name}</title>
</head>
<body>
<h1>Hi ${name}</h1>
</body>
</html>
7. For convenience, change templating/development.ini to reload templates automatically with pyramid.reload_templates:

[app:main]
use = egg:tutorial
pyramid.reload_templates = true
pyramid.includes =
    pyramid_debugtoolbar

[server:main]
use = egg:waitress#main
listen = localhost:6543
8. Our unit tests in templating/tutorial/tests.py can focus on data:

import unittest

from pyramid import testing


class TutorialViewTests(unittest.TestCase):
    def setUp(self):
        self.config = testing.setUp()

    def tearDown(self):
        testing.tearDown()

    def test_home(self):
        from .views import home

        request = testing.DummyRequest()
        response = home(request)
        # Our view now returns data
        self.assertEqual('Home View', response['name'])

    def test_hello(self):
        from .views import hello

        request = testing.DummyRequest()
        response = hello(request)
        # Our view now returns data
        self.assertEqual('Hello View', response['name'])


class TutorialFunctionalTests(unittest.TestCase):
    def setUp(self):
        from tutorial import main
        app = main({})
        from webtest import TestApp

        self.testapp = TestApp(app)

    def test_home(self):
        res = self.testapp.get('/', status=200)
        self.assertIn(b'<h1>Hi Home View', res.body)

    def test_hello(self):
        res = self.testapp.get('/howdy', status=200)
        self.assertIn(b'<h1>Hi Hello View', res.body)
8. Now run the tests:

$VENV/bin/pytest tutorial/tests.py -q
....
4 passed in 0.46 seconds
9. Run your Pyramid application with:

$VENV/bin/pserve development.ini --reload
10. Open http://localhost:6543/ and http://localhost:6543/howdy in your browser.

Analisis
Pemisahan yang Lebih Baik
Struktur kode sekarang terlihat jauh lebih baik. View kita (views.py) sekarang fokus murni pada kode Python (logika bisnis), dan tidak lagi berisi string HTML.

Peran Renderer
1. Decorator @view_config sekarang memiliki parameter renderer yang menunjuk ke file template (misalnya, renderer='home.pt').
2. Fungsi view tidak lagi membuat objek Response secara manual. Sebaliknya, view sekarang cukup mengembalikan data (dalam contoh ini, sebuah dictionary Python).
3. Pyramid kemudian secara otomatis mengambil data ini dan meneruskannya ke renderer (Chameleon) dan file template (home.pt) untuk menghasilkan respons HTML akhir.

Template yang Dapat Digunakan Kembali
Perlu dicatat bahwa dalam contoh ini, kedua view (home dan hello) menggunakan file template yang sama (home.pt), tetapi menyediakan data yang berbeda untuk itu.

Dampak pada Pengujian
Perubahan ini berdampak positif pada unit test kita.

1. Unit test (TutorialViewTests) sekarang dapat fokus pada kontrak berbasis data (data-oriented contract) dengan kode view.
2. Kita hanya perlu menguji apakah view mengembalikan dictionary dengan data yang benar (misalnya response['name']), yang jauh lebih sederhana, lebih bersih, dan lebih stabil daripada harus mengurai dan memvalidasi seluruh string HTML.
3. Pengujian HTML yang di-render secara penuh diserahkan kepada functional test (TutorialFunctionalTests), yang memang merupakan tugasnya.
