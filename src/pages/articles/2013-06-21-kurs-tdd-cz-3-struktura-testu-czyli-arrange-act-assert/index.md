---
title: Kurs TDD cz. 3 — Struktura testu, czyli Arrange-Act-Assert
date: "2013-06-21T19:59:27.000Z"
layout: post
draft: false
path: "/posts/kurs-tdd-3-struktura-test-czyli-arrange-act-assert"
category: "Programowanie"
tags:
  - "TDD"
  - "Kurs TDD"
description: "Czas na omówienie jak, strukturalnie, powinien wyglądać wzorcowy test jednostkowy (Arrange-Act-Assert)."
---

Po [wstępie do TDD](/posts/kurs-tdd-1-wstep/ "Kurs TDD część 1: Wstęp") i [omówieniu różnic między testami jednostkowymi, a integracyjnymi](/posts/kurs-tdd-2-testy-jednostkowe-a-testy-integracyjne/ "Kurs TDD część 2: Testy jednostkowe, a testy integracyjne"), czas na omówienie jak strukturalnie powinien wyglądać wzorcowy test jednostkowy. Będzie to pewnie najkrótszy wpis tej serii, ale jednocześnie jeden z najważniejszych. Pozwoli bowiem na pierwszy kontakt z testem jednostkowym w praktyce.

# Arrange–Act–Assert

Strukturę testu jednostkowego definiuje zasada Arrange–Act–Assert (AAA):

*   **Arrange**: wszystkie dane wejściowe i _preconditions,_
*   **Act**: działanie na metodzie/funkcji/klasie testowanej,
*   **Assert**: upewnienie się, że zwrócone wartości są zgodne z oczekiwanymi.

Jakie korzyści płyną ze stosowania tego wzorca? Przede wszystkim porządek; wzorzec zapewnia logiczny porządek w pojedynczym teście — część przygotowania danych wejściowych jest odseparowana od części weryfikacyjnej. Ponadto, nie mieszamy naszych asercji w trakcie wywołania testowanego obiektu.

## Przykład

Jak mawiali Chińczycy—Jeden kod mówi więcej niż tysiąc słów. Poniższy kod napisany jest zgodnie z AAA: 
```csharp
[Test]
public void Add_AddingTwoValues_ReturnsProperValue()
{
    // Arrange:
    var calc = new Calculator();
 
    // Act:
    int result = calc.Add(2, 3);
 
    // Assert:
    Assert.AreEqual(5, result);
}
```
 Na powyższym, bardzo prostym, przykładzie testujemy funkcjonalność dodawania klasy `Calculator`. Chcemy sprawdzić czy (niebrzegowy) przypadek dodania dwóch liczb zwróci oczekiwany wynik.
 
 W pierwszej części metody inicjalizujemy obiekt klasy Calculator. Wszelkie inicjalizacje zmiennych, obiektów oraz _mocków_ (o czym będzie w późniejszych częściach) znajdować powinny się w pierwszej części metody — **Arrange**.
 
 W kolejnej linijce wykonujemy proste działanie arytmetyczne: dodajemy do siebie dwie liczby całkowite – 2 i 3. Celem tego etapu, **Act**, jest uruchomienie funkcjonalności testowanej klasy. Będzie to zazwyczaj uruchomienie metody, choć nie jest to reguła (zawsze możemy przetestować np. odczyt/zapis danego property).
 
 Ostatnia linijka to wykonanie asercji (**Assert**). Sprawdzamy, czy zwrócony wynik z powyższego wywołania jest taki sam jak oczekujemy. Tutaj oczekujemy, że metoda naszej klasy po dodaniu liczb 2 i 3, zwróci nam liczbę 5.
 
 I jeszcze kilka kwestii technicznych. Atrybut `[Test]` i klasa `Assert` pochodzą z NUnita, który należy do rodziny frameworków testowych. Kolejność parametrów metody `AreEqual` jest następująca: pierwszym parametrem jest wartość oczekiwana, drugim zwrócona przez działanie naszej metody. W kolejnej części cyklu dowiemy się jak, krok po kroku, stworzyć pierwszy test jednostkowy. Zrozumienie wzorca Arrange-Act-Assert jest niezbędne do budowania dobrych testów.

# Zobacz też

*   [http://c2.com/cgi/wiki?ArrangeActAssert](http://c2.com/cgi/wiki?ArrangeActAssert "http://c2.com/cgi/wiki?ArrangeActAssert")


<!-- tdd-course-infobox-start -->
<div class="boxBorder">

<div style="text-align: center; font-size: 40px">Kurs TDD</div>

----

<div class="row">
<div class="column">

Część I: Testy jednostkowe – wstęp

1. [Wstęp do TDD](/posts/kurs-tdd-1-wstep/)
2. [Testy jednostkowe, a integracyjne](/posts/kurs-tdd-2-testy-jednostkowe-a-testy-integracyjne/)
3. Struktura testu, czyli Arrange-Act-Assert
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