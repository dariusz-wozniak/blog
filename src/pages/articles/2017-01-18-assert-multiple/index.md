---
title: NUnit — Assert.Multiple
date: "2017-01-18T19:37:25.000Z"
layout: post
draft: false
path: "/posts/assert-multiple/"
category: "Programowanie"
tags:
  - "TDD"
  - "C#"
  - "NUnit"
description: "NUnit w wersji 3.6 wprowadził dość ciekawą funkcję — Assert.Multiple. Dzięki niej, dostajemy informacje na temat wszystkich testów, które nie przeszły. Przyjrzyjmy się temu bliżej… Do tej pory, używając kilku asercji naraz, jedna niespełniona asercja powodowała, że dalszy kod nie jest wykonywany."
---

NUnit w wersji 3.6 wprowadził dość ciekawą funkcję — `Assert.Multiple`. Dzięki niej, dostajemy informacje na temat wszystkich testów, które nie przeszły. Przyjrzyjmy się temu bliżej… Do tej pory, używając kilku asercji naraz, jedna niespełniona asercja powodowała, że dalszy kod nie jest wykonywany. Na przykład:

```csharp
Assert.That(2 + 2, Is.EqualTo(4));
Assert.That(2 + 2, Is.EqualTo(5));
Assert.That(2 + 2, Is.EqualTo(6));
```

Dla powyższego kodu:

*   Pierwsza asercja jest spełniona, idziemy dalej.
*   Druga asercja nie jest spełniona, w związku z czym dalsze wykonanie kodu jest przerwane.
*   Nie wiemy czy trzecia asercja jest spełniona lub nie.

`Assert.Multiple` działa następująco:

*   W przypadku czerwonego testu, dalsze asercje wykonywane są nadal.
*   W komunikacie błędu dostajemy informacje nt. wszystkich niespełnionych asercji.

Dla kodu

```csharp
 Assert.Multiple(() =>
 {
     Assert.That(2 + 2, Is.EqualTo(4));
     Assert.That(2 + 2, Is.EqualTo(5));
     Assert.That(2 + 2, Is.EqualTo(6));
});
```

Otrzymamy komunikat:

```
One or more failures in Multiple Assert block:
1) Expected: 5
But was: 4
2) Expected: 6
But was: 4
```

Czy to jest nowość w świecie frameworków do testowania? Nie. Analogiczną funkcjonalność w Świętej Pamięci MbUnicie można zastosować przez nałożenie atrybutu MultipleAsserts. **Kiedy stosować?**

*   Dla testów sparametryzowanych, w dalszym ciągu pozostałbym przy [dotychczasowych rozwiązaniach (np. `TestCase`)](/posts/kurs-tdd-8-testy-parametryzowane).
*   Jedna z zasad testowania jednostkowego, mówi nam aby stosować jedną asercję (logiczną) per test. Przykładowo, przy zwracaniu złożonego obiektu, możemy posłużyć się wieloma asercjami, aby poznać które asercje są spełnione, a które nie.
*   `Assert.Multiple` nadaje się też do testów niejednostkowych, np. integracyjnych, akceptacyjnych, itp., tam gdzie to rozwiązanie daje nam pełniejszy obraz danego testu.

Po więcej, odsyłam do oficjalnej dokumentacji: https://github.com/nunit/docs/wiki/Multiple-Asserts.