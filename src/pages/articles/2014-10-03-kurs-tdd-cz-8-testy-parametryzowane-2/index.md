---
title: Kurs TDD cz. 8 — Testy parametryzowane
redirect_from: 
  - "/2014/10/03/kurs-tdd-cz-8-testy-parametryzowane-2/"
date: "2014-10-03T19:44:17.000Z"
layout: post
draft: false
path: "/posts/kurs-tdd-8-testy-parametryzowane"
category: "Programowanie"
tags:
  - "TDD"
  - "Kurs TDD"
  - "C#"
  - "NUnit"
description: "Celem testów parametryzowanych jest przekazanie zestawu wartości danych do metody testowej. Każdy zestaw danych jest traktowany jako osobny test."
---

Temat testów parametryzowanych pojawił się przy okazji pisania [naszego "drugiego" testu jednostkowego](/posts/kurs-tdd-5-nasz-drugi-test-jednostkowy "Kurs TDD cz. 5: Nasz drugi test jednostkowy"). Wtedy to skorzystaliśmy z atrybutu `[TestCase]` pozwalającym na zdefiniowanie zestawów wartości, które są przekazywane do metody testowej. Stosując atrybuty do testów parametryzowanych (m. in. `TestCase`, `Values`, `Range`, `Random`) ograniczamy liczbę naszych metod testowych i redukujemy ilość redundantnego kodu. Ponadto, zyskujemy na czytelności naszych testów.

