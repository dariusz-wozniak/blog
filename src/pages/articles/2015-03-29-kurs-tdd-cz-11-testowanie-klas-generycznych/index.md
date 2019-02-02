---
title: Kurs TDD cz. 11 — Testowanie klas generycznych
date: "2015-03-29T12:48:16.000Z"
layout: post
draft: false
path: "/posts/kurs-tdd-11-testowanie-klas-generycznych"
category: "Programowanie"
tags:
  - "TDD"
  - "Kurs TDD"
  - "C#"
description: "W niniejszym artykule przyjrzymy się w jaki sposób możemy przetestować klasy generyczne za pomocą funkcjonalności NUnita pod nazwą Generic Test Fixture."
---

W niniejszym artykule przyjrzymy się w jaki sposób możemy przetestować klasy generyczne za pomocą funkcjonalności NUnita pod nazwą Generic Test Fixture. Dla przykładu, weźmy sobie metodę dodawania w generycznej klasie kalkulatora: 

```csharp
public class GenericCalculator<T>
{
    public T Add(T a, T b)
    {
        return (dynamic) a + (dynamic) b;
    }
}
```

 W NUnit sprawa z testowaniem klas generycznych ma się prosto, a służy do tego tzw. Generic Test Fixture. Aby przetestować taką klasę, należy w parametrze atrybutu `[TestFixture]` podać generyczny typ (jeden lub więcej), który będzie przekazywany niżej do testów. Jeżeli chcemy stworzyć instancję naszej generycznej klasy, należy skorzystać z [`new()` constraint](https://msdn.microsoft.com/en-us/library/sd2w2ew5%28v=vs.140%29.aspx "new() constraint").
 
 Dla klasy `GenericCalculator`, chcielibyśmy przetestować metodę dodawania dla typów numerycznych – int, float, double i decimal. Test dla takiej kombinacji będzie wyglądał następująco: 

```csharp
[TestFixture(typeof(int))]
[TestFixture(typeof(float))]
[TestFixture(typeof(double))]
[TestFixture(typeof(decimal))]
public class GenericCalculatorTests<T>
{
    [Test]
    public void AdditionTest()
    {
        var calculator = new GenericCalculator<T>();
 
        dynamic result = calculator.Add((dynamic) 2, (dynamic) 3);
 
        Assert.That(result, Is.EqualTo(5));
    }
}
```

 Do naszego testu przekazane są więc cztery numeryczne typy i dla każdego z nich sprawdzany jest wynik dodawania. Wynikowo, otrzymujemy cztery testy, każdy z innym typem `GenericCalculator`.
 
 Jeśli chcemy stworzyć instancję klasy generycznej, musimy podać wspomniany wcześniej `new()` constraint. Dla przykładu, weźmy sobie test na właściwość `Count` listy. Po dodaniu 2 elementów, Count musi być równy 2: 

```csharp
[TestFixture(typeof(ArrayList))]
[TestFixture(typeof(List<int>))]
[TestFixture(typeof(Collection<int>))]
public class ListsTests<T> where T : IList, new()
{
    [Test]
    public void CountTest()
    {
        var list = new T { 2, 3 };
 
        Assert.That(list, Has.Count.EqualTo(2));
    }
}
```

 Poszło gładko! Generic Test Fixture pozwala w prosty sposób na przekazanie zadeklarowanych przez nas generycznych typów do testów.
 
 P.S. Kod z niniejszego artykułu i całego kursu dostępny jest na GitHubie: [https://github.com/dariusz-wozniak/TddCourse](https://github.com/dariusz-wozniak/TddCourse/ "https://github.com/dariusz-wozniak/TddCours").

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
11. Testowanie klas generycznych
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