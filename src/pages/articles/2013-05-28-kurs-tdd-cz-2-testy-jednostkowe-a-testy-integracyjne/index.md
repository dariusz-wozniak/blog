---
title: Kurs TDD cz. 2 — Testy jednostkowe a testy integracyjne — różnice
redirect_from: 
  - "/2013/05/28/kurs-tdd-czesc-2-testy-jednostkowe-a-testy-integracyjne/"
date: "2013-05-28T21:19:04.000Z"
layout: post
draft: false
path: "/posts/kurs-tdd-2-testy-jednostkowe-a-testy-integracyjne/"
category: "Programowanie"
tags:
  - "TDD"
  - "Kurs TDD"
description: "W części drugiej kursu przyjrzymy się rodzajom testów w świecie Test-Driven Development i poznamy różnice między testami jednostkowymi, a integracyjnymi."
---

W [części pierwszej](/posts/kurs-tdd-1-wstep/ "części pierwszej") kursu omówiłem czemu służy technika i jak wygląda proces pisania wedle TDD, opisałem cykl Red-Green-Refactor, a na koniec przedstawiłem korzyści płynące z jej używania. W części drugiej kursu przyjrzymy się rodzajom testów w świecie Test-Driven Development i poznamy różnice między testami jednostkowymi, a integracyjnymi. Cztery główne rodzaje testów w świecie TDD to:

*   testy jednostkowe (_unit tests_) — testujemy pojedynczą, jednostkową część kodu: zazwyczaj klasę lub metodę;
*   testy integracyjne (_integration tests_) — testujemy kilka komponentów systemu jednocześnie;
*   testy regresyjne (_regression tests_) — po wprowadzeniu naszej zmiany uruchamiane są wszystkie testy w danej domenie biznesowej celem sprawdzenia czy zmiana nie spowodowała błędu w innej części systemu;
*   testy akceptacyjne (_acceptance tests_) — testy mające na celu odpowiedzieć na pytanie czy aplikacja spełnia wymagania biznesowe.

Jest tego sporo więcej, testy black/white box, testy zgodności (_conformance tests)_, _smoke tests_, itp. W TDD kluczową rzeczą jest zrozumienie różnic między testami jednostkowymi, a integracyjnymi.

# Różnice między testami jednostkowymi, a integracyjnymi

Aby zrozumieć TDD należy wiedzieć czym jest test jednostkowy oraz czym jest integracyjny. TDD oparte jest tylko i wyłącznie o testy jednostkowe, tj.—piszemy jednostkowe testy przed implementacją kodu (Red-Green-Refactor), badamy pokrycie testów jednostkowych, uruchamiamy przy domyślnym buildzie testy jednostkowe. Testy integracyjne mogą być pisane opcjonalnie.

Podstawową różnicą między obydwoma rodzajami testów jest to, że **testy jednostkowe testują pojedyncza część kodu**, natomiast **testy integracyjne mają na celu sprawdzenie kilka komponentów działających razem**.

W teście jednostkowym wszystkie zależności powinny być zastąpione przez tzw. _mock objects_ (będzie o tym mowa w kolejnych częściach), które symulują ich zachowanie. W teście integracyjnym możemy skorzystać z kilku lub wszystkich zależności systemu.

Dla przykładu—funkcja naszej metody to pobranie z bazy danych pewnych informacji i zwrócenie posortowanych danych. Uszczegółówmy sobie ten przykład jeszcze bardziej—pobieramy z bazy danych listę osób i sortujemy je alfabetycznie wg nazwiska. W testach jednostkowych musimy zasymulować (albo: zastąpić) połączenie z bazą danych i przygotować ręcznie listę osób. Możemy to najprościej wykonać poprzez wspomniane już _mock objects_, które mogą być stworzone ręcznie lub przy pomocy dowolnego frameworka do mockowania. W teście integracyjnym możemy sobie pozwolić na połączenie z rzeczywistą bazą danych, dzięki czemu będziemy operować na rzeczywistych danych, pobranych z bazy.

