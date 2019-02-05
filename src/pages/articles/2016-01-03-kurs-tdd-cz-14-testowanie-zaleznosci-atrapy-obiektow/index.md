---
title: Kurs TDD cz. 14 — Testowanie zależności (atrapy obiektów)
redirect_from:
    - "/2016/01/03/kurs-tdd-cz-14-testowanie-zaleznosci-atrapy-obiektow/"
date: "2016-01-03T18:22:34.000Z"
layout: post
draft: false
path: "/posts/kurs-tdd-14-testowanie-zaleznosci-atrapy-obiektow"
category: "Programowanie"
tags:
  - "TDD"
  - "Kurs TDD"
  - "C#"
description: "W części czternastej kursu Test-Driven Development omówimy technikę testowania zależności za pomocą atrap (ang. mock)."
---

W części czternastej kursu Test-Driven Development omówimy **technikę testowania zależności za pomocą atrap** (jęz. ang. "mock").

Test jednostkowy z definicji testuje zachowanie w izolacji, a więc bez zależności zewnętrznych. Takimi zależnościami są najczęściej inne klasy lub interfejsy, które posiadają zachowanie.

Testy jednostkowe powinny być wykonane w izolacji, gdyż zewnętrzne zależności mogą być:

- czasochłonne,
- niezaimplementowane lub nieukończone (np. interfejs, klasa abstrakcyjna, zalążek metody, pusta metoda),
-  niedeterministyczne lub niepowtarzalne (np. aktualny czas),
-  związane z dużą ilością innych klas (high class coupling),
- związane ze stanami, które są trudne do powtórzenia.

Co jeśli nasz moduł korzysta z zewnętrznych modułów (klas, metod, interfejsów, itp.)? Do tego służy tytułowa atrapa. Atrapa żyje tylko i wyłącznie w świecie testów, a jej celem jest symulacja zachowania prawdziwej zależności w oparciu o dane wyjściowe, które sami zdefiniujemy. 

Dla .NET zostało napisanych sporo dobrych (a przy okazji złych) frameworków mających pomóc tworzenie atrap. W tej części kursu posłużymy się własnoręcznie napisaną atrapą, co pozwoli lepiej zrozumieć istotę problemu. W praktyce rzadko zachodzi konieczność pisania atrap ręcznie. W istocie, jeśli framework nie radzi sobie ze stworzeniem atrapy, to zazwyczaj oznacza to problem z naszym projektem klas czy zależności między nimi. W kolejnych częściach kursu zobaczymy w jaki sposób tworzy się atrapy w najpopularniejszych frameworkach dla .NET.

# Nomenklatura

