---
title: Kurs TDD cz. 19 — Mock, stub, fake, spy, dummy
date: "2016-03-26T19:15:03.000Z"
layout: post
draft: false
path: "/posts/kurs-tdd-19-mock-stub-fake-spy-dummy"
category: "Programowanie"
tags:
  - "TDD"
  - "Kurs TDD"
description: "Nomenklatura w świecie TDD, a w szczególności ta dotycząca tworzenia atrap, jest źródłem wielu niejasności. Powodem takiego stanu jest fakt, że definicje różnią się w zależności od źródła, tj. książki, lub frameworka. W tym wpisie poznamy charakterystykę takich obiektów testowych jak mock, stub, fake, spy i dummy."
---

Nomenklatura w świecie TDD, a w szczególności ta dotycząca tworzenia atrap, jest źródłem wielu niejasności. Powodem takiego stanu jest fakt, że definicje różnią się w zależności od źródła, tj. książki, lub frameworka. W poprzednich częściach poznaliśmy trzy najbardziej popularne frameworki do tworzenia atrap dla .NET, dla których:

*   Nie ma podziału na rodzaje atrap.
*   Atrapa (pod różną nazwą – _mock_, _fake_, _substitute_), poza subtelnymi różnicami, ma tę samą definicję i służy do jednakowych celów.

![podzial definicji_2](9e324d00-6f32-494b-a5d6-b91ec2253f49.png)

Sprawa komplikuje się gdy mamy do czynienia z literaturą (książki i blogi) dotyczącą testów jednostkowych. Tutaj jest o tyle trudniej ponieważ:

*   Istnieje podział atrap ze względu na cel i zachowanie. Najbardziej popularnym podziałem jest (w moim subiektywnym odczuciu) podział wprowadzony przez Gerarda Meszarosa w książce _xUnit Test Patterns_ na _mock, stub, fake, test spy, dummy_:

![podzial definicji](f0f46491-262f-43d2-b9c5-e8bacb8a0f92.png)

*   Definicje atrap są różne, a niekiedy nawet wykluczające się:

![Terminology Cross-Reference](2347a2e5-288c-4a91-b0fb-cea0dc1f503b.png)

W tym artykule przedstawię atrapy z _xUnit Test Patterns_ napisane w Moq. 

# _Dummy_

_Dummy_ (z ang. imitacja, marionetka) jest najprostszą z atrap, gdyż... nie robi absolutnie nic! Jego zadaniem jest tylko i wyłącznie spełnienie założeń sygnatury.

Dla przykładu, przyjrzyjmy się klasie `Customer`:

```csharp
public class Customer
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
 
    public Customer(string firstName, string lastName)
    {
        if (firstName == null) throw new ArgumentNullException(nameof(firstName));
        if (lastName == null) throw new ArgumentNullException(nameof(lastName));
 
        FirstName = firstName;
        LastName = lastName;
    }
}
```

 Chcąc przetestować pierwszego _null-guard_, musimy przekazać wartość null jako `firstName`. Nie przejmujemy się czym jest `lastName`, gdyż kod związany z tą zmienną nie będzie wykonywany.
 
 Przykład testu: 
 
 ```csharp
[Test]
public void DummyExample()
{
    string firstName = null;
    string lastName = It.IsAny<string>(); // Dummy
 
    Action act = () => new Customer(firstName, lastName);
 
    act.ShouldThrow<ArgumentNullException>();
}
```

 Sygnatura metody została spełniona, nasz dummy object nie robi absolutnie nic, poza tym że udaje jakikolwiek obiekt typu string. Alternatywnym sposobem tworzenia obiektów dummy jest wykorzystanie atrap z zachowaniem `Strict`. Oznacza to, że stworzona atrapa wyrzuci wyjątek przy wywołaniu którejś z jej składowych. Dla przypomnienia, zachowanie atrap dla Moq opisane zostało [w części 15. kursu: "Wstęp do Moq"](/posts/kurs-tdd-15-wstep-do-moq). Równie dobrze, zamiast obiektu _dummy_, możemy też przesłać null lub `default(T)`.

# _Stub_

