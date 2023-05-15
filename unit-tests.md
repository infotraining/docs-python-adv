# Testy jednostkowe

Testowanie programu pozwala sprawdzić, czy program zachowuje się w pożądany sposób.
Manualne testowanie programu jest czasochłonne.
Dlatego dobrą praktyką jest pisanie automatycznych testów, które mogą zostać szybko wykonane przez komputer.
Testuje się nie tylko całe programy, ale też pojedyncze funkcje czy klasy.

Standardowa biblioteka Pythona posiada moduły ``unittest`` i ``doctest`` ułatwiające pisanie takich testów.

Alternatywą dla standardowych bibliotek jest `pytest`. Jest to jedna z najczęściej używanych bibliotek to testów jednostkowych.

## Unittest

### Wprowadzenie

``unittest`` jest jedną z najpopularniejszych bibliotek stosowanych do pisania testów jednostkowych. Wynika to jej następujących cech:

* jest częścią standardowej biblioteki Pythona, co oznacza, że jest dostępna wszędzie, gdzie jest zainstalowany Python.
* jest inspirowana biblioteką ``JUnit`` z Javy. Osoby, które pisały wcześniej testy w ``JUnit`` bardzo szybko nauczą się pisać testy w Pythonie. Ponadto, wykorzystano sprawdzony interfejs.
    Z drugiej strony, wzorowanie się na bibliotece Javy powoduje, że testy są dosyć rozwlekłe i "rozgadane".
* potrafi automatycznie znaleźć wszystkie testy i wykonać je.

### Przykład

Testy grupuje się w klasach dziedziczących po ``unittest.TestCase``.
Każda metoda zaczynająca się od ``test_`` to jeden test.
Wywołanie ``unittest.main()`` powoduje uruchomienie wszystkich testów (nie jest potrzebne ręczne wymienianie nazw wszystkich testów).

Powszechnie przyjętą konwencją jest separacja testów i testowanego kodu, poprzez umieszczenie ich w osobnych plikach.
Dodatkowo, testy umieszcza się w osobnym katalogu, a nie w tym samym katalogu co testowany moduł.
Dzięki temu możliwa jest instalacja pakietu bez instalowania bibliotek wykorzystywanych tylko przez testy.

```python
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
```

Ostatnie dwie linie powyższego kodu gwarantują uruchomienie testów, ale tylko gdy plik z kodem zostanie bezpośrednio uruchomiony, a nie zaimportowany.

```bash
python test_converters.py
```

### Uruchamianie testów

Możemy wykonać testy wpisując w konsoli.

```bash
python -m unittest test_my_module.TestAdd
```

Ogromną zaletą biblioteki ``unittest`` jest możliwość uruchomienia wszystkich testów bez określania, gdzie się one znajdują.
W takiej sytuacji biblioteka ``unittest`` poszukuje testów we wszystkich plikach znajdujących się w aktualnym katalogu, podkatalogach, podkatalogach podkatalogów itd.

```bash
python -m unittest
```

Przydatnym przełącznikiem jest ``--failfast``  (lub ``-f``), który zatrzymuje wykonywanie testów po pierwszym teście, który nie przeszedł.
Ułatwia to skupienie się na naprawieniu testu, ponieważ na wyjściu pojawiają się informacje dotyczące tylko jednego nieprzechodzącego testu.

```bash
python -m unittest --failfast
```

Niektóre środowiska programistyczne, takie jak PyCharm, posiadają wsparcie dla uruchamiania testów.

### ``setUp`` i ``tearDown``

Jeżeli na początku lub na końcu każdego testu wykonujemy operacje, które powtarzają się w innych testach, wtedy można umieścić je w metodach ``setUp`` i ``tearDown``.
``setUp`` jest metodą wykonywaną na początku każdego testu, natomiast ``tearDown`` -- po wykonaniu testu, niezależnie od tego, czy test przeszedł, czy nie.

Jest to świetne miejsce na:

* uzyskanie zasobów, które są potrzebne w każdym teście (np. połączenie z bazą danych),
* skonfigurowanie środowiska w ten sam sposób dla każdego testu (np. w ``setUp`` -- utworzenie przykładowych tabel i rekordów w bazie danych, a w ``tearDown`` -- "posprzątanie" po teście, tzn. wyczyszczenie testowej bazy danych).

```python
class TestAdd(unittest.TestCase):
    def setUp(self):
        print("setUp")

    def tearDown(self):
        print("tearDown")

    def test_one(self):
        print("test one")

    def test_two(self):
        print("test two")
```

```bash
setUp
test one
tearDown
setUp
test two
tearDown
```

