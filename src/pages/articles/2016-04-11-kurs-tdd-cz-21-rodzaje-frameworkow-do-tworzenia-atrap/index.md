---
title: Kurs TDD cz. 21 — Rodzaje frameworków do tworzenia atrap
redirect_from:
    - "/2016/04/11/kurs-tdd-cz-21-rodzaje-frameworkow-do-tworzenia-atrap/"
date: "2016-04-11T16:52:10.000Z"
layout: post
draft: false
path: "/posts/kurs-tdd-21-rodzaje-frameworkow-do-tworzenia-atrap/"
category: "Programowanie"
tags:
  - "TDD"
  - "Kurs TDD"
description: "Rodzaje framerków do tworzenia atrap możemy podzielić na dwie kategorie: constrained (z ang. ograniczony) i unconstrained (nieograniczony)"
---

**Rodzaje framerków do tworzenia atrap** możemy podzielić na dwie kategorie:

*   _constrained_ (z ang. ograniczony)
*   _unconstrained_ (nieograniczony)

# _constrained_

Do pierwszej kategorii zaliczamy wszystkie do tej pory poznane frameworki do tworzenia atrap – [Moq](/posts/kurs-tdd-15-wstep-do-moq), [FakeItEasy](/posts/kurs-tdd-17-fakeiteasy), [NSubstite](/posts/kurs-tdd-18-nsubstitute) – a także Rhino Mocks, NMock oraz EasyMock. Ich cechą charakterystyczną jest ograniczona możliwość tworzenia atrap.

Biblioteki te generują kod dziedziczący atrapy w czasie rzeczywistym, w oparciu o kod pośredni (IL). Najczęściej atrapy są tworzone w oparciu o wzorzec projektowy dynamicznego proxy, który wymaga tego aby kod umożliwiał dziedziczenie. Oznacza to, że aby stworzyć atrapę potrzebujemy interfejsu do naszej klasy lub metody wirtualnej. Jako, że kluczem do stworzenia atrapy jest dziedziczenie, nasza klasa/metoda nie może być statyczna, niepubliczna, sealed oraz musi posiadać publiczny konstruktor. Oznacza to też, że kod zawarty w konstruktorze oraz polach klasy jest wykonywany przy tworzeniu atrapy.

**Czym jest dynamiczne proxy?**

> Dynamiczne proxy to klasa, która implementuje interfejs lub klasę w trakcie wykonywania programu (_run-time_).

Biblioteki te są zwykle darmowe, a obiekty proxy są zwykle tworzone przy użyciu biblioteki Castle.Windsor.

# _unconstrained_

Frameworki o "nieograniczonych możliwościach" to Typemock Isolator, JustMock i Microsoft Fakes. Są one napisane w oparciu o Common Language Runtime (CLR) Profiler, który udostępnia API pozwalające na większą kontrolę nad wykonywanym kodem. Możemy zatem wstrzykiwać nasz kod przed kompilacją kodu pośredniego oraz mamy dostęp do _eventów_ wywoływanych w trakcie wykonywania naszego kodu. Większa kontrola nad generowanym kodem oznacza, że frameworki te pozwalają na tworzenie atrap dla kodu, który nie musi być dziedziczony, a więc klasy/metody statyczne, prywatne, biblioteki zewnętrzne, klasy systemowe (np. DateTime), itp. Ze względu na stopień skomplikowania pisania kodu w oparciu o API Profilera, frameworki te (poza Microsoftowym) są dostępne odpłatnie.

# Biblioteki

Przykłady bibliotek

**_constrained_** generują kod pośredni w trakcie wykonywania (run-time):

*   Moq
*   FakeItEasy
*   NSubstitute
*   Rhino Mocks
*   NMock
*   EasyMock
*   JustMock Lite

**_unconstrained_** bazujące na API Profilera:

*   Typemock Isolator
*   JustMock
*   Microsoft Fakes (dawniej Moles)

# No dobra, to który typ biblioteki wybrać?

Osobiście preferuję podejście przy użyciu "ograniczonych" frameworków, gdyż wymuszają one pisanie dobrego kodu w oparciu o programowanie zorientowane obiektowo. Podobnie jest z refaktoryzacją i poprawkami w nieprzetestowanym kodzie. Izolacja miejsca, które zmieniamy oraz tworzenie kodu który będzie w łatwy sposób testowalny również powoduje zwiększenie jakości kodu. Nie chcę przez to powiedzieć, że samo wykorzystanie bibliotek _constrained_ gwarantuje nam poprawę kodu "gratis". Dostajemy jednak informację zwrotną na temat designu naszych klas—nie możemy stworzyć atrapy jeśli nasza klasa nie jest testowalna z powodu np. statyczności lub braku interfejsu.

Użycie bibliotek _unconstrained_ wiąże się z kilkoma pułapkami, m.in.

*   Możemy testować zbyt dużo nie wiedząc o tym. Niemal pełna dowolność powoduje, że możemy np. zacząć testować moduły prywatne, które z reguły nie powinny być testowane explicite.
*   Nie mamy informacji zwrotnej o jakości naszych klas względem programowania zorientowanego obiektowo.
*   _Vendor lock-in_. Uzależniamy się od dostawcy naszego frameworka. O ile można w prosty sposób zmigrować kod między Moq, FakeItEasy i NSubstitute, tak w przypadku tej klasy bibliotek może być o wiele trudniej. API frameworków _unconstrained_ różnią się w bardziej znaczący sposób, przez co migracja kodu może być o wiele bardziej bolesna.

Frameworki _constrained_ wymagają więcej wiedzy oraz doświadczenia na temat testowania oraz dobrego kodu, ale dzięki temu zyskujemy natychmiastową informację zwrotną na temat jakości naszego kodu. Ja stosuję z powodzeniem tę grupę bibliotek zarówno w przypadku _greenfield_ (nowy kod), jak i _brownfield_ (stary kod).

A wy jakie macie zdanie na temat tych dwóch grup frameworków?

# Źródła

*   Roy Osherove. _Art of Unit Testing._ 2014\. Str. 110-114
*   [Oracle Java SE Documentation: Dynamic Proxy Classes](https://docs.oracle.com/javase/6/docs/technotes/guides/reflection/proxy.html)
*   [Microsoft Development Network: Profiling Overview](https://msdn.microsoft.com/en-us/library/bb384493.aspx?f=255&MSPPError=-2147217396)

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
8. Rodzaje frameworków do tworzenia atrap

Część III: Teoria

22. [Pokrycie kodu testami (Code Coverage)](/posts/kurs-tdd-22-pokrycie-kodu-testami-code-coverage/)
1. [Czy to się opłaca?](/posts/kurs-tdd-23-czy-to-sie-oplaca/)
1. [Czy pisać testy jednostkowe do istniejącego kodu (legacy code)?](/posts/kurs-tdd-24-czy-pisac-testy-jednostkowe-do-istniejacego-kodu-legacy-code/)
1. [Otwarte pytania](/posts/kurs-tdd-25-otwarte-pytania/)

</div>
</div>
</div>
<!-- tdd-course-infobox-end -->