---
title: "Kurs TDD cz. 22 — Pokrycie kodu testami (code coverage)"
date: "2016-06-13T19:56:51.000Z"
layout: post
draft: false
path: "/posts/kurs-tdd-22-pokrycie-kodu-testami-code-coverage/"
category: "Programowanie"
tags:
  - "TDD"
  - "Kurs TDD"
description: "Przyjrzymy się bliżej tematowi pokrycia kodu testami (code coverage)."
---

**Pokrycie kodu (_code coverage_)** testami to:

> (liczba wyrażeń pokrytych testami) ÷ (liczba wszystkich wyrażeń) × 100%

Innymi słowy, jest to procentowy współczynnik pokrycia kodu testami. Pokrycie kodu najczęściej mierzy się badając liczbę wyrażeń (_statements_), choć niekiedy spotkać się można z pokryciem kodu opartym o:

- ilość linii kodu,
- ilość branchy (_branch coverage_), 
- ilość stanów (_condition coverage_),
- ilość funkcji, modułów lub metod.

Kiedyś w rozmowie jeden z menadżerów chwalił się, że jego zespół utrzymał założone 90% pokrycia kodu testami jednostkowymi. Z rozmowy z członkiem innego zespołu, programista poinformował mnie, że jego zespół ma roczny cel pokrycia nowo pisanego kodu testami jednostkowymi w stopniu 85%. Jest to przykład, który zapewne zna sporo ludzi ze swojego lub bliskiego podwórka. Czy zespół posiadający stopień pokrycia kodu równym 90% jest lepszy, skuteczniejszy, bądź bardziej produktywny niż zespół z pokryciem 85%? Jakie jest minimum i optimum pokrycia kodu testami jednostkowymi? Dlaczego żaden z zespołów nie postawił sobie za cel pokrycia stuprocentowego? Przyjrzymy się bliżej problemowi…

---

> "I expect a high level of coverage. Sometimes managers require one. There's a subtle difference." — Brian Marick

---

Pokrycie kodu testami jednostkowymi w 100% brzmi jak utopia. Jak to jednak najczęściej bywa, utopijne wizje zmieniają się bardzo szybko w piekło…

Pierwszy przykład: Nasze testy jednostkowe nie posiadają żadnych asercji, a ich pokrycie wynosi właśnie sto procent. Pożytek z takich testów jest żaden, jednak samo pokrycie jest "idealne".

Drugi prz... absurd – spójrzmy na przykładową metodę dzielenia:

```csharp
public float Divide(int dividend, int divisor)
{
    return (float)dividend / divisor;
}
```

 Tutaj, 100% pokrycia uzyskamy podstawiając dwie dowolne liczby. Samo pokrycie kodu testami jednostkowymi nie powie nam o jakości parametrów testowych, a w powyższym przypadku istnieje sporo przypadków brzegowych, które nie zostają uwzględnione w tej metryce.
 
 Przykład numer 3. Spójrzmy na kod:
 
 ```csharp
public class CustomerValidator
{
    public bool Validate(ICustomer customer)
    {
        if (customer.Age < 18) return false;
        return true;
    }
}
```

 Powyższa metoda wydaje się być na pierwszy rzut oka całkiem w porządku. Co jeśli natomiast podstawimy za zmienną customer wartość null? Nasze testy, pomimo pełnego pokrycia nie wskażą na ten problem. I pytanie do tego przykładu, która sytuacja jest korzystniejsza – czy lepiej jest zostawić kod taki jak powyżej, ale ze stuprocentowym pokryciem czy może lepiej jest obsłużyć przypadek nullowy kosztem pokrycia kodu?

---

> "100 percent coverage is still far from enough." — z _Facts and Fallacies of Software Engineering_

---