Drogi Czytelniku! Jeśli powyższy akapit nie jest dla Ciebie do końca jasny, nie przejmuj się! Powinieneś wiedzieć jedynie, że w testach jednostkowych klasy powinny być w izolacji względem innych zależności, tj. innych klas i zasobów (np. baza danych, Internet, input/output). Wszystkie te zależności powinny być _mockowane_. Jest to jednak temat na dalszą część tego kursu. Jeśli nauczysz się jak _mockować_ obiekty, wszystko stanie się o wiele jaśniejsze :)

Ta i pozostałe różnice zostały podane w poniższej tabeli:

| Zagadnienie | Test jednostkowy | Test integracyjny |
|-------------|------------------|-------------------|
| **Zależności** | Testowany **jednostkowy element (klasa, metoda) w izolacji**. | Testowana **więcej niż jedna wewnętrzna lub zewnętrzna zależność.** |
| **Punkt awarii (_failure point_)** | Tylko **jeden potencjalny punkt awarii** (jedna logiczna asercja per test*). | **Wiele potencjalnych punktów awarii.** |
| **Szybkość działania** | **Bardzo szybko**, dużo poniżej 1 sekundy. | **Może trwać długo**, ze względu na czasochłonne operacje np. dostęp do bazy danych, I/O, operacje na sesji.
| **Konfiguracja** | Test musi działać na każdej maszynie **bez dodatkowej konfiguracji.** | Test **może być zależny od konfiguracji**, np. machine.config (login/hasło) do bazy danych.

\* — Test jednostkowy może zawierać więcej niż jeden Assert pod warunkiem, że wszystkie testują jeden element. Czytaj więcej: [http://ayende.com/blog/2684/what-is-one-assert](http://ayende.com/blog/2684/what-is-one-assert "http://ayende.com/blog/2684/what-is-one-assert").

# Struktura w kodzie

W związku z różnicami jakie dzielą obydwa typy testów, zarówno jednostkowe, jak i integracyjne testy powinny być umieszczone w oddzielnych projektach: 

![solution](ed3218cd-f05e-4116-a534-d13e772c971a.png)

# Bonus!

Dobrą ilustracją testu jednostkowego i testu integracyjnego są poniższe zdjęcia:

W teście jednostkowym testujemy nasze atomiczne elementy (klasy, metody) w izolacji od pozostałych elementów systemu:

![Unit Test](814b2a81-0f4c-4da0-8727-9cdb0940c194.jpg "test jednostkowy")

W teście integracyjnym testujemy wiele lub wszystkie moduły naszego systemu:

![IntegrationTest](addc32e6-a4f9-48d6-83bb-7475e39ac6b0.jpg "test integracyjny")

Źródła zdjęć: 

* [http://indianautosblog.com/2008/12/iihs-suzuki-sx4-crash-test](http://indianautosblog.com/2008/12/iihs-suzuki-sx4-crash-test "http://indianautosblog.com/2008/12/iihs-suzuki-sx4-crash-test")
* [http://www.rpmgo.com/automotive-articles/car-parts.html](http://www.rpmgo.com/automotive-articles/car-parts.html "http://www.rpmgo.com/automotive-articles/car-parts.html")


<!-- tdd-course-infobox-start -->
<div class="boxBorder">

<div style="text-align: center; font-size: 40px">Kurs TDD</div>

----

<div class="row">
<div class="column">

Część I: Testy jednostkowe – wstęp

1. [Wstęp do TDD](/posts/kurs-tdd-1-wstep/)
2. Testy jednostkowe, a integracyjne
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

22. [Pokrycie kodu testami (Code Coverage)](/posts/kurs-tdd-22-pokrycie-kodu-testami-code-coverage/)
1. [Czy to się opłaca?](/posts/kurs-tdd-23-czy-to-sie-oplaca/)
1. [Czy pisać testy jednostkowe do istniejącego kodu (legacy code)?](/posts/kurs-tdd-24-czy-pisac-testy-jednostkowe-do-istniejacego-kodu-legacy-code/)
1. [Otwarte pytania](/posts/kurs-tdd-25-otwarte-pytania/)

</div>
</div>
</div>
<!-- tdd-course-infobox-end -->