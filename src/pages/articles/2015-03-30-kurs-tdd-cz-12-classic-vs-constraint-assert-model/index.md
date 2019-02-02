---
title: Kurs TDD cz. 12 — Classic vs. Constraint Assert Model
date: "2015-03-30T15:00:48.000Z"
layout: post
draft: false
path: "/posts/kurs-tdd-12-classic-vs-constraint-assert-model"
category: "Programowanie"
tags:
  - "TDD"
  - "Kurs TDD"
  - "C#"
  - "NUnit"
description: "Rzecz być może dla niektórych mało istotna, dla niektórych w ogóle nie istotna, ale niezależnie od istotności sprawy – myślę, że warta wpisu na blogu. NUnit posiada dwa modele asercji: Classic Assert Model oraz Constraint-Based Assert Model."
---

Rzecz być może dla niektórych mało istotna, dla niektórych w ogóle nie istotna, ale niezależnie od istotności sprawy – myślę, że warta wpisu na blogu. NUnit posiada dwa modele asercji:

*   Classic Assert Model
*   Constraint-Based Assert Model

Semantyka klasycznego modelu jest wszystkim dobrze znana:

```csharp
Assert.AreEqual(5, sum);
Assert.AreSame(personA, personB);
Assert.IsTrue(condition);
Assert.IsNotNull(model);
Assert.IsEmpty(phoneNumbers);
Assert.GreaterOrEqual(7, result);
Assert.IsInstanceOfType(typeof(IList), phoneNumbers);
Assert.Throws<DivideByZeroException>(() => calc.Divide(2, 0));
StringAssert.Contains("abc", text);
CollectionAssert.IsEmpty(person.PhoneNumbers);
```

 Co nowego wprowadza ten drugi? Wszystkie asercje Constraint-Based Assert Model wywoływane są z metody That klasy Assert, która to metoda przyjmuje co najmniej dwa parametry: 

```csharp
Assert.That(object actual, IConstraint constraint);
```

 `IConstraint` jest interfejsem na podstawie którego budujemy nasze wyrażenia. NUnit idzie nam z pomocą i posiada pakiet niezbędnych `IConstraints` w formie Syntax Helpers - między innymi klas `Is` i `Has`. Oto dwie asercje opisane różnymi modelami:

```csharp
Assert.AreEqual(5, sum); // Classic Assert Model
Assert.That(sum, Is.EqualTo(5)); // Constraint-Based Assert Model
```

 Przykłady asercji Constraint-Based Assert Model: 

```csharp
Assert.That(text, Is.EqualTo("Hello"));
Assert.That(exception1, Is.SameAs(exception2));
Assert.That(exception1, Is.Not.SameAs(exception2));
Assert.That(nObject, Is.Null);
Assert.That(anObject, Is.Not.Null);
Assert.That(condition, Is.True);
Assert.That(condition, Is.False);
Assert.That(aDouble, Is.NaN);
Assert.That(aDouble, Is.Not.NaN);
Assert.That(aString, Is.Empty);
Assert.That(collection, Is.Empty);
Assert.That(collection, Is.Unique);
Assert.That(7, Is.GreaterThan(3));
Assert.That(7, Is.GreaterThanOrEqualTo(3));
Assert.That(7, Is.AtLeast(3));
Assert.That(7, Is.GreaterThanOrEqualTo(7));
Assert.That(7, Is.AtLeast(7));
Assert.That(3, Is.LessThan(7));
Assert.That(3, Is.LessThanOrEqualTo(7));
Assert.That(3, Is.AtMost(7));
Assert.That(3, Is.LessThanOrEqualTo(3));
Assert.That(3, Is.AtMost(3));
Assert.That("Hello", Is.TypeOf(typeof(string)));
Assert.That("Hello", Is.Not.TypeOf(typeof(int)));
Assert.That(phrase, Text.Contains("tests fail"));
Assert.That(phrase, Text.Contains("make").IgnoreCase);
Assert.That(array, Is.All.Not.Null);
Assert.That(array, Is.All.InstanceOfType(typeof(string)));
Assert.That(array, Is.All.GreaterThan(0));
Assert.That(array, Is.Unique);
Assert.That(iarray, Has.None.Null);
Assert.That(iarray, Has.No.Null);
Assert.That(sarray, Has.None.EqualTo("d"));
Assert.That(iarray, Has.None.LessThan(0));
Assert.That(iarray, Has.Member(3));
Assert.That(sarray, Has.Member("b"));
Assert.That(sarray, Has.No.Member("x"));
Assert.That(iarray, Is.Ordered);
Assert.That(sarray, Is.Ordered.Descending);
Assert.That(SomeMethod, Throws.TypeOf<ArgumentException>());
```

 Argumenty podane dla IConstraint możemy składać w operacje logiczne. Służą do tego syntax helpers `Is.Not`, `Is.All` oraz operatory `&` i `|`:

```csharp
Assert.That(2 + 2, Is.Not.EqualTo(5);
Assert.That(new int[] { 1, 2, 3 }, Is.All.GreaterThan(0));
Assert.That(2.3, Is.GreaterThan(2.0) & Is.LessThan(3.0));
Assert.That(3, Is.LessThan(5) | Is.GreaterThan(10));
```

 Możemy też tworzyć swoje własne Constraints. Aby to zrobić, musimy zaimplementować klasę abstrakcyjną `Constraint`: 

```csharp
public abstract class Constraint
{
    public abstract bool Matches(object actual);
    public virtual bool Matches(ActualValueDelegate del);
    public virtual bool Matches<T>(ref T actual);
    public abstract void WriteDescriptionTo(MessageWriter writer);
    public virtual void WriteMessageTo(MessageWriter writer);
    public virtual void WriteActualValueTo(MessageWriter writer);
}
```

 Który z tych modeli jest "lepszy" albo bardziej czytelny? Jest to oczywiście kwestia gustu.
 
 Ciekawostka: Na podstawie Constraint-Based Assert Model powstała biblioteka C++ o nazwie [Snowhouse](https://github.com/joakimkarlsson/snowhouse "https://github.com/joakimkarlsson/snowhouse").

 ## Źródła

*   [http://nunit.org/index.php?p=classicModel&r=2.6.4](http://nunit.org/index.php?p=classicModel&r=2.6.4 "http://nunit.org/index.php?p=classicModel&r=2.6.4")
*   [http://nunit.com/index.php?p=constraintModel&r=2.5](http://nunit.com/index.php?p=constraintModel&r=2.5 "http://nunit.com/index.php?p=constraintModel&r=2.5")
*   [http://nunit.com/index.php?p=customConstraints&r=2.5](http://nunit.com/index.php?p=customConstraints&r=2.5 "http://nunit.com/index.php?p=customConstraints&r=2.5")

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
12. Classic vs. Constraint Assert Model
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