_Stub_ (z ang. zalążek) to nieco bardziej zaawansowany _dummy_. Dodatkowo jednak, _stub_ potrafi zwracać zdefiniowane przez nas wartości, o ile o nie poprosimy. _Stub_ też nie wyrzuci błędu, jeśli nie zdefiniowaliśmy danego stanu (np. metody void są puste, a niezdefiniowane wartości wyjścia zwracają wartości domyślne).

Przykładowy test:

```csharp
[Test]
public void StubExample()
{
    var customerValidator = new CustomerValidator();
    var customer = Mock.Of<ICustomer>(c => c.GetAge() == 21); // Stub
 
    bool validate = customerValidator.Validate(customer);
 
    validate.Should().BeTrue();
}
```

# _Fake_

_Fake_ (z ang. podróbka, falsyfikat) jest z kolei wariancją _stuba_ i ma na celu symulowanie bardziej złożonych interakcji. Jeśli atrapa posiada złożone interakcje, której symulacja przy pomocy _stuba_ jest niewykonalna (ograniczenia frameworka) lub bardzo złożona, możemy wykorzystać _fake_'a. Jest to z reguły własnoręcznie napisana klasa, która posiada minimalną funkcjonalność aby spełnić założenia interakcji.

Przykład: Rozważmy klasę, która generuje raporty z interfejsu `ICustomerRepository`.

```csharp
public class CustomerReportingService
{
    private readonly ICustomerRepository _customerRepository;
 
    public CustomerReportingService(ICustomerRepository customerRepository)
    {
        _customerRepository = customerRepository;
    }
 
    public string GenerateReport()
    {
        return string.Join("\n", _customerRepository.AllCustomers.Select(x =>
          x.FirstName + " " + x.LastName));
    }
```

 Gdzie `ICustomerRepository` to:
 
 ```csharp 
public interface ICustomerRepository
{
    IReadOnlyList<ICustomer> AllCustomers { get; }
    bool Add(ICustomer customer);
}
```
 Aby przetestować metodę `GenerateReport` klasy `CustomerReportingService` potrzebny nam jest dostęp do kolekcji `AllCustomers` interfejsu `ICustomerRepository`. Jako że kolekcja ta jest niezmienna (immutable), stworzenie _stuba_ dla interfejsu `ICustomerRepository` wiązałoby się z dość skomplikowanym kodem (a w przypadku niektórych bibliotek – stworzenie takiego _stuba_ jest niemożliwe). Można zatem stworzyć klasę (w projekcie testowym!), która będzie wykonywać minimum do wykonania testu. Takim minimum jest w tym przypadku dodanie elementu ICustomer do kolekcji. Ostatecznie, nasz _fake_ wygląda następująco:
 
 ```csharp
internal class FakeCustomerRepository : ICustomerRepository
{
    private readonly List<ICustomer> _customers = new List<ICustomer>();
 
    public IReadOnlyList<ICustomer> AllCustomers => _customers.AsReadOnly();
 
    public bool Add(ICustomer customer)
    {
        _customers.Add(customer);
        return true;
    }
}
```

 Natomiast test wygląda następująco:
 
 ```csharp
[Test]
public void FakeExample()
{
    var customerRepository = new FakeCustomerRepository(); // Fake

    customerRepository.Add(Mock.Of<ICustomer>(c =>
      c.FirstName == "John" && c.LastName == "Kowalski"));

    customerRepository.Add(Mock.Of<ICustomer>(c =>
      c.FirstName == "Steve" && c.LastName == "Jablonsky"));
 
    var customerReportingService = new CustomerReportingService(customerRepository);
 
    string report = customerReportingService.GenerateReport();
 
    report.Should().Be("John Kowalski\nSteve Jablonsky");
}
```

 Podstawowe pytanie jakie może się nasunąć to...: Czemu by nie użyć obiektu klasy CustomerRepository, która jest już zaimplementowana:
 
 ```csharp
public class CustomerRepository : ICustomerRepository
{
    private readonly List<ICustomer> _allCustomers;
    private readonly ICustomerValidator _customerValidator;
 
    public IReadOnlyList<ICustomer> AllCustomers => _allCustomers.AsReadOnly();
 
    public CustomerRepository(ICustomerValidator customerValidator)
    {
        _allCustomers = new List<ICustomer>();
 
        _customerValidator = customerValidator;
    }
 
    public bool Add(ICustomer customer)
    {
        if (_customerValidator.Validate(customer))
        {
            _allCustomers.Add(customer);
            return true;
        }
 
        return false;
    }
}
```
 _Fake_ ma tutaj istotną przewagę nad implementacją produkcyjną; przede wszystkim wykonuje minimum funkcjonalności do spełnienia założeń testu. Oznacza to, że nie potrzebujemy redundantnych interakcji, takich jak np. z `ICustomerValidator` oraz walidacja przy dodawaniu. Uważny czytelnik zauważył też, że istnieje niebezpieczeństwo związane z pisaniem _fake_'ów. Co w przypadku jeśli logika biznesowa _fake_'a będzie różna od implementacji w taki sposób, że test jednostkowy nie wykryje błędu? Trzeba pamiętać o dwóch kwestiach:

