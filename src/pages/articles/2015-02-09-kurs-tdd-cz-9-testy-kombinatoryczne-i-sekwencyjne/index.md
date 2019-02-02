---
title: Kurs TDD cz. 9 — Testy kombinatoryczne i sekwencyjne
date: "2015-02-09T19:45:09.000Z"
layout: post
draft: false
path: "/posts/kurs-tdd-9-testy-kombinatoryczne-i-sekwencyjne"
category: "Programowanie"
tags:
  - "TDD"
  - "Kurs TDD"
  - "C#"
  - "NUnit"
description: "Naturalnym krokiem po omówieniu testów parametryzowanych jest przejście do testów kombinatorycznych i sekwencyjnych. Do dyspozycji mamy dwa atrybuty NUnita — [Combinatorial] oraz [Sequential]."
---

Naturalnym krokiem po [omówieniu testów parametryzowanych](/posts/kurs-tdd-8-testy-parametryzowane "Kurs TDD cz. 8: Testy parametryzowane") jest przejście do testów kombinatorycznych i sekwencyjnych. Do dyspozycji mamy dwa atrybuty NUnita — `[Combinatorial]` oraz `[Sequential]`. Sprawa jest bardzo prosta, więc zrozumienie działania tych dwóch funkcji nie przysporzy żadnych problemów.

Bieżący kod z tego kursu znajduje się na GitHubie: [https://github.com/dariusz-wozniak/TddCourse/](https://github.com/dariusz-wozniak/TddCourse/).

# Testy kombinatoryczne

Kombinatoryczne testy wykonywane są na zestawie wszystkich możliwych przypadków. Wartości przekazywane są w atrybucie parametru `[Values]`. Tak naprawdę, użycie atrybutu `[Combinatorial]` jest opcjonalne – gdy metoda będzie miała co najmniej dwa parametry opatrzone atrybutem `[Values]`, test kombinatoryczny będzie domyślnym dla takiego scenariusza. Przykład użycia testu kombinatorycznego: 

```csharp
[Test]
[Combinatorial]
public void Divide_DividendIsPositiveAndDivisorIsNegative_ReturnsNegativeQuotient(
    [Values(1, 2, 3, 4)] int dividend,
    [Values(-1, -2, -3)] int divisor)
{
    var calc = new Calculator();
    float quotient = calc.Divide(dividend, divisor);
 
    Assert.That(quotient < 0);
}
```

 W powyższym przykładzie chcemy udowodnić, że jeśli dzielna jest dodatnia, a dzielnik ujemny, to wynik ma być ujemny. Dla wartości dzielnej aplikujemy cztery wartości, dla dzielnika 3, a zatem 4 x 3 = 12 przypadków:

 ```
(1,-1)
(1,-2)
(1,-3)
(2,-1)
(2,-2)
(2,-3)
(3,-1)
(3,-2)
(3,-3)
(4,-1)
(4,-2)
(4,-3)
```

# Testy sekwencyjne

W teście sekwencyjnym dane wykonywane są w porządku, w jakim zostały zdefiniowane. Podobnie jak w przypadku testów kombinatorycznych, dane wprowadzamy w atrybucie parametru `[Values]`. Przykład: 

```csharp
[Test]
[Sequential]
public void Divide_DivisorIsNegativeOfDividend_ReturnsMinusOne(
    [Values(1, 2, 30)] int dividend,
    [Values(-1, -2, -30)] int divisor)
{
    var calc = new Calculator();
    float quotient = calc.Divide(dividend, divisor);
 
    Assert.That(quotient == -1);
}

Powyższy
```

 Powyższy test ma dowieść, że a ÷ (-a) = -1. Dla powyższego zestawu danych, zostały wygenerowane następujące przypadki:
 
```
(1,-1)
(2,-2)
(30,-30)
```
 
 A co jeśli ilość parametrów się nie zgadza? NUnit przypisze brakującym elementom wartość `default(T)`.

# Podsumowanie

Do testów kombinatorycznych i sekwencyjnych wykorzystujemy atrybuty `[Combinatorial]` i `[Sequential]`. Zobacz też: [Kurs TDD cz. 8: Testy parametryzowane](/posts/kurs-tdd-8-testy-parametryzowane "Kurs TDD cz. 8: Testy parametryzowane").

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
9. Testy kombinatoryczne i sekwencyjne
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