Czy posiadając upragione 100% możemy być pewni, że nasz kod jest równie wysoko wolny od błędów? Rozważamy cały czas przypadek testów jednostkowych, a jak wiemy, testy takie wykonywane są w izolacji. Jak się okazuje, blisko 35% defektów w oprogramowaniu powstaje przez brakujące ścieżki logiczne, a 40% na wskutek wykonania unikalnej ich kombinacji. Sto procent pokrycia kodu testami jednostkowymi nie wykryje nam tej klasy błędów.

Błędy wynikające z interakcji między obiektami nie mogą być pokryte w pełni przez testy jednostkowe. Rozwiązaniem tego problemu mogą być testy interakcyjne, akceptacyjne lub manualne.

# 80%, 85% czy 90%?

Skąd wzięły się te liczby? Dlaczego nie celujemy np. w 86% lub 91%? Co jeśli zespół osiągnął 1 punkt procentowy mniej niż założono? Problem nie leży w samej ilości pokrycia kodu. Główny zarzut dla podejścia pokrycia jako celu to fakt, że ta liczba nie ma odzwierciedlenia z rzeczywistą jakością kodu i testów. Wysoka wartość metryki może zasłaniać wiele ukrytych problemów związanych z jakością testów. Może też zmusić do optymalizacji testów względem tej metryki przy braku uwagi na samą jakość kodu i testów. W skrajnym przypadku zespół piszący słaby kod oraz kiepskie testy, może osiągnąć 100% pokrycia i spełnić dany cel. W takim scenariuszu metryka została spełniona, lecz korelacja z jakością jest odwrotnie proporcjonalna.

# Czy zrezygnować więc z tej metryki?

Samo pokrycie kodu testami to… doskonała metryka. Metryka, która pozwala znaleźć obszary kodu niepokryte testami. Narzędzie do mierzenia pokrycia kodu powinno należeć do obowiązkowego wyposażenia każdego programisty. Za pomocą tego typu narzędzia jesteśmy w stanie błyskawicznie ocenić, które części kodu nie zostały pokryte testami oraz przeanalizować czy do danej części kodu potrzebne są testy.

# Podsumowanie

Nie używaj metryki pokrycia kodu jako celu zespołowego. Nie ma korelacji między jakością kodu, a pokryciem kodu testami jednostkowymi. Wprowadzenie tej metryki może przyczynić się nawet do spadku jakości kodu.

Pokrycie kodu pozwala jednak w prosty sposób zidentyfikować obszar kodu niewytestowanego.

# Czytaj dalej

Polecam lekturę: [How much test coverage do you need? – The Testivus Answer](http://www.developertesting.com/archives/month200705/20070504-000425.html)

# Źródła

*   Robert L. Glass Facts and Fallacies of Software Engineering
*   Brian Merick [How to Misuse Code Coverage](http://www.exampler.com/testing-com/writings/coverage.pdf)
*   Martin Fowler [Test Coverage](http://martinfowler.com/bliki/TestCoverage.html)
*   Mark Seemann [Code coverage is a useless target measure](http://blog.ploeh.dk/2015/11/16/code-coverage-is-a-useless-target-measure/)


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
6. [Mock, stub, fake, spy, dummy](/posts/kurs-tdd-19-mock-stub-fake-spy-dummy)
7. [Mockowanie DateTime.Now, random, static, itp.](/posts/kurs-tdd-20-mockowanie-datetime-now-random-static-itp)
8. [Rodzaje frameworków do tworzenia atrap](/posts/kurs-tdd-21-rodzaje-frameworkow-do-tworzenia-atrap/)

Część III: Teoria

1.  Pokrycie kodu testami (Code Coverage)
2. [Czy to się opłaca?](/posts/kurs-tdd-23-czy-to-sie-oplaca/)
3. [Czy pisać testy jednostkowe do istniejącego kodu (legacy code)?](/posts/kurs-tdd-24-czy-pisac-testy-jednostkowe-do-istniejacego-kodu-legacy-code/)
4. [Otwarte pytania](/posts/kurs-tdd-25-otwarte-pytania/)

</div>
</div>
</div>
<!-- tdd-course-infobox-end -->