*   Testy jednostkowe wykonywane są w izolacji od innych klas. Błędy interakcji pomiędzy klasami powinny być wykryte przez inny rodzaj testów, np. integracyjny lub akceptacyjny.
*   _Fake_'i pisze się w tych szczególnych przypadkach, gdzie ciężko lub niemożliwym jest stworzenie stuba oraz implementacja danego zachowania jest przejrzysta i prosta. Takim przykładem jest kolekcja i dodawanie do niej elementów.

Alternatywnie, _fake_'i można definiować przy użyciu funkcjonalności `Callback`, którą posiada większość frameworków do tworzenia atrap.

# _Mock_

_Mock_ (z ang. imitacja, atrapa) potrafi weryfikować zachowanie obiektu testowanego. Jego celem jest sprawdzenie czy dana składowa została wykonana.

Przykład:

```csharp
[Test]
public void MockExample()
{
    var customerValidator = new CustomerValidator();
    var customer = Mock.Of<ICustomer>(c => c.GetAge() == 21); // Mock
 
    customerValidator.Validate(customer);
 
    Mock.Get(customer).Verify(x => x.GetAge()); // Verification of Mock
}
```

# _Spy_

_Spy_ (z ang. szpieg) to _mock_ z dodatkową funkcjonalnoscią. O ile _mock_ rejestrował czy dana składowa została wywołana, to _spy_ sprawdza dodatkowo ilość wywołań.

Przykład:

```csharp
[Test]
public void SpyExample()
{
    var customerValidator = new CustomerValidator();
    var customer = Mock.Of<ICustomer>(c => c.GetAge() == 21); // Spy
 
    customerValidator.Validate(customer);
 
    Mock.Get(customer).Verify(x => x.GetAge(), Times.Once); // Verification of Spy
}
```

# W skrócie

*   **_Dummy_**
    *   Jego celem jest "zaspokojenie sygnatury".
    *   Nie są wykonywane na nim żadne interakcje.
*   **_Stub_**
    *   Minimalna implementacja do interakcji między obiektami.
    *   Metody void są puste.
    *   Wartości zwracane przez składowe są domyślne dla danego typu lub zdefiniowane ("hard-coded").
*   **_Fake_**
    *   Zawiera implementację logiki, która imituje kod produkcyjny, ale w możliwie najprostszy sposób.
*   **_Mock_**
    *   Weryfikuje zachowanie poprzez rejestrowanie czy dana interakcja została wykonana.
*   **_Spy_**
    *   Weryfikuje zachowanie poprzez rejestrowanie ilości wykonanych interakcji.

