*****************
Testy jednostkowe
*****************

Testowanie programu pozwala sprawdzić, czy program zachowuje się w pożądany sposób.
Manualne testowanie programu jest czasochłonne.
Dlatego dobrą praktyką jest pisanie automatycznych testów, które mogą zostać szybko wykonane przez komputer.
Testuje się nie tylko całe programy, ale też pojedyncze funkcje czy klasy.

Standardowa biblioteka Pythona posiada moduły ``unittest`` i ``doctest`` ułatwiające pisanie takich testów.
Ponadto, omówiona zostanie biblioteka ``pytest``, która jest popularna pomimo tego, że nie jest częścią standardowej biblioteki.

Unittest
========

Wprowadzenie
------------

``unittest`` jest najpopularniejszą biblioteką stosowaną do pisania testów jednostkowych. Wynika to jej następujących cech:

*   ``unittest`` jest częścią standardowej biblioteki Pythona, co oznacza, że jest dostępna wszędzie, gdzie jest zainstalowany Python.

*   ``unittest`` jest inspirowana biblioteką ``JUnit`` z Javy.
    Osoby, które pisały wcześniej testy w ``JUnit`` bardzo szybko nauczą się pisać testy w Pythonie.
    Ponadto, wykorzystano sprawdzony interfejs.
    Z drugiej strony, wzorowanie się na bibliotece Javy powoduje, że testy są dosyć rozwlekłe i "rozgadane".

*   ``unittest`` potrafi automatycznie znaleźć wszystkie testy i wykonać je.

Przykład
--------

Testy grupuje się w klasy dziedziczące po ``unittest.TestCase``.
Każda metoda zaczynająca się od ``test_`` to jeden test.
Wywołanie ``unittest.main()`` powoduje uruchomienie wszystkich testów (nie jest potrzebne ręczne wymienianie nazw wszystkich testów).

Powszechnie przyjętą konwencją jest separacja testów i testowanego kodu, poprzez umieszczenie ich w osobnych plikach.
Dodatkowo, testy umieszcza się w osobnym katalogu, a nie w tym samym katalogu co testowany moduł.
Dzięki temu możliwa jest instalacja pakietu bez instalowania bibliotek wykorzystywanych tylko przez testy.

.. code-block:: python

    # converters.py
    import re

    def url_converter(url):
        if not url:
            raise ValueError("Empty url")

        pattern = r'(http://[\w-]+(\.[\w-]+)*((/[\w-]*)?))'
        regexp = re.compile(pattern)
        return regexp.sub('<a href="\\1">\\1</a>', url)

    # test_converters.py
    import unittest

    from converters import url_converter

    class UrlConverterTests(unittest.TestCase):
        def test_converts_url_to_ahref(self):
            url = "http://www.python.org"
            expected_ahref = '<a href="http://www.python.org">http://www.python.org</a>'
            result = url_converter(url)
            self.assertEqual(result, expected_ahref)

    if __name__ == "__main__":
        unittest.main()

Ostatnie dwie linie powyższego kodu gwarantują uruchomienie testów, ale tylko gdy plik z kodem zostanie bezpośrednio uruchomiony, a nie zaimportowany.

.. code-block:: bash

    $ python test_converters.py

Uruchamianie testów
-------------------

Możemy wykonać testy wpisując w konsoli.

.. code-block:: bash

    $ python -m unittest test_my_module.TestAdd

Ogromną zaletą biblioteki ``unittest`` jest możliwość uruchomienia wszystkich testów bez określania, gdzie się one znajdują.
W takiej sytuacji biblioteka ``unittest`` poszukuje testów we wszystkich plikach znajdujących się w aktualnym katalogu, podkatalogach, podkatalogach podkatalogów itd.

.. code-block:: bash

    $ python -m unittest

Przydatnym przełącznikiem jest ``--failfast``  (lub ``-f``), który zatrzymuje wykonywanie testów po pierwszym teście, który nie przeszedł.
Ułatwia to skupienie się na naprawieniu testu, ponieważ na wyjściu pojawiają się informacje dotyczące tylko jednego nieprzechodzącego testu.

.. code-block:: bash

    $ python -m unittest --failfast

Niektóre środowiska programistyczne, takie jak PyCharm, posiadają wsparcie dla uruchamiania testów.

``setUp`` i ``tearDown``
------------------------

Jeżeli na początku lub na końcu każdego testu wykonujemy operacje, które powtarzają się w innych testach, wtedy można umieścić je w metodach ``setUp`` i ``tearDown``.
``setUp`` jest metodą wykonywaną na początku każdego testu, natomiast ``tearDown`` -- po wykonaniu testu, niezależnie od tego, czy test przeszedł, czy nie.

Jest to świetne miejsce na:

- uzyskanie zasobów, które są potrzebne w każdym teście (np. połączenie z bazą danych),

- skonfigurowanie środowiska w ten sam sposób dla każdego testu (np. w ``setUp`` -- utworzenie przykładowych tabel i rekordów w bazie danych, a w ``tearDown`` -- "posprzątanie" po teście, tzn. wyczyszczenie testowej bazy danych).

.. code-block:: python

    class TestAdd(unittest.TestCase):
        def setUp(self):
            print("setUp")

        def tearDown(self):
            print("tearDown")

        def test_one(self):
            print("test one")

        def test_two(self):
            print("test two")