Warto zauważyć, że, generalnie rzecz biorąc, w Pythonie nazwy metod piszemy małymi literami, a poszczególne słowa rozdzielamy podkreślnikami, np. ``tear_down``, ``set_up``.
Jednak konwencja ta nie zawsze jest przestrzegana, nawet w obrębie biblioteki standardowej, czego przykładem jest ``unittest``.
Jest ona wzorowana na bibliotece ``JUnit`` napisanej w Javie, gdzie obowiązuje konwencja "camelCase".

### ``self.assert*``

W środku każdego testu możemy wykorzystać szereg metod zaczynających się od ``assert``, np. ``assertEqual``.
Test przechodzi, jeżeli wszystkie takie asercje są prawdziwe.
Jeżeli chociaż jedna taka asercja nie będzie spełniona, wówczas wykonywanie testu jest natychmiast przerywane i wykonywana jest metoda ``tearDown``.

Najczęściej wykorzystywane asercje to:

* ``self.assertTrue(condition)`` i ``self.assertFalse(condition)``,

* ``self.assertEqual(got, expected)`` i ``self.assertNotEqual(got, expected)``,

* ``self.assertIn(element, collection)`` i ``self.assertNotIn(element, collection)``,

* ``self.assertIsInstance(obj, class_)`` i ``self.assertNotIsInstance(obj, class_)``.

Do sprawdzenia, czy dany blok kodu rzuca wyjątek, można użyć ``self.assertRaises(ExceptionType)``:

```python
class UrlConverterTests(unittest.TestCase):
    def test_raises_exception_for_empty_string(self):
        url = ""

        with self.assertRaises(ValueError):
            url_converter(url)
```

Jeżeli sprawdzenie typu rzuconego wyjątku to za mało, możemy uzyskać do niego dostęp:

```python
class UrlConverterTests(unittest.TestCase):
    def test_raises_exception_for_empty_string(self):
        url = ""

        with self.assertRaises(ValueError) as ex:
            url_converter(url)
        self.assertEqual(ex.message, 'Empty url')
```

## Doctest

``doctest`` jest częścią standardowej biblioteki Pythona, ale oferuje zupełnie inne podejście do testowania.
Zamiast umieszczać testy w osobnych plikach, testy można umieścić w docstringu testowanej funkcji, klasy lub metody.
Takie podejście ma kilka przewag nad ``unittest``:

* Ponieważ testy są częścią docstringa, pełnią wówczas jednocześnie rolę dokumentacji.
    Jeżeli są to krótkie testy, jest to wówczas bardzo dobra dokumentacja.

* Testy są trzymane blisko obiektu, który jest testowany, co jest generalnie pożądane, ponieważ nie ma potrzeby przeskakiwania między plikami.

Niestety, to podejście ma też pewne istotne wady.
Przede wszystkim, takie podejście nie skaluje się wraz z coraz bardziej skomplikowanymi testami, co oznacza, że sprawdza się ono głównie w przypadku krótkich testów dla prostych funkcji.

```python
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
```

## Pytest

### Wprowadzenie

Zasadniczą wadą biblioteki ``unittest`` jest rozwlekłość pisanych w nich testów.
Każdy test musi być metodą umieszczoną w klasie. Nazwy metod takie jak ``self.assertEqual`` są dosyć rozwlekłe.

Czy można pisać krótsze, czytelniejsze testy?

Odpowiedź brzmi tak, wystarczy doinstalować bibliotekę ``pytest``.

```bash
pip install pytest
```

``pytest`` pozwala umieścić testy w funkcjach, których nazwa zaczyna się od ``test_``.
Ponadto, można użyć asercji zamiast metod takich jak ``self.assertEqual``.

### Proste testy

W bibliotece każda funkcja zaczynająca się od `test_*` jest traktowana jak test. 

Asercje są przeprowadzane za pomocą standardowej instrukcji `assert`.

```python
# my_module.py
def increment(x):
    if x == 0:
        raise ValueError('Zero is not valid value.')
    return x + 1

# test_my_module.py
import pytest

from my_module import inc

def test_increment_returns_next_value():
    # Poniższa asercja jest znacznie czytelniejsza niż self.assertEqual(inc(3), 4)
    assert increment(3) == 4
```

Asercja wyjątków jest przeprowadzana za pomocą menadżera kontekstu `pytest.raises`:

```python
def test_increment_raises_when_invalid_argument():
    with pytest.raises(ValueError):
        increment(0)
```

### Grupowanie testów w klasach

Testy mogą być grupowane w klasach. Klasa grupująca powinna zaczynać się od `Test*`, w przeciwnym razie testy są pomijane. Nie jest wymagane dziedziczenie po klasie bazowej, tak jak ma to miejsce w bibliotece `unittest`.

```python
class TestMultiple:
    def test_first(self):
        assert 5 == 5

    def test_second(self):
        assert 10 == 10
```

### Uruchamianie testów