W Moq nie ma rozróżnienia na poszczególne typy, wszystko jest mockiem; aby zasymulować działanie poszczególnych rodzajów atrap możemy wykorzystać poszczególne elementy frameworka (lub języka C#):

*   **_Dummy_**
    *   `It.IsAny<T>()`
    *   `MockBehavior.Strict`
    *   alternatywnie: null, `default(T)`
*   **_Stub_**
    *   `Setup()`
    *   `MockBehavior.Loose`
*   **_Mock_**
    *   `Verify()`
*   **_Spy_**
    *   `Verify()` + `Times`
*   **_Fake_**
    *   ręcznie stworzna klasa Fake
    *   `Callback()`

# Podsumowanie

Pytanie jakie może się nasunąć, to —

Czy pomimo, że...:

*   Definicje atrap w zależności od źródła są różne, a niekiedy wykluczające się, oraz
*   W niektórych frameworkach nie ma rozróżnienia na rodzaje atrap, to...

...czy warto pamiętać podział i definicje atrap?

Odpowiedź zależna jest od kontekstu. Moim zdaniem:

*   Warto pamiętać o podstawach teoretycznych testów jednostkowych i atrap z _xUnit Test Patterns_. Jest to wiedza historyczna, ale jednocześnie pozwala na zrozumienie co dokładnie chcemy testować, zwłaszcza w kontekście weryfikacji stanu vs. zachowania.
*   Niektóre frameworki posiadają rozróżnienie na rodzaje atrap (np. Rhino Mocks).
*   Umiejętność pisania ręcznych _fake_'ów może okazać się bardzo przydatna. Pomimo, że z punktu widzenia technicznego _fake_'i można napisać przy użyciu frameworka (choć nie każdego i nie zawsze), to czytelność i zarządzanie jest niekiedy lepsza na korzyść ręcznie napisanej klasy.
*   Rodzaje atrap i ich różnice pozwalają na zrozumienie funkcjonalności frameworków w których takiego nie ma rozróżnienia.

Przypominam, że kod źródłowy całego kursu TDD, jak i tego rozdziału jest dostępny na GitHubie: [https://github.com/dariusz-wozniak/TddCourse](https://github.com/dariusz-wozniak/TddCourse).

# Źródła

*   [xUnit Patterns: Test Double](http://xunitpatterns.com/Test%20Double.html)
*   [xUnit Patterns: Mocks, Fakes, Stubs and Dummies](http://xunitpatterns.com/Mocks,%20Fakes,%20Stubs%20and%20Dummies.html)
*   [Niraj Bhatt: Dummy vs Stub vs Spy vs Fake vs Mock](https://nirajrules.wordpress.com/2011/08/27/dummy-vs-stub-vs-spy-vs-fake-vs-mock/)
*   [Dirk's Daily Digits: Exploring the Continuum of Test Doubles](http://www.vanderbist.com/index.php/2009/03/23/exploring-the-continuum-of-test-doubles/)
*   [Wikipedia \[en\]: Mock object](https://en.wikipedia.org/wiki/Mock_object)
*   [Martin Fowler: Mocks Aren't Stubs](http://martinfowler.com/articles/mocksArentStubs.html)

# P.S. All-in-one

Wszystkie typy w jednym miejscu:

```csharp
[TestFixture]
public class TestDoubles
{
    [Test]
    public void DummyExample()
    {
        string firstName = null;
        string lastName = It.IsAny<string>(); // Dummy
 
        Action act = () => new Customer(firstName, lastName);
 
        act.ShouldThrow<ArgumentNullException>();
    }
 
    [Test]
    public void StubExample()
    {
        var customerValidator = new CustomerValidator();
        var customer = Mock.Of<ICustomer>(c => c.GetAge() == 21); // Stub
 
        bool validate = customerValidator.Validate(customer);
 
        validate.Should().BeTrue();
    }
 
    [Test]
    public void MockExample()
    {
        var customerValidator = new CustomerValidator();
        var customer = Mock.Of<ICustomer>(c => c.GetAge() == 21); // Mock
 
        customerValidator.Validate(customer);
 
        Mock.Get(customer).Verify(x => x.GetAge()); // Verification of Mock
    }
 
    [Test]
    public void SpyExample()
    {
        var customerValidator = new CustomerValidator();
        var customer = Mock.Of<ICustomer>(c => c.GetAge() == 21); // Spy
 
        customerValidator.Validate(customer);
 
        Mock.Get(customer).Verify(x => x.GetAge(), Times.Once); // Verification of Spy
    }
 
    [Test]
    public void FakeExample()
    {
        var customerRepository = new FakeCustomerRepository(); // Fake

        customerRepository.Add(Mock.Of<ICustomer>(c =>
          c.FirstName == "John" && c.LastName == "Kowalski"));

        customerRepository.Add(Mock.Of<ICustomer>(c =>
          c.FirstName == "Steve" && c.LastName == "Jablonsky"));
 
        var customerReportingService = new CustomerReportingService(customerRepository);
 
        string report = customerReportingService.GenerateReport();
 
        report.Should().Be("John Kowalski\nSteve Jablonsky");
    }
}
 
internal class FakeCustomerRepository : ICustomerRepository
{
    private readonly List<ICustomer> _customers = new List<ICustomer>();
 
    public IReadOnlyList<ICustomer> AllCustomers => _customers.AsReadOnly();
 
    public bool Add(ICustomer customer)
    {
        _customers.Add(customer);
        return true;
    }
}
```

<!-- tdd-course-infobox-start -->
<div class="boxBorder">

<div style="text-align: center; font-size: 40px">Kurs TDD</div>

----

<div class="row">
<div class="column">

Część I: Testy jednostkowe – wstęp

1. [Wstęp do TDD](/posts/kurs-tdd-1-wstep/)
2. [Testy jednostkowe, a integracyjne](/posts/kurs-tdd-2-testy-jednostkowe-a-testy-integracyjne/)
3. [Struktura testu, czyli Arrange-Act-Assert](/posts/kurs-tdd-3-struktura-test-czyli-arrange-act-assert)
4. [Nasz pierwszy test jednostkowy](/posts/kurs-tdd-4-nasz-pierwszy-test-jednostkowy)
5. [Nasz drugi test jednostkowy](/posts/kurs-tdd-5-nasz-drugi-test-jednostkowy)
6. [Dobre i złe praktyki testów jednostkowych](/posts/kurs-tdd-6-dobre-i-zle-praktyki-testow-jednostkowych)
7. [Inicjalizacja i czyszczenie danych (SetUp i TearDown)](/posts/kurs-tdd-7-inicjalizacja-i-czyszczenie-danych-setup-i-teardown/)
8. [Testy parametryzowane](/posts/kurs-tdd-8-testy-parametryzowane)
9. [Testy kombinatoryczne i sekwencyjne](/posts/kurs-tdd-9-testy-kombinatoryczne-i-sekwencyjne)
10. [Teorie](/posts/kurs-tdd-10-teorie)
11. [Testowanie klas generycznych](/posts/kurs-tdd-11-testowanie-klas-generycznych)
12. [Classic vs. Constraint Assert Model](/posts/kurs-tdd-12-classic-vs-constraint-assert-model)
13. [Testowanie wywołań asynchronicznych (async await)](/posts/kurs-tdd-13-testowanie-wywolan-asynchronicznych-async-await)

</div>

<div class="column">

Część II: Atrapy obiektów

14. [Testowanie zależności – atrapy obiektów](/posts/kurs-tdd-14-testowanie-zaleznosci-atrapy-obiektow)
2. [Moq cz. 1 – Wstęp](/posts/kurs-tdd-15-wstep-do-moq)
3. [Moq cz. 2 – Argument Matching, Verify, Callback](/posts/kurs-tdd-16-zaawansowane-techniki-moq-argument-matching-verify-callback)
4. [FakeItEasy](/posts/kurs-tdd-17-fakeiteasy)
5. [NSubstitute](/posts/kurs-tdd-18-nsubstitute)
6. Mock, stub, fake, spy, dummy
7. [Mockowanie DateTime.Now, random, static, itp.](/posts/kurs-tdd-20-mockowanie-datetime-now-random-static-itp)
8. [Rodzaje frameworków do tworzenia atrap](/posts/kurs-tdd-21-rodzaje-frameworkow-do-tworzenia-atrap/)

Część III: Teoria

22. [Pokrycie kodu testami (Code Coverage)](/posts/kurs-tdd-22-pokrycie-kodu-testami-code-coverage/)
1. [Czy to się opłaca?](/posts/kurs-tdd-23-czy-to-sie-oplaca/)
1. [Czy pisać testy jednostkowe do istniejącego kodu (legacy code)?](/posts/kurs-tdd-24-czy-pisac-testy-jednostkowe-do-istniejacego-kodu-legacy-code/)
1. [Otwarte pytania](/posts/kurs-tdd-25-otwarte-pytania/)

</div>
</div>
</div>
<!-- tdd-course-infobox-end -->