.. code-block:: pycon

    setUp
    test one
    tearDown
    setUp
    test two
    tearDown

Warto zauważyć, że, generalnie rzecz biorąc, w Pythonie nazwy metod piszemy małymi literami, a poszczególne słowa rozdzielamy podkreślnikami, np. ``tear_down``, ``set_up``.
Jednak konwencja ta nie zawsze jest przestrzegana, nawet w obrębie biblioteki standardowej, czego przykładem jest ``unittest``.
Jest ona wzorowana na bibliotece ``JUnit`` napisanej w Javie, gdzie obowiązuje konwencja "camelCase".

``self.assert*``
----------------

W środku każdego testu możemy wykorzystać szereg metod zaczynających się od ``assert``, np. ``assertEqual``.
Test przechodzi, jeżeli wszystkie takie asercje są prawdziwe.
Jeżeli chociaż jedna taka asercja nie będzie spełniona, wówczas wykonywanie testu jest natychmiast przerywane i wykonywana jest metoda ``tearDown``.

Najczęściej wykorzystywane asercje to:

* ``self.assertTrue(condition)`` i ``self.assertFalse(condition)``,

* ``self.assertEqual(got, expected)`` i ``self.assertNotEqual(got, expected)``,

* ``self.assertIn(element, collection)`` i ``self.assertNotIn(element, collection)``,

* ``self.assertIsInstance(obj, class_)`` i ``self.assertNotIsInstance(obj, class_)``.

Do sprawdzenia, czy dany blok kodu rzuca wyjątek, można użyć ``self.assertRaises(ExceptionType)``:

.. code-block:: python

    class UrlConverterTests(unittest.TestCase):
        def test_raises_exception_for_empty_string(self):
            url = ""

            with self.assertRaises(ValueError):
                url_converter(url)

Jeżeli sprawdzenie typu rzuconego wyjątku to za mało, możemy uzyskać do niego dostęp:

.. code-block:: python

    class UrlConverterTests(unittest.TestCase):
        def test_raises_exception_for_empty_string(self):
            url = ""

            with self.assertRaises(ValueError) as ex:
                url_converter(url)
            self.assertEqual(ex.message, 'Empty url')

Doctest
=======

``doctest`` jest częścią standardowej biblioteki Pythona, ale oferuje zupełnie inne podejście do testowania.
Zamiast umieszczać testy w osobnych plikach, testy można umieścić w docstringu testowanej funkcji, klasy lub metody.
Takie podejście ma kilka przewag nad ``unittest``:

-   Ponieważ testy są częścią docstringa, pełnią wówczas jednocześnie rolę dokumentacji.
    Jeżeli są to krótkie testy, jest to wówczas bardzo dobra dokumentacja.

-   Testy są trzymane blisko obiektu, który jest testowany, co jest generalnie pożądane, ponieważ nie ma potrzeby przeskakiwania między plikami.

Niestety, to podejście ma też pewne istotne wady.
Przede wszystkim, takie podejście nie skaluje się wraz z coraz bardziej skomplikowanymi testami, co oznacza, że sprawdza się ono głównie w przypadku krótkich testów dla prostych funkcji.

.. code-block:: python

    import doctest
    import math

    def factorial(n):
        """Return the factorial of n, an exact integer >= 0.

        >>> factorial(3)
        6
        >>> factorial(30)
        265252859812191058636308480000000
        >>> factorial(-1)
        Traceback (most recent call last):
            ...
        ValueError: n must be >= 0
        >>> [factorial(n) for n in range(6)]
        [1, 1, 2, 6, 24, 120]
        """

        if not n >= 0:
            raise ValueError("n must be >= 0")
        result = 1
        factor = 2
        while factor <= n:
            result *= factor
            factor += 1
        return result

    # Uruchomienie testów
    if __name__ == "__main__":
        doctest.testmod()


Pytest
======

Wprowadzenie
------------

Zasadniczą wadą biblioteki ``unittest`` jest rozwlekłość pisanych w nich testów.
Każdy test musi być metodą umieszczoną w klasie.
Nazwy metod takie jak ``self.assertEqual`` są dosyć rozwlekłe.
Czy można pisać krótsze, czytelniejsze testy?
Odpowiedź brzmi tak, wystarczy doinstalować bibliotekę ``pytest``.

.. code-block:: bash

    $ pip install pytest

``pytest`` pozwala umieścić testy w funkcjach, których nazwa zaczyna się od ``test_``.
Ponadto, można użyć asercji zamiast metod takich jak ``self.assertEqual``.

Przykład
--------

.. code-block:: python

    # my_module.py
    def inc(x):
        if x == 0:
            raise ValueError('Zero is not valid value.')
        return x + 1

    # test_my_module.py
    import pytest

    from my_module import inc

    def test_answer():
        # Poniższa asercja jest znacznie czytelniejsza niż self.assertEqual(inc(3), 4)
        assert inc(3) == 4

    def test_invalid_argument():
        with pytest.raises(ValueError):
            func(0)

Uruchamianie testów
-------------------

``pytest``, podobnie jak ``unittest`` potrafi sam znaleźć wszystkie testy w aktualnym katalogu i podkatalogach.

.. code-block:: bash

    $ pytest

    platform linux -- Python 3.5.2, pytest-3.0.4, py-1.4.31, pluggy-0.4.0
    rootdir: doc, inifile:
    collected 2 items

    test_my_module.py ..

    2 passed in 0.04 seconds
