****************
Moduły i pakiety
****************

Moduły
======

Wprowadzenie
------------

Modułem jest plik źródłowy z rozszerzeniem *.py* zawierający kod Pythona.
Moduł może zawierać definicje funkcji, klas, a także niezależny od nich kod.
Moduł może zawierać dokumentację informującą o sposobie jego działania i zastosowaniach.

Wewnątrz modułu jego nazwa (nazwa pliku bez rozszerzenia .py) jest dostępna pod globalną zmienną *__name__*, pod warunkiem że moduł jest importowany.
Jeśli jednak moduł nie jest importowany, a uruchomiony z konsoli, wówczas ``__name__ == "__main__"``.
Dlatego często występującym idiomem jest sprawdzenie, czy moduł został zaimportowany, czy też uruchomiony:

.. code-block:: python

    if __name__ == "__main__":
        ...  # kod, który ma być wykonany tylko, gdy moduł jest uruchamiany, a nie importowany

Moduł po zaimportowaniu jest obiektem.

W dalszej części rozdziału będziemy posługiwać się modułem umieszczonym w pliku *fibo.py*:

.. code-block:: python

    # Fibonacci numbers module
    def fib(n): # wypisuje na standardowe wyjście ciąg Fibonacciego do n
        a, b = 0, 1
        while b < n:
            print(b, end=' ')
            a, b = b, a+b

    def fib2(n): # zwraca ciąg Fibonacciego do n, jako listę
        result = []
        a, b = 0, 1
        while b < n:
            result.append(b)
            a, b = b, a+b
        return result

Import modułów
--------------

Przed użyciem modułu należy go zaimportować.

Istnieją trzy sposoby importu:

*   Najprostszy, służący do zaimportowania całego modułu:

.. code-block:: pycon

    >>> import fibo
    >>> fibo.fib(1000)
    1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987
    >>> fibo.fib2(1000)
    1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987
    >>> fibo.__name__
    'fibo'

*   Możliwe jest zaimportowanie jedynie określonych funkcji lub klas z danego modułu.
    Sam moduł ``fibo`` nie jest wtedy dostępny.
    Dostępna jest tylko zaimportowana funkcja ``fib``.

.. code-block:: pycon

    >>> from fibo import fib
    >>> fib(500)
    1 1 2 3 5 8 13 21 34 55 89 144 233 377
    >>> fib2(500)  # funkcja fib2 nie została zaimportowana
    NameError                                 Traceback (most recent call last)
    ...
    NameError: name 'fib2' is not defined
    >>> fibo  # moduł fibo nie jest dostępny
    NameError                                 Traceback (most recent call last)
    ...
    NameError: name 'fibo' is not defined

*   Możliwe jest zaimportowanie od razu wszystkich obiektów zdefiniowanych w danym module, bez ich wyliczania.

    Nie jest to jednak rekomendowane, ponieważ nie mamy kontroli nad tym, jakie dokładnie obiekty zostaną zaimportowane.
    W efekcie, możliwa jest sytuacja, w której niechcący przesłaniamy zaimportowaną lub zdefiniowaną wcześniej funkcję lub klasę.

.. code-block:: pycon

    >>> def fib(n):
    ...     print('Ta metoda zostanie przez przypadek przesłonięta.')
    ...
    >>> from fibo import *
    >>> fib(500)  # wykonywana jest funkcja fib z modułu fibo, zamiast lokalnej funkcji
    1 1 2 3 5 8 13 21 34 55 89 144 233 377
    >>> fib2(500)
    [1 1 2 3 5 8 13 21 34 55 89 144 233 377]
    >>> fibo  # sam moduł nie jest dostępny
    NameError                                 Traceback (most recent call last)
    ...
    NameError: name 'fibo' is not defined

W dwóch pierwszych przypadkach możliwe jest także zaimportowanie modułu lub zdefiniowanych w nim atrybutów pod inną nazwą:

.. code-block:: pycon

    >>> import fibo as fibo2
    >>> from fibo import fib, fib2 as fib22

Lokalizacja modułów - PYTHONPATH
--------------------------------

Kiedy moduł o nazwie *mname* jest importowany, interpreter przeszukuje:

*   bieżący katalog,

*   listę katalogów określoną w zmiennej systemowej *PYTHONPATH*

*   oraz domyślną ścieżkę instalacji (np. ``/usr/local/lib/python``).

*sys.path* jest lista katalogów przeglądanych przez interpreter w trakcie wykonywania instrukcji import.

