---
title: Kurs TDD cz. 6 — Dobre i złe praktyki testów jednostkowych
redirect_from: 
  - "/2013/11/18/kurs-tdd-czesc-6-dobre-i-zle-praktyki-testow-jednostkowych/"
date: "2013-11-17T23:36:56.000Z"
layout: post
draft: false
path: "/posts/kurs-tdd-6-dobre-i-zle-praktyki-testow-jednostkowych"
category: "Programowanie"
tags:
  - "TDD"
  - "Kurs TDD"
description: "W tej części opisane zostaną dobre i złe praktyki stosowane przy pisaniu testów jednostkowych."
---

W tej części opisane zostaną dobre i złe praktyki stosowane przy pisaniu testów jednostkowych.

Co ciekawe, praktyki te odbiegają niekiedy od ogólnie przyjętych założeń i standardów kodowania. Przykładem może być zasada DRY—Don't Repeat Yourself. W TDD kopiuj-wklej to niemal chleb powszedni; w TDD możemy stosować zasadę zgoła odwrotną—Do Repeat Yourself! Taka odrębność wynika z tego, że w kodzie testowym musimy zminimalizować prawdopodobieństwo pojawienia się błędu. Kod testowy musi być też bardzo prosty i czytelny. Kod testowy nie jest kodem produkcyjnym i w konsekwencji nie będziemy mieli potrzeby rozwijania kodu ani jego funkcjonalności.

Nie chodzi o to, by w TDD zapomnieć o dobrych praktykach programowania. Zasada DRY może w TDD przybierać postać Do Repeat Yourself jeśli chodzi o małe kawałki kodu, ale w przypadku testowania starszego, zawiłego i nie pokrytego testami kodu, wyodrębnienie wspólnej logiki dla wszystkich testów jednostkowych może okazać się koniecznością. Jak zawsze – nadrzędną zasadą jest zasada zdrowego rozsądku.

Wprowadziłem podział zasad na dwie grupy. Do pierwszej wrzuciłem żelazne zasady pisania testów jednostkowych, których powinniśmy się trzymać zawsze i wszędzie. O części z nich była mowa w poprzednich częściach, tutaj zostały zebrane w całość. W drugiej części przedstawiłem dobre i złe praktyki. Nie należą one do kanonu TDD; zostały zebrane z różnych źródeł, nie tylko książkowych, ale też blogów, StackOverflow czy też z mojego doświadczenia.

# Najważniejsze zasady pisania testów jednostkowych

*   **Szybkość**: Testy jednostkowe powinny uruchamiać się szybko, tak aby nie opóźniały znacząco wykonanie builda. Szybko oznacza dużo poniżej 1 s.
*   **Izolacja**: Testy powinny być od siebie odizolowane i niezależne od siebie. Test nie powinien uruchamiać innego testu.
*   **Powtarzalność**: Testy powinny być powtarzalne na każdym środowisku. Testy nie mogą mieć stanów początkowych, ani zasobów do wyczyszczenia. Oznacza to także brak zależności w stosunku do zasobów zewnętrznych (baza danych, system plików, itd.)
*   **Zgodność**: Testy powinny dawać ten sam rezultat za każdym uruchomieniem.
*   **Atomiczność**: Atomiczne testy oznaczają, że test jednoznacznie jest zielony lub czerwony. Nie ma przypadku, gdy test jest zielony, ale testowana logika nie działa prawidłowo z jakichś powodów. Nie ma testów częściowo poprawnych.
*   **Asercja:** Każdy test powinien mieć co najmniej jedną asercję. To jest największa oczywistość, ale należy też pamiętać o tym, że test bez asercji przechodzi jako zielony (poza przypadkiem kiedy testowany kod wyrzuci wyjątek). Ktoś może napisać test bez asercji, aby sprawdzić czy jego kod nie wyrzuci wyjątku. Framework do testowania powinien mieć oddzielną metodę w klasie Assert do sprawdzania czy wyjątek został lub nie został wyrzucony przez zadany kod.
*   **Rostrzygalność**: Oprócz stanu czerwony i zielony, testy mogą mieć też stan nierozstrzygnięty (ang. _inconclusive_). Taki stan oznacza, że nie udało się rozstrzygnąć czy dana asercja jest spełniona lub nie.
*   **Zasada pojedynczej odpowiedzialności**: Jeden test jednostkowy powinien testować jedną logiczną asercję (lub inaczej: jedno zachowanie). Oznacza to, że nie umieszczamy wielu asercji testujących różne zachowania w jednym teście jednostkowym. Musimy podzielić taki test na n metod (n - liczba asercji) lub wprowadzić testy sparametryzowane. Ta zasada nie oznacza, że jeden test powinien mieć tylko jeden Assert. Wiele wywołań Assert może przecież testować jedną logicznę asercję. Przykładem użycia wielu metod klasy Assert w obrębie jednej logicznej asercji może być asercja sprawdzająca poprawność elementów kolekcji.
*   **Niezależność**: Wszystkie zależności wewnętrzne (np. zależne klasy, interfejsy) oraz zewnętrzne (np. baza danych, system, sieć wewnętrzna, Internet, web service) powinny być zastąpione przez _test doubles_ (_stuby, fake-i, mocki,_ itd.) Dzięki niezależności zyskujemy nie tylko oddzielenie się od implementacji konkretnej zależności, ale także szybkość i reużywalność testu.
*   _Second Class Citizens_: Kod testowy nie jest kodem drugiej kategorii. Należy o niego dbać, aktualizować, refaktoryzować, robić review w taki sam sposób jak kod produkcyjny.