Kod źródłowy do kursu TDD pojawił się na GitHubie i jest dostępny pod adresem: [https://github.com/dariusz-wozniak/TddCourse](https://github.com/dariusz-wozniak/TddCourse "https://github.com/dariusz-wozniak/TddCourse"). Zapraszam do ściągania i dalszej lektury...

# Testy parametryzowane

Celem testów parametryzowanych jest przekazanie zestawu wartości danych do metody testowej. Każdy zestaw danych jest traktowany jako osobny test. W NUnit ideę parametryzacji testów realizuje się przy pomocy atrybutów:

*   `[TestCase]` (zestaw wartości dla wszystkich parametrów metody)
*   `[Values]` (zestaw wartości dla parametru metody)
*   `[Range]` (zakres od-do z zadanym krokiem dla parametru metody)
*   `[Random]` (zakres od-do oraz ilość wartości do wygenerowania dla parametru metody)

Oraz:

*   `[TestCaseSource]` (zestaw wartości odczytany ze zmiennej dla wszystkich parametrów metody)
*   `[ValueSource]` (zestaw wartości odczytany ze zmiennej dla parametru metody)

Zobrazujmy te przypadki kodem źródłowym na przykładzie funkcji `Divide` pochodzącej z [naszego kalkulatora, o którym była mowa w części piątej kursu](/posts/kurs-tdd-5-nasz-drugi-test-jednostkowy). Niektóre z testów (przykładowo te z atrybutem `Random`) mają jedynie na celu pokazanie w jaki sposób realizowana jest idea parametryzacji testów. Nie zawsze istnieje sens użycia danej techniki w przypadku metody Divide, dlatego też skupimy się tutaj nie na ich zasadności, a na tym jaki uzyskujemy rezultat.

### `TestCase`

Przepis jest następujący – Atrybutowi metody `TestCase` przypisujemy kolejno wartości, które są odwzorowaniem parametrów naszego testu.

Notka: NUnit konwertuje typy za pomocą metody [`Convert.ChangeType`](http://msdn.microsoft.com/en-us/library/system.convert.changetype%28v=vs.110%29.aspx).

```csharp
[TestCase(4, 2, 2.0f)]
[TestCase(-4, 2, -2.0f)]
[TestCase(4, -2, -2.0f)]
[TestCase(0, 3, 0.0f)]
[TestCase(5, 2, 2.5f)]
[TestCase(1, 3, 0.333333343f)]
public void Divide_ReturnsProperValue(int dividend, int divisor, float expectedQuotient)
{
    var calc = new Calculator();
    var quotient = calc.Divide(dividend, divisor);
    Assert.AreEqual(expectedQuotient, quotient);
}

```
Spójrzmy na sygnaturę metody oraz jeden z zestawu testów: 

```csharp
[TestCase(-4, 2, -2.0f)]
public void Divide_ReturnsProperValue(int dividend, int divisor, float expectedQuotient)
```

Wartości z atrybutu `TestCase` przekazywane są kolejno do parametrów metody:

*   dividend (dzielna) = -4
*   divisor (dzielnik) = 2
*   expectedQuotient (oczekiwany iloraz) = -2

W ostatnich dwóch liniach testu sprawdzamy czy nasz kalkulator radzi sobie z zadanym zestawem danych: 

```csharp
var quotient = calc.Divide(dividend, divisor);
Assert.AreEqual(expectedQuotient, quotient);
```

### `Values`

W atrybucie Values przekazujemy konkretne wartości do parametru metody. Przykład: 
```csharp
[Test]
public void Divide_DividendIsZero_ReturnsQuotientEqualToZero(
    [Values(-2, -1, 1, 2)] double divisor)
{
    var calc = new Calculator();
    float quotient = calc.Divide(0, divisor);
 
    Assert.AreEqual(0, quotient);
}
```
 W powyższym przykładzie chcemy dowieść, że gdy dzielna jest równa zeru, to niezależnie od wartości dzielnika, iloraz ma być równy również zeru. Wartości dzielnika: {-2, -1, 1, 2}. W tym teście mamy więc cztery zestawy danych, wszystkie zdefiniowane w atrybucie `Values`.

### `Range`

W atrybucie `Range` przekazujemy wartości graniczne zakresu oraz krok (domyślnie = 1). Przykład: 

```csharp
[Test]
public void Divide_DividendIsZero_ReturnsQuotientEqualToZero(
  [Range(from: 1, to: 5, step: 1)] int divisor)
//...
```

 Powyższy kod wygeneruje zestaw wartości od 1 do 5 z krokiem 1: {1, 2, 3, 4, 5}.

### `Random`

W atrybucie `Random` podajemy wartości graniczne zakresu oraz liczbę wartości, które mają się wygenerować losowo. Przykład: 

```csharp
[Test]
public void Divide_DividendAndDivisorAreRandomPositiveNumbers_ReturnsPositiveQuotient(
    [Random(min: 1, max: 100, count: 10)] double dividend,
    [Random(min: 1, max: 100, count: 10)] double divisor)
{
    var calc = new Calculator();
    float quotient = calc.Divide(dividend, divisor);
    Assert.That(quotient > 0);
}
```

 W tym przykładzie generujemy po 10 wartości losowych dla każdego parametru z zakresu od 1 do 100. Tym razem użyliśmy atrybutu dla dwóch parametrów, w związku z czym ilość zestawów testowych jest równa 10 × 10 = 100.

# Podsumowanie

*   Testy parametryzowane znacznie ułatwiają testowanie kodu w przypadkach, kiedy mamy zdefiniowane zestawy wartości. Dzięki temu uzyskamy wiele testów bez niepotrzebnej duplikacji kodu oraz poprawimy ich czytelność.
*   Zestaw wartości może być dostarczany przez nie-programistę, np. przez klienta, analityka biznesowego lub testera.

# Trzy grosze

*   NUnit posiada także atrybuty `[TestCaseSource]` oraz `[ValueSource]`, które nie zostały ukazane wyżej, ale są bajecznie proste. Argumenty te odczytują wartości z podanej zmiennej — W argumencie atrybutu podajemy nazwę zmiennej, z której mają być odczytane wartości. Ma to następujące korzyści:
    *   Jeden zestaw danych możemy przekazywać do wielu metod testowych.
    *   Dane mogą być odczytane z zewnętrznego źródła (np. plik, baza danych). Pamiętajmy tylko, że test jednostkowy powinien wykonywać się w pamięci, a więc czytanie danych z zewnętrznego źródła jest niezgodne z jego definicją. Nic nie stoi jednak na przeszkodzie, aby tę funkcję wykorzystać w innego rodzaju testów, np. w testach integracyjnych lub akceptacyjnych.
*   Pokazane sposoby realizacji testów parametryzowanych dotyczą tylko jednego frameworka — NUnit. W innych frameworkach możemy tych koncepcji nie znaleźć lub występują one pod inną nazwą.
*   Nie należy używać atrybutu `Random` w kontekście testu jednostkowego. Jest to niezgodne z zasadą mówiącą, że test jednostkowy powinien być powtarzalny. W przypadku zestawu danych który za każdym razem jest generowany losowo, takiej powtarzalności nie ma.

# Zobacz też

*   [NUnit - Theory](https://github.com/nunit/docs/wiki/Theory-Attribute)
*   [AutoFixture](https://github.com/AutoFixture)

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
8. Testy parametryzowane
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