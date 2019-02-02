---
title: Kurs TDD cz. 7 — Inicjalizacja i czyszczenie danych (SetUp i TearDown)
date: "2014-03-27T14:51:40.000Z"
layout: post
draft: false
path: "/posts/kurs-tdd-7-inicjalizacja-i-czyszczenie-danych-setup-i-teardown/"
category: "Programowanie"
tags:
  - "TDD"
  - "Kurs TDD"
  - "C#"
  - "NUnit"
description: "W tej części kursu zajmiemy się pojęciem inicjalizacji i czyszczenia danych."
---

W tej części kursu zajmiemy się pojęciem inicjalizacji i czyszczenia danych. Wielkiej filozofii tutaj nie ma; w NUnit działa to tak:

*   Jeśli chcemy, aby jedna z metod uruchamiała się **przed każdym uruchomieniem naszego testu**, aby np. inicjalizować dane, nakładamy na nią atrybut `[SetUp]`.
*   Jeśli zechcemy, aby metoda uruchamiała się **po każdym uruchomieniu testu** w celu np. czyszczenia danych — nakładamy atrybut `[TearDown]`.

Reguły gry są jasne:

*   Jeśli użyjemy więcej niż jeden atrybut `[SetUp]` i `[TearDown]` dla jednej klasy testowej, to kod skompiluje się poprawnie, ale **testy nie uruchomią się.**
*   Jeśli chcemy natomiast uruchomić kod jednorazowo przed wywołaniem\\po wywołaniu **wszystkich** naszych testów w obrębie jednej klasy testowej, możemy skorzystać z atrybutów `[TestFixtureSetUp]` i `[TestFixtureTearDown]`.

Całą ideę prezentuje poniższy kod: 
```csharp
public class SuccessTests
{
    [SetUp]
    public void Init() {...}
 
    [TearDown]
    public void Dispose() {...}
 
    [Test]
    public void Add() {...}
}
```

# Dlaczego tego nie stosować?

Z mojego punktu widzenia, nie powinniśmy używać atrybutów `[SetUp]` i `[TearDown]` w świecie testów jednostkowych. Główne powody to:

*   Kod, w którym korzystamy z tych atrybutów staje się trudniejszy do czytania...
*   ...oraz debugowania
*   W niektórych przypadkach, możemy zainicjalizować więcej zmiennych niż tak naprawdę potrzebujemy.
*   Jeśli pojawi się błąd w naszych testach, będzie on trudniejszy do zlokalizowania.
*   Dodatkowy problem to fakt, że programista musi znać te atrybuty. Niby znikoma, ale jednak istniejąca krzywa uczenia się.

Co zatem używać do inicjalizacji? Najprostsze metody są najlepsze, a zatem:

*   Jeśli nie mamy zbyt wielu rzeczy do ustawienia, możemy sobie pozwolić na [duplikację kodu](/posts/kurs-tdd-6-dobre-i-zle-praktyki-testow-jednostkowych) w naszych testach.
*   W przypadku kiedy musimy złożyć coś większego, najlepiej przenieść wspólny kod do metody. Należy też zadać sobie pytanie o to, czy aby nasze klasy są poprawnie zbudowane pod kątem obiektowego projektowania.

Czyszczenie danych? Taki problem nie powinien przecież występować w przypadku testów jednostkowych, gdyż nie korzystamy z zewnętrznych zależności, prawda?

# Konkluzja?

**Atrybuty `[SetUp]` i `[TearDown]` nie powinny być używane w naszych testach jednostkowych.** Ich stosowanie prowadzi do tego, że nasze testy są trudniejsze w zrozumieniu. Zamiast atrybutów, lepiej posłużyć się duplikacją kodu (możemy sobie na to pozwolić w świecie testów jednostkowych) lub wrzuceniem wspólnego kodu do metody.

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
7. Inicjalizacja i czyszczenie danych (SetUp i TearDown)
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

22. [Pokrycie kodu testami (Code Coverage)](/posts/kurs-tdd-22-pokrycie-kodu-testami-code-coverage/)
1. [Czy to się opłaca?](/posts/kurs-tdd-23-czy-to-sie-oplaca/)
1. [Czy pisać testy jednostkowe do istniejącego kodu (legacy code)?](/posts/kurs-tdd-24-czy-pisac-testy-jednostkowe-do-istniejacego-kodu-legacy-code/)
1. [Otwarte pytania](/posts/kurs-tdd-25-otwarte-pytania/)

</div>
</div>
</div>
<!-- tdd-course-infobox-end -->