# Dobre i złe praktyki pisania testów jednostkowych

W tym paragrafie chciałbym przedstawić dobre i złe wzorce pisania testów jednostkowych. Podobnie jak z pewnymi kontrowersjami wokół dobrych i złych zasad programowania (np. singleton), nie każdy może się z nimi zgodzić. Wiele osób może mieć inne zdanie co do tego czy np. testować zmienne prywatne. Postarałem się tutaj wrzucić worek zasad, które z mojego punktu widzenia są dobre lub złe.

*   **Arrange-Act-Assert**: Kod powinien być ustrykturyzowany wg zasady AAA. Czytaj: [Kurs TDD część 3: Struktura testu, czyli Arrange-Act-Assert](/posts/kurs-tdd-3-struktura-test-czyli-arrange-act-assert/ "Kurs TDD część 3: Struktura testu, czyli Arrange-Act-Assert").
*   **Podział assembly**: Każde assembly powinno zawierać osobne typy testów i być nazwane wg konwencji: `Tests.Unit`, `Tests.Integration`, `Tests.Acceptance`, itd. Jest kilka powodów dla których powinniśmy wprowadzić taki podział:
    *   Po pierwsze, powinniśmy zawsze i bez dodatkowej analizy wiedzieć czy potrzebujemy odpowiedniej konfiguracji aby uruchomić test oraz czy musimy mieć zapewniony dostęp do zewnętrznych zależności.
    *   Musimy też jednoznacznie stwierdzić, czy test czerwony wynika z braku konfiguracji, braku dostępu do zależności, błędu logiki w kodzie produkcyjnym, z błędu w kodzie testowym lub nieaktualnym kodzie testowym. Podział zawęża nam tę grupę poszukiwania.
    *   Ponadto czerwony test jednostkowy ma priorytet dużo większy niż test czerwony integracyjny.
    *   Podział testów ułatwia nam dodatkowo konfigurację builda i pozwala na szybkie dołączenie odpowiedniej kategorii testów.
