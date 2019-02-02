---
title: Kurs TDD cz. 10 — Teorie
date: "2015-02-25T20:03:22.000Z"
layout: post
draft: false
path: "/posts/kurs-tdd-10-teorie"
category: "Programowanie"
tags:
  - "TDD"
  - "Kurs TDD"
  - "C#"
  - "NUnit"
description: "Teoria jest specjalnym rodzajem testu, w którym weryfikujemy dane twierdzenie przy pomocy założeń (assumptions)."
---

Doskonałym uzupełnieniem wpisów o [testach parametryzowanych](/posts/kurs-tdd-8-testy-parametryzowane "Kurs TDD cz. 8: Testy parametryzowane") i [kombinatorycznych](/posts/kurs-tdd-9-testy-kombinatoryczne-i-sekwencyjne "Kurs TDD cz. 9: Testy kombinatoryczne i sekwencyjne") jest omówienie tzw. "teorii". Teoria jest specjalnym rodzajem testu, w którym weryfikujemy dane twierdzenie przy pomocy założeń (ang. _assumptions_). Dla porównania:

*   W **zwykłym teście** dostarczamy zbiór danych wejściowych metodzie testowanej, a następnie weryfikujemy zbiór danych wyjściowych ze zbiorem danych wyjściowych oczekiwanych.
*   **Teoria** ma na celu weryfikację ogólnego twierdzenia dla danych wejściowych spełniających żądane kryteria.

Rozbijmy teorię teorii na czynniki pierwsze i wtedy wszystko stanie się jasne i proste...

# Dane wejściowe

Dane wejściowe do teorii dostarczane są za pomocą:

*   zmiennych oznaczonych atrybutem `[Datapoint]`:

```csharp
[Datapoint]
public int Positive = 1;
```

*   lub tablic oznaczonych atrybutem `[Datapoints]`:

```csharp
[Datapoints]
public int[] Positive = { 1, 2 };
```
 Dane te są przekazywane do parametrów metody testowej w sposób kombinatoryczny. Technicznie możliwe jest też przekazanie danych wejściowych w ten sam sposób, co w przypadku [testów parametryzowanych](/posts/kurs-tdd-8-testy-parametryzowane "Kurs TDD cz. 8: Testy parametryzowane"), ale wtedy pisanie teorii nie ma sensu – wystarczy przecież zwykły test.

# Założenia

Teoria jest weryfikowana dla wszystkich danych wejściowych spełniających zadane przez nas kryteria. Do zdefiniowania założeń wykorzystujemy metodę `Assume.That`, która działa podobnie jak `Assert.That`, ale jeśli podany warunek nie zostanie spełniony, to rezultatem testu będzie stan "nierozstrzygnięty" (_Inconclusive_). Stan ten nie oznacza ani powodzenia testu, ani jego niepowodzenia. Taki stan powinniśmy (oraz nasze buildy) traktować obojętnie.

Ponadto, założenia (`Assume.That`) cechują się dodatkowymi własnościami:

*   Jeśli wszystkie założenia w ramach jednej teorii zostaną niespełnione, to test się nie powiedzie, niezależnie od dalszych jego asercji.
*   Założenia mogą być umieszczone w dowolnym miejscu metody (nie tylko na jej początku).
*   Poza założeniami, teoria zachowuje się jak zwykły test, tj. asercje determinują stan testu.

# Przykład

Nasza klasa Calculator jest idealnym przykładem możliwości wykorzystania teorii w praktyce. Jako przykład może posłużyć twierdzenie, że jeśli dzielna jest dodatnia, a dzielnik ujemny, to wynik dzielenia musi być ujemny. 

```csharp
public class Theory
{
    [Datapoint] public int Negative = -1;
 
    [Datapoint] public int Positive = 1;
 
    [Theory]
    public void WhenDividendIsPositiveAndDivisorIsNegative_TheQuotientIsNegative(int dividend,
        int divisor)
    {
        Assume.That(dividend > 0);
        Assume.That(divisor < 0);
 
        var calculator = new Calculator();
 
        float quotient = calculator.Divide(dividend, divisor);
 
        Assert.That(quotient < 0);
    }
}
```

Przyjrzyjmy się bliżej powyższemu przykładowi. Zestaw danych wejściowych składa się z dwóch intów: {-1, 1}. Obydwie wartości trafiają do parametrów metody testowej – _dividend_ i _divisor_. Kombinatorycznie mamy więc 4 przypadki testowe:

```
(-1,-1)
(-1,1)
(1,-1)
(1,1)
```

Pierwsze założenie `Assume.That(dividend > 0)` mówi o tym, że dzielna ma być dodatnia. Jeśli ten warunek nie zostanie spełniony, to test ma status nierozstrzygniętego.

W przeciwnym wypadku, sprawdzane jest drugie założenie `Assume.That(divisor < 0)`, które mówi o tym że dzielnik ma być ujemny. Tak samo jak wyżej – jeśli warunek będzie nieprawdą, test będzie nierozstrzygnięty.

W przeciwnym wypadku, test podąży ścieżką zwykłego testu, a jego stan będzie zależny od asercji. W wyniku, otrzymamy następujące rezultaty:

![fkJHZ[1]](cca60214-6b3e-4d71-9888-68af902a5bc5.png)

Widzimy, że testy nierozstrzygnięte nie wpływają na stan teorii. Działoby się to tylko wtedy (o czym mówiliśmy wcześniej) jeśli wszystkie założenia w danej metodzie byłyby niespełnione. Proste? I to bardzo!!

# Jakie są zalety takiego rozwiązania?

Po pierwsze, nasz zapis kodu przypomina bardziej próbę udowodnienia twierdzenia aniżeli zestaw metod nie związanych ze sobą w żaden sposób. Zapis przypadków testowych jest w takim przypadku łatwiejszy do czytania i zarządzania.

Po drugie, posiadamy odseparowaną część dla danych wejściowych, które można dodatkowo sensownie nazwać. Dane te są filtrowane przez nasze założenia.

Teorie można stosować z powodzeniem nie tylko dla twierdzeń matematycznych, ale też w algorytmach, strukturach danych, czy też logice biznesowej.

Należy pamiętać o tym, że przypadki testowe w teoriach generują się kombinatorycznie, dzięki czemu ilość naszych testów, a co za tym idzie – czas ich wykonania – może drastycznie wzrosnąć. Ponadto, teorie powinno się wykorzystywać tylko tam, gdzie mamy do czynienia z twierdzeniem, które chcemy udowodnić. W przypadku testów, które oparte są o przykładowe dane (_example-driven tests_), lepiej jest użyć tych "zwykłych" testów.

# Zobacz też

- xUnit.net również posiada wsparcie dla teorii – w pakiecie [xunit.extensions](https://www.nuget.org/packages/xunit.extensions/ "xunit.extensions"). Semantyka i sposób korzystania jest jednak inny niż w NUnicie.
- Kod do teorii z tego postu (minimalnie rozszerzony) znajduje się na GitHubie: [https://github.com/dariusz-wozniak/TddCourse/](https://github.com/dariusz-wozniak/TddCourse/ "github.com/dariusz-wozniak/TddCourse/").

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
10. Teorie
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