Zanim przejdziemy do części właściwej, przyjrzyjmy się nieco nomenklaturze wokół testowania zależności. Zacznijmy od tego czym właściwie jest "mock" i jaki jest jego polski odpowiednik. "Mock" oznacza w języku angielskim "atrapę", czyli [wg słownika języka polskiego PWN](http://sjp.pwn.pl/sjp/atrapa;2551130):

> atrapa
> «imitacja jakiegoś przedmiotu»

Sam osobiście preferuję odpowiednik angielski "mock", "mockować" zamiast "atrapa", "tworzyć atrapę", gdyż jest to dla mnie bardziej naturalne. W tym artykule celowo używam określenia atrapa częściej niż mock, gdyż mock posiada w świecie testów jednostkowych wiele, wykluczających się niekiedy, definicji. Definicją mocka, a także innych pojęć (fake, stub, itp.) zajmiemy się w jednej z kolejnej części. W tym artykule mock czy atrapa przyjmuje definicję klasy z zadanymi przez nas oczekiwanymi wartościami wyjściowymi, dzięki której możemy przetestować daną zależność.

# Do dzieła!

Tym razem zamiast kalkulatora, weźmy na tapetę klasę która waliduje pewne warunki biznesowe. Najprostszy z brzegu przykład: klasa walidator do interfejsu `ICustomer`: 

```csharp
public class CustomerValidator
{
    public bool Validate(ICustomer customer)
    {
        throw new NotImplementedException();
    }
}
```

 Interfejs `ICustomer` posiada metodę Age, która zwraca typ liczby całkowitej: 

```csharp
public interface ICustomer
{
    int GetAge();
}
```

 Bazując na powyższym przykładzie, chcemy aby walidator sprawdził czy wiek użytkownika wynosi powyżej 18 lat. Nasza zależność (ang. dependency) to interfejs `ICustomer`, który posiada metodę `GetAge`. Zaraz, ale na jakiej podstawie wyliczany jest wiek klienta...? Hipotetycznie, przy takim designie interfejsu, data urodzenia mogłaby być wstrzykiwana przez konstruktor. Przy czym dla nas nie ma to większego znaczenia, gdyż nie chcemy zależeć w naszym teście od konkretnej implementacji.
 
 Wracając do punktu wyjścia, nasze warunki biznesowe dla walidatora interfejsu `ICustomer` są następujące:

1.  Gdy `ICustomer` jest nullem, wyrzuć wyjątek typu `ArgumentNullException`.
2.  Gdy wiek klienta wynosi poniżej 18, zwróć wartość false.
3.  W przeciwnym razie, zwróć wartość true.

## Kryterium nr 1: Gdy ICustomer jest nullem, wyrzuć wyjątek typu ArgumentNullException.

Dla przypadku numer 1, nie musimy tworzyć atrapy interfejsu `ICustomer`. Zgodnie z zasadami Test-Driven Development, zaczynamy od testu: 

```csharp
[Test]
public void WhenCustomerIsNull_ThenArgumentNullExceptionIsThrown()
{
    var validator = new CustomerValidator();
 
    Action action = () => validator.Validate(null);
 
    action.ShouldThrow<ArgumentNullException>();
}
```

 Uwaga: W powyższym teście skorzystaliśmy z płynnych asercji opisanych w artykule: [„Płynne asercje”, czyli jak ułatwić sobie życie korzystając z Fluent Assertions?](/posts/plynne-asercje-czyli-jak-ulatwic-sobie-zycie-korzystajac-z-fluent-assertions).
 
 Następnie, piszemy implementację spełniającą warunek w teście: 

```csharp
public bool Validate(ICustomer customer)
{
    if (customer == null) throw new ArgumentNullException(nameof(customer));
    throw new NotImplementedException();
}
```

## Kryterium nr 2: Gdy wiek klienta wynosi poniżej 18, zwróć wartość false.

Aby przetestować punkt drugi i trzeci, potrzebujemy atrapy dla wszystkich zależności walidatora. W naszym przypadku, mamy jedną taką zależność, a jest nią interfejs `ICustomer`. Zwykle atrapy tworzymy za pomocą frameworków, jednak aby zobrazować lepiej ich ideę, stworzymy taką atrapę ręcznie: 

```csharp
internal class CustomerMock : ICustomer
{
    private readonly int _expectedAge;
 
    public CustomerMock(int expectedAge)
    {
        _expectedAge = expectedAge;
    }
 
    public int GetAge() => _expectedAge;
}
```

 I wszystko jasne! Stworzyliśmy atrapę, w której z góry możemy zdefiniować nasze dane wyjściowe, a więc w naszym przypadku - wiek. Nasz walidator jest już całkowicie pozbawiony zewnętrznych zależności, gdyż jedyny zależny interfejs posiada już atrapę.
 
 Piszemy więc test dla warunku nr 2: "Gdy wiek klienta wynosi poniżej 18, zwróć wartość false." 

```csharp
[Test]
public void WhenCustomerHasAgeLessThan18_ThenValidationFails()
{
    var validator = new CustomerValidator();
    var customer = new CustomerMock(expectedAge: 16);
 
    bool validate = validator.Validate(customer);
 
    validate.Should().BeFalse();
}
```

 I kod, który spełnia powyższy warunek: 

```csharp
public bool Validate(ICustomer customer)
{
    if (customer == null) throw new ArgumentNullException(nameof(customer));
    return false;
}
```

## Kryterium nr 3: W przeciwnym razie, zwróć wartość true.

Na koniec został nam test dla warunku 3: "W przeciwnym razie, zwróć wartość true." Wykorzystamy atrybut `Values`, gdyż chcemy przetestować logikę dla wartości brzegowej (wiek = 18) i jednej wartości niebrzegowej (na przykład 19). 

```csharp
[Test]
public void WhenCustomerHasAgeGreaterThanOrEqualTo18_ThenValidationPasses(
  [Values(18, 19)] int expectedAge)
{
    var validator = new CustomerValidator();
    var customer = new CustomerMock(expectedAge: expectedAge);
 
    bool validate = validator.Validate(customer);
 
    validate.Should().BeTrue();
}
```

 Oraz finalny kod walidatora: 

```csharp
public class CustomerValidator
{
    private const int MinimumAge = 18;
 
    public bool Validate(ICustomer customer)
    {
        if (customer == null) throw new ArgumentNullException(nameof(customer));
 
        if (customer.GetAge() < MinimumAge) return false;
        return true;
    }
}
```

Voilà: Zakończyliśmy pisać nasz walidator przy użyciu TDD!

# Podsumowanie

Nasz walidator testuje zależność zewnętrzną nie polegając na jej implementacji. Stworzyliśmy ręcznie atrapę pochodną od interfejsu ICustomer, w której możemy zdefiniować dane wyjściowe.

W dalszych rozdziałach poznamy sposoby tworzenia atrap za pomocą już istniejących frameworków. Tymczasem zachęcam do dalszej zabawy z ręcznymi atrapami. A jeśli ktoś ma więcej czasu, to sugeruję napisać mini-framework do generowania takich atrap.

Przypominam, że kod źródłowy całego kursu TDD, jak i tego rozdziału jest dostępny na GitHubie: [https://github.com/dariusz-wozniak/TddCourse](https://github.com/dariusz-wozniak/TddCourse).

# Linki zewnętrzne

*   [List of Automated Testing (TDD/BDD/ATDD/SBE) Tools and Frameworks for .NET](https://github.com/dariusz-wozniak/List-of-Testing-Tools-and-Frameworks-for-.NET)
*   [Wikipedia (PL): Atrapa obiektu](https://pl.wikipedia.org/wiki/Atrapa_obiektu)
*   [Wikipedia (EN): Mock object](https://en.wikipedia.org/wiki/Mock_object)


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

14. Testowanie zależności – atrapy obiektów
2. [Moq cz. 1 – Wstęp](/posts/kurs-tdd-15-wstep-do-moq)
3. [Moq cz. 2 – Argument Matching, Verify, Callback](/posts/kurs-tdd-16-zaawansowane-techniki-moq-argument-matching-verify-callback)
4. [FakeItEasy](/posts/kurs-tdd-17-fakeiteasy)
5. [NSubstitute](/posts/kurs-tdd-18-nsubstitute)
6. [Mock, stub, fake, spy, dummy](/posts/kurs-tdd-19-mock-stub-fake-spy-dummy)
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