*   **Nazewnictwo**: Najważniejszą regułą w zakresie nazewnictwa testów jest stosowanie takiej nazwy, która pozwoli określić jednoznacznie co testujemy i wskazać jakie dane wejściowe zostały wprowadzone. Jest kilka konwencji nazewnictwa testów jednostkowych, a jej wybór to preferencja osobista lub projektowa. Ja korzystam najczęściej z konwencji [\[UnitOfWork\_StateUnderTest\_ExpectedBehavior\]](http://osherove.com/blog/2005/4/3/naming-standards-for-unit-tests.html). Pamiętać należy o tym, że to nie wybór konwencji, a czytelność nazw naszych testów jest najważniejsza.
*   **Testowanie prywatnych składowych**: Nie powinno się testować prywatnych składowych klasy (private, internal); powinniśmy testować jedynie publiczne API klas. Nie interesuje nas ich wewnętrzna implementacja, wobec czego nie powinniśmy testować jej prywatnych składowych.
*   **Testy zawierające konfigurację**: Testy jednostkowe nie powinny mieć żadnej konfiguracji.
*   **Testy korzystające z konsoli systemowej**: Testy nie powinny zawierać odwołań do konsoli systemowej. Niektórzy używają Console.Writeline w celu sprawdzenia czegoś ręcznie, jest to jednak oczywisty antywzorzec.
*   **Łapanie wszystkich wyjątków**: Łapanie wszystkich wyjątków i objęcie w try-catch może spowodować niewyłapanie błędu w logice testowanego kodu. Do asercji związanych z oczekiwaniem wyjątku służą oddzielne metody klasy Assert.
*   **Oczekiwanie typu Exception**: Jeśli spodziewamy się innego wyjątku niż typu Exception, powinniśmy sprecyzować typ oczekiwanego wyjątku.
*   **Oczekiwanie na wyjątek w niewłaściwym miejscu**: Powinniśmy sprecyzować, która część kodu wyrzuci nam wyjątek. Przykładem złej praktyki jest stosowanie atrybutu \[ExpectedException\] (NUnit). Założenie takiego atrybutu skutkuje, że oczekujemy na wyjątek w całym teście. Powinniśmy zastosować metodę Throws klasy Assert, która odnosi się do konkretnego kodu, który ma rzucić wyjątek.
*   **Stare testy**: ...powinny być usunięte**.**
*   **Testowanie tylko ścieżki optymistycznej (_happy path_)**: Należy pamiętać, że powinniśmy zawsze testować nie tylko przypadki ze ścieżki optymistycznej, ale też wszystkie przypadki brzegowe, przypadki z null-ami oraz wyjątki.
*   **Instrukcje warunkowe w teście**: Test jednostkowy nie powinien zawierać instrukcji warunkowych (_if, switch_). Wiąże się to z zasadą pojedynczej odpowiedzialności testu. Test z instrukcją warunkową powinniśmy podzielić na tyle testów, ile istnieje bloków warunkowych.
*   **Stosowanie pętli w teście**: Test jednostkowy nie powinien zawierać pętli (_for, foreach, while_). Wprowadzenie logiki powoduje wzrost ryzyka powstania błędu w naszym teście jednostkowym. Taki test będzie też trudniejszy w zrozumieniu. Z kodu testowego usuwamy pętle i zastępujemy go zapisem/odwołaniem ręcznym, co spowoduje najczęściej większą ilością linii kodu, ale na korzyść łatwiejszego zrozumienia, lepszej czytelności i mniejszej szansy na zaistnienie błędu w teście.

# Podsumowanie

Wymieniłem większość znanych wzorców i antywzorców TDD. Największą bolączką TDD jest jednak traktowanie kodu testowego jako gracza drugiej kategorii. Jak już zostało wspomniane przy okazji punktu "Second Class Citizen", testy powinny być zarządzane i traktowane z taką samą dbałością jak kod produkcyjny.

# PS. TDD FIRST

Warto nadmienić, że w kontekście TDD istnieje zasada FIRST, która mówi że testy jednostkowe powinny być:

1.  _Fast —_ szybkie
2.  _Independent —_ niezależne od innych testów
3.  _Repeatable —_ powtarzalne (uruchomone N razy, zawsze zwrócą te same rezultaty)
4.  _Self-checking —_ stwierdzające czy test przeszedł lub nie (brak ręcznej interpretacji)
5.  _Timely —_ pisane razem z testem produkcyjnym

# PPS. Linki

*   [James Carr: TDD Anti-Patterns](http://blog.james-carr.org/2006/11/03/tdd-anti-patterns/)
*   [StackOverflow: TDD Anti-patterns catalogue](http://stackoverflow.com/questions/333682/tdd-anti-patterns-catalogue)
*   [JUnit Anti-patterns](http://www.exubero.com/junit/antipatterns.html)
*   [Clean TDD Sheet 1.2 \[PDF\]](http://www.planetgeek.ch/wp-content/uploads/2011/02/Clean-TDD-Cheat-Sheet-V1.2.pdf)

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
6. Dobre i złe praktyki testów jednostkowych
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