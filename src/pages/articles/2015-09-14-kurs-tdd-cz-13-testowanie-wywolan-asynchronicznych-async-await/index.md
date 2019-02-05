---
title: Kurs TDD cz. 13 — Testowanie wywołań asynchronicznych (async await)
redirect_from: 
  - "/2015/09/14/kurs-tdd-cz-13-testowanie-wywolan-asynchronicznych-async-await/"
date: "2015-09-14T18:01:36.000Z"
layout: post
draft: false
path: "/posts/kurs-tdd-13-testowanie-wywolan-asynchronicznych-async-await"
category: "Programowanie"
tags:
  - "TDD"
  - "Kurs TDD"
  - "C#"
  - "NUnit"
description: "C# 5.0 wniósł wiele dobroci, m.in. obsługę wywołań asynchronicznych za pomocą słów kluczowych async i await. Rozwiązanie, ze względu na prostotę obsługi i skuteczność, cieszy się do dziś sporą popularnością. Jak testować wywołania asynchroniczne? Tego dowiemy się w tym odcinku!"
---

C# 5.0 wniósł wiele dobroci, m.in. obsługę wywołań asynchronicznych za pomocą słów kluczowych `async` i `await`. Rozwiązanie, ze względu na prostotę obsługi i skuteczność, cieszy się do dziś sporą popularnością. Jak testować wywołania asynchroniczne? Tego dowiemy się w tym odcinku!

Załóżmy, że nasza metoda testowa to asynchroniczne wywołanie dzielenia (na potrzeby artykułu dodaliśmy wywołanie `Task.Delay`): 

```csharp
public async Task<float> DivideAsync(double dividend, double divisor)
{
    if (divisor == 0) throw new DivideByZeroException();
 
    await Task.Delay(millisecondsDelay: 1000)
              .ConfigureAwait(continueOnCapturedContext: false);
 
    return (float)dividend/(float)divisor;
}
```

 W jaki sposób możemy napisać testy do powyższego kodu?

## Sposób 1: standardowy

Najprostszy sposób przetestowania asynchronicznej metody przedstawiony jest poniżej: 
```csharp
[Test]
public async void DivideAsyncTest()
{
    var calculator = new Calculator();
    float quotient = await calculator.DivideAsync(10, 2);
    Assert.That(quotient, Is.EqualTo(5));
}
```

## Sposób 2: `async` `await` i wyrażenie lambda

Drugim sposobem jest wykorzystanie wyrażeń lambda w kontekście wywołań asynchronicznych: 

```csharp
[Test]
public void DivideAsyncLambdaTest()
{
    var calculator = new Calculator();
    Assert.That(async () => await calculator.DivideAsync(10, 2), Is.EqualTo(5));
}
```
## Testowanie wyjątków

Aby przetestować czy asynchroniczna metoda rzuca wyjątek, możemy wykorzystać wyrażenie lambda. Tutaj test będzie wyglądać następująco: 

```csharp
[Test]
public void WhenDivisorIsZero_ThenDivideByZeroExceptionIsThrown()
{
    var calculator = new Calculator();
    Assert.Throws<DivideByZeroException>(async () => await calculator.DivideAsync(10, 0));
}
```

## Na koniec

Baczny czytelnik powinien zauważyć, że najpierw napisaliśmy kod, a później testy. W tym przypadku omawiamy jednak techniczne aspekty testów jednostkowych; pisząc kod nie zapominajmy o kolejności najpierw testy, później logika biznesowa.

Kod źródłowy z tej części kursu jest dostępny na GitHubie: [https://github.com/dariusz-wozniak/TddCourse/](https://github.com/dariusz-wozniak/TddCourse/).

## Źródła

*   [simoneb's blog: Async Support in NUnit](http://simoneb.github.io/blog/2013/01/19/async-support-in-nunit/)
*   [Thomas Levesque's .NET Blog: Async unit tests with NUnit](http://www.thomaslevesque.com/2015/02/01/async-unit-tests-with-nunit/)


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
13. Testowanie wywołań asynchronicznych (async await)

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