.. code-block:: pycon

    >>> import sys
    >>> sys.path
     ['',
     'C:\\Python27\\lib\\site-packages\\rested-1.1.0-py2.7.egg',
     'C:\\Program Files (x86)\\DreamPie\\share\\dreampie',
     'C:\\Python27\\python27.zip',
     'C:\\Python27\\DLLs',
    ...]

W szczególności, lista ta może zostać zmodyfikowana, jeżeli chcemy ładować moduły z określonego katalogu.

Przestrzeń nazw modułu
----------------------

Pliki modułów podczas operacji importowania stają się obiektami.
Obiekty te zawierają zmienne istniejące w module:

.. code-block:: pycon

    >>> import fibo
    >>> print([x for x in fibo.__dict__ if not x.startswith("__")])
    ['fib', 'fib2']

Podczas importu wykonywane są wszystkie instrukcje zawarte w module.
Przypisania na najwyższym poziomie modułu tworzą atrybuty obiektu modułu.
Dostęp do przestrzeni nazw modułu odbywa się za pomocą *__dict__* lub *dir(M)*

Modyfikacja modułów
-------------------

Operacja importowania modułu odbywa się tylko raz, podczas pierwszego użycia instrukcji *import* lub *from*.
Ponowne użycie instrukcji *import* lub *from* do zaimportowania raz załadowanego modułu nie spowoduje ponownego wykonania kodu tego modułu, nawet jeżeli instrukcje importu znajdują się w innym pliku.
Aby jeszcze raz załadować (wykonać) moduł, należy użyć funkcji ``importlib.reload``.
W Pythonie 2 jest to wbudowana funkcja, tzn. nie trzeba jej importować.

.. code-block:: pycon

    >>> import fibo
    >>> fibo.fib(1000)

    >>> from importlib import reload
    >>> imp.reload(fibo)

Funkcja ``reload`` jest przydatna przy jednoczesnej pracy w pliku i interaktywnej konsoli.
Po zmodyfikowaniu pliku, możemy załadować jego nowszą wersję wywołując ``reload``.

Ukrywanie danych w modułach
---------------------------

Przy wykonywaniu ``from module import *`` importowane są tylko atrybuty, które nie zaczynają się od podkreślnika.
Tym samym funkcje i klasy zaczynające się od podkreślnika są niejako "chronione" i nie udostępniane innym modułom, chyba że zostaną jawnie zaimportowane (``from module import _funkcja``).

Alternatywnie, w module można zdefiniować specjalny atrybut ``__all__``, który określa listę nazw wszystkich obiektów, które są "publiczne", tzn. importowane przy wykonaniu instrukcji ``from module import *``:

.. code-block:: python

    __all__ = ['fib', 'fib2']

Pakiety
=======

Wprowadzenie
------------

W celu zorganizowania plików modułów w logiczną całość możemy zorganizować je w strukturę pakietów modułów.
Jest ona oparta na hierarchii katalogów (folderów) systemu operacyjnego.

.. code-block:: pycon

    >>> import mymodules.fibo

Kropka oznacza, że w podkatalogu *mymodules* znajdziemy moduł *fibo*.
Katalog *mymodules* musi być umieszczony w jednym z katalogów znajdujących się na ścieżce wyszukiwania modułów (np. w zmiennej środowiskowej *PYTHONPATH*).

Aby katalogi były przeszukiwane podczas importu, muszą zawierać plik o nazwie *__init__.py*.

.. code-block:: pycon

    mycode\
        mymodules\
            __init__.py
            fibo.py
            fobo.py

W powyższym przykładzie, ``mycode`` **nie** jest pakietem.

Plik *__init__.py* może zawierać instrukcje, które zostaną automatycznie wykonane podczas importu pakietu.

Importowanie pakietów
---------------------

Aby wykonać funkcję *foo* z modułu *fibo*, musimy ją zaimportować:

.. code-block:: pycon

    >>> import mymodules.fibo
    >>> mymodules.fibo.foo(100)

lub:

.. code-block:: pycon

    >>> import mymodules.fibo as fi
    >>> fi.foo(100)

lub:

.. code-block:: pycon

    >>> from mymodules.fibo import foo
    >>> foo(100)

Wewnątrz pliku ``fobo.py`` możemy zaimportować moduł ``fibo`` używając względnej ścieżki:

.. code-block:: pycon

    >>> from . import fibo
    >>> fibo.foo(100)

lub:

    >>> from .fibo import foo
    >>> foo(100)