``pytest``, podobnie jak ``unittest`` potrafi sam znaleźć wszystkie testy w aktualnym katalogu i podkatalogach.

```bash
$ pytest -v
```

```bash
============ test session starts ============
platform linux -- Python 3.7.3, pytest-5.4.3, py-1.8.1, pluggy-0.13.1
cachedir: .pytest_cache
rootdir: /chatapp
collected 3 items 

test_simple.py::test_something FAILED [ 33%]
test_simple.py::TestMultiple::test_first PASSED [ 66%]
test_simple.py::TestMultiple::test_second PASSED [100%]
```

Przydatną opcją jest `-k EXPRESSION`, która powoduje uruchamienie tylko testów, których nazwa pasuje do podanego wyrażenia tekstowego (case-insensitive):

```bash
$ pytest -v -k first

$ pytest -v -k "not something"
```

### Fikstury

Dostarczanie obiektów, skonfigurowanych na potrzeby testu jest realizowane w `pytest` za pomocą funkcji z dekoratorem `@pytest.fixture`. Taka funkcja może być później użyta w teście jako parametr. Powoduje to wstrzyknięcie do testu odpowiednio skonfigurowanego obiektu.

```py
@pytest.fixture()
def warehouse():
    warehouse = InMemoryWarehouse()
    warehouse.add("ProductA", 50)
    return warehouse

def test_order_is_filled_if_enough_items_in_warehouse(warehouse):
    order_service = OrderService(warehouse)
    order = order_service.process_order("ProductA", 50)
    assert order.is_filled()
```

Fikstury mogą też przyjmować jako parametry inne fikstury:

```py
# Arrange
@pytest.fixture
def first_entry():
    return "a"

# Arrange
@pytest.fixture
def order(first_entry):
    return [first_entry]
```

```py
def test_string(order):
    # Act
    order.append("b")

    # Assert
    assert order == ["a", "b"]
```

Możemy użyć dowolną liczbę fiktur w teście (lub innej fiksturze):

```py
@pytest.fixture
def first_entry():
    return "a"

@pytest.fixture
def second_entry():
    return 2

@pytest.fixture
def order(first_entry, second_entry):
    return [first_entry, second_entry]

@pytest.fixture
def expected_list():
    return ["a", 2, 3.0]

def test_string(order, expected_list):
    order.append(3.0)
    assert order == expected_list
```

Fikstura może być też implementowana jako "fabryka" (może być to potrzebne, gdy wynik fikstury jest potrzebny wiele razy w teście):

```py
@pytest.fixture
def make_customer_record():
    def _make_customer_record(name):
        return {"name": name, "orders": []}

    return _make_customer_record


def test_customer_records(make_customer_record):
    customer_1 = make_customer_record("Lisa")
    customer_2 = make_customer_record("Mike")
    customer_3 = make_customer_record("Meredith")
```

#### Fikstury - setup & teardown

Implementacja wzorca setup & teardown w `pytest` wykorzystuje generator:

```py
@pytest.fixture
def new_user(user_service):
    # Setup
    user = user_service.create_user()
    
    yield user
    
    # Teardown
    user_service.delete(user)


def test_sending_email_to_new_user(new_user):
    email = Email(subject="Greetings!", body="Welcome")
    response = new_user.send_email(email)
    assert response.status == 'OK'
```

#### Wbudowane fikstury

* `tmp_path` - dostarcza tymczasową ścieżkę do plików, która z każdym uruchomieniem testu jest inna

```py
def test_tmp(tmp_path):
    f = tmp_path / "file.txt"
    print("FILE: ", f)

    f.write_text("Hello World")

    fread = tmp_path / "file.txt"
    assert fread.read_text() == "Hello World"
```

```bash
test_tmppath.py::test_tmp 
FILE: /tmp/pytest-of-amol/pytest-3/test_tmp0/file.txt
PASSED
```

* `capsys` - pozwala odczytać tekst wypisywany w konsoli (`sys.stdout` i `sys.stderr`) 

```py
def myapp():
    print("MyApp Started")

def test_capsys(capsys):
    myapp()

    out, err = capsys.readouterr()
    
    assert out == "MyApp Started\n"
```

### Parametryzacja testów

Dekorator `@mark.paramterize` umożliwia łatwą parametryzacje testów:

```py
@pytest.mark.parametrize("test_input,expected", [("3+5", 8), ("2+4", 6), ("6*9", 42)])
def test_eval(test_input, expected):
    assert eval(test_input) == expected
```

Parametryzacja może być również stosowana dla całej klasy testów:

```py
@pytest.mark.parametrize("n,expected", [(1, 2), (3, 4)])
class TestClass:
    def test_simple_case(self, n, expected):
        assert n + 1 == expected

    def test_weird_simple_case(self, n, expected):
        assert (n * 1) + 1 == expected
```