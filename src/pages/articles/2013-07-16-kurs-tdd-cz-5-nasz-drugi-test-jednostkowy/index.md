---
title: Kurs TDD cz. 5 — Nasz drugi test jednostkowy
date: "2013-07-16T17:46:48.000Z"
layout: post
draft: false
path: "/posts/kurs-tdd-5-nasz-drugi-test-jednostkowy"
category: "Programowanie"
tags:
  - "TDD"
  - "Kurs TDD"
  - "C#"
  - "NUnit"
description: "W tej części omówię jak wykonać kilka prostych technik, tj. jak zgrupować testy za pomocą atrybutu [TestCase], testować wyjątki, testować zdarzenia."
---

[W poprzedniej części kursu](/posts/kurs-tdd-4-nasz-pierwszy-test-jednostkowy) "Kurs TDD część 4: Nasz pierwszy test jednostkowy") omówiłem w jaki sposób ustawić środowisko Visual Studio aby móc pisać i uruchamiać testy. W tej części omówię jak wykonać kilka prostych technik, tj. jak:

*   zgrupować testy za pomocą atrybutu `[TestCase]`,
*   testować wyjątki,
*   testować zdarzenia.

Na tapetę idzie przykład dzielenia; chcemy napisać funkcjonalność i testy mając na uwadze, że:

*   metoda `Divide` należy do klasy `Calculator`,
*   metoda `Divide` przyjmuje dwa parametry wejściowe — obydwa typu int; zwracanym typem jest float,
*   po skończonym obliczeniu wywoływane jest zdarzenie `CalculatedEvent`,
*   w przypadku, gdy dzielnik jest równy 0, wyrzucamy wyjątek typu `DivideByZeroException`.

I tyle! Początkowy szkielet klasy Calculator wyglądać będzie tak:

```csharp
class Calculator
{
    public float Divide(int dividend, int divisor)
    {
        throw new NotImplementedException();
    }
 
    public event EventHandler CalculatedEvent;
 
    protected virtual void OnCalculated()
    {
        var handler = CalculatedEvent;
        if (handler != null) handler(this, EventArgs.Empty);
    }
}
```

# Test przypadków niebrzegowych i brzegowych

Pamiętając, że na wejściu mamy dwa inty, a na wyjściu float możemy rozpisać tablicę przypadków, dla których chcemy sprawdzić poprawność naszego wyniku. Mogą to być następujące testy:

*   dzielenie liczb dodatnich, zwracany typ jest liczbą całkowitą, np. 4 ÷ 2 = 2,
*   dzielenie liczby dodatniej przez ujemną i odwrotnie, zwracany typ jest liczbą całkowitą, np. -4 ÷ 2 = -2 i 4 ÷ (-2) = -2,
*   dzielenie zera przez dowolną liczbę, np. 0 ÷ 3 = 0,
*   zwracany typ jest ułamkiem skończonym, np. 5 ÷ 2 = 2,5,
*   zwracany typ jest ułamkiem nieskończonym lub zaokrąglonym, np. 1 ÷ 3 = 0,333333343f.<sup>[1]</sup>

Aby uniknąć pisania sześciu metod (można, ale da się to zrobić lepiej) czy też pisania wszystkich asercji w jednym teście ([co, jak już wiemy, jest złym wzorcem](/posts/kurs-tdd-4-nasz-pierwszy-test-jednostkowy/ "Kurs TDD część 4: Nasz pierwszy test jednostkowy")) możemy skorzystać z NUnitowego atrybutu [`[TestCase]`](https://github.com/nunit/docs/wiki/TestCase-Attribute). Jako parametry atrybutu podajemy dane wejściowe oraz (opcjonalnie) wartość oczekiwaną, natomiast definicja poszczególnych elementów zawarta jest w parametrach testu jednostkowego. Nasz test będzie wyglądać tak: 

```csharp
[TestCase(4, 2, 2.0f)]
[TestCase(-4, 2, -2.0f)]
[TestCase(4, -2, -2.0f)]
[TestCase(0, 3, 0.0f)]
[TestCase(5, 2, 2.5f)]
[TestCase(1, 3, 0.333333343f)]
public void Divide_ReturnsProperValue(int dividend, int divisor, float expectedQuotient)
{
    var calc = new Calculator();
    var quotient = calc.Divide(dividend, divisor);
    Assert.AreEqual(expectedQuotient, quotient);
}
```

 Dzięki atrybutowi `[TestCase]` mamy 6 testów w jednej metodzie. W okienku rezultatów testu, wszystkie pojawiają się jako podrzędne do głównego testu. Wygląda to tak:
 
 ![testcase](12e4f69b-0dc1-4146-b925-39f0a0071525.png)
 
 Bardzo ważna kwestia jaka tutaj się pojawiła to przypadek dzielenia 1 ÷ 3. Dzięki arytmetyce liczb zmiennoprzecinkowych uzyskamy wynik 0.3333333**4**3f. Wyjaśnienie skąd się wział taki wynik znajduje się w literaturze zamieszczonej w przypisach. Najważniejsza jest jednak świadomość tego faktu i uwzględnienie go w testach.

# Testowanie wyrzucenia wyjątku

W przypadku dzielenia musimy obsłużyć przypadek dzielenia przez zero. Założyliśmy, że w takim przypadku wyrzucamy błąd typu `System.DivideByZeroException`. W NUnicie testowanie wyjątków możemy wykonać przez wywołanie [`[Assert.Throws`]](https://github.com/nunit/docs/wiki/Assert.Throws). Tutaj również podanie typu jest opcjonalne. Jako parametr przekazujemy delegat kodu, który chcemy wykonać.

```csharp
[Test]
public void Divide_DivisionByZero_ThrowsException()
{
    var calc = new Calculator();
    Assert.Throws<DivideByZeroException>(() => calc.Divide(2, 0));
}
```
 Wyrażenie lambda możemy zastąpić anonimową metodą: 
```csharp
Assert.Throws(delegate { calc.Divide(2, 0); });
```

# Testowanie zdarzenia

Założyliśmy, że po wykonaniu obliczeń, wołamy zdarzenie `CalculatedEvent`. Sam NUnit nie wspiera natywnie testowania zdarzeń, jednak możemy zastosować prosty trik—po wywołaniu zdarzenia zmieniamy wartość flagi. Asercji dokonujemy na podstawie wartości tej flagi. Jeśli zdarzenie zostało wywołane, test przechodzi pozytywnie: 
```csharp
[Test]
public void Divide_OnCalculatedEventIsCalled()
{
    var calc = new Calculator();
 
    bool wasEventCalled = false;
    calc.CalculatedEvent += (sender, args) => wasEventCalled = true;
 
    calc.Divide(1, 2);
 
    Assert.IsTrue(wasEventCalled);
}
```
 Wyrażenie lambda możemy zastąpić anonimową metodą: 

```csharp
calc.CalculatedEvent += delegate { wasEventCalled = true; };
```
# Implementacja

Po napisaniu testów do naszego kodu, możemy przystąpić do napisania implementacji metody dzielenia. Zachęcam do napisania implementacji we własnym zakresie!

Ostateczna postać klasy wygląda tak: 
```csharp
class Calculator
{
    public float Divide(int dividend, int divisor)
    {
        if (divisor == 0) throw new DivideByZeroException();
 
        float result = (float)dividend / divisor;
        OnCalculated();
        return result;
    }
 
    public event EventHandler CalculatedEvent;
 
    protected virtual void OnCalculated()
    {
        var handler = CalculatedEvent;
        if (handler != null) handler(this, EventArgs.Empty);
    }
}
```
 Wszystkie testy są zielone. W tak prostym przykładzie nie trzeba nic refaktoryzować! _Fin!_

# Podsumowanie

W tej części kursu poznaliśmy:

*   Przydatność atrybutu `[TestCase]`, który niewielkim kosztem generuje przypadek testowy.
*   Sposoby testowania wyjątków za pomocą NUnit.
*   Sposób testowania zdarzeń.

Ponadto dowiedliśmy że _float_, ze względu na arytmetykę liczb zmiennoprzecinkowych, nie jest odpowiednim typem jako typ zwracany przy dzieleniu. Lepszym okazałby się _decimal_. Wybrałem jednak _float_, aby pokazać naturę testów. Oczekujemy nie do końca prawidłowej (z punktu widzenia matematycznego) wartości (przypadek dzielenia 1 ÷ 3) i dzięki temu pojawienie się oczekiwanego wyniku nie powinno nas zaskoczyć. Dzięki TDD wykrylibyśmy taki błąd przy zmianie typu zwracanego: np. z _decimal_ na _float_. Dobranie typu parametrów wejściowych jako _int_ też jest celowe, choć w praktyce bardzo niebezpieczne. Na szczególną uwagę zasługuje linijka: 

```csharp
float result = (float)dividend / divisor;
```
 Jaki wynik otrzymalibyśmy bez rzutowania zmiennej dividend na float? Zachęcam do eksperymentowania.

# Przypisy

<sup>[1]</sup> Dlaczego 1 ÷ 3 = 0,333333343f? Czytaj więcej na ten temat:

*   [C# 5.0 in a Nutshell](http://books.google.pl/books?id=t1de8nSVYnkC&pg=PA234&lpg=PA234&dq=0.333333343+float+c%23&source=bl&ots=24tgZxIm6z&sig=U_te6Rc-o5mnNukFVtz5gkv_45o&hl=en&sa=X&ei=y3blUbPuD4HUtQbi34Eg&redir_esc=y#v=onepage&q=0.333333343%20float%20c%23&f=false)
*   [C# in Depth: Binary floating point and .NET](http://csharpindepth.com/Articles/General/FloatingPoint.aspx)
*   [Stack Overflow: Precision issues with Visual Studio 2010](http://stackoverflow.com/questions/8520160/precision-issues-with-visual-studio-2010)

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
5. Nasz drugi test jednostkowy
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