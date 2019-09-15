---
title: Automatyczne testowanie guard clauses w konstruktorach
redirect_from:
    -
date: "2019-09-14T13:00:00.000Z"
layout: post
draft: false
path: "/posts/automatyczne-testowanie-guard-clauses"
category: "Programowanie"
tags:
  - "TDD"
  - "C#"
  - "Auto-Mocking"
  - "AutoFixture"
  - "Moq"
description: "W testach bardzo często mamy do czynienia z powtarzaniem tego samego kodu. Tak samo ma się sprawa z testowaniem _null-checków_ (zw. inaczej jako _guard clauses_) w konstruktorach. Jeśli chcemy napisać test dla takiego przypadku, to zazwyczaj jest on napisany jako osobna metoda. Jak możemy sobie uprościć życie? Z pomocą przychodzi biblioteka AutoFixture.Idioms."
---

W testach bardzo często mamy do czynienia z powtarzaniem tego samego kodu. Tak samo ma się sprawa z testowaniem _null-checków_ (zw. inaczej jako _guard clauses_) w konstruktorach. Jeśli chcemy napisać test dla takiego przypadku, to zazwyczaj jest on napisany jako osobna metoda. Jak możemy sobie uprościć życie? Z pomocą przychodzi biblioteka AutoFixture.Idioms.

Aby móc skorzystać z funkcji do automatycznego testowania _null-checków_ w konstruktorach, potrzebujemy zainstalować dwa nugety:

- AutoFixture.Idioms
- AutoFixture.AutoMoq

W przykładowym kodzie użyjemy:

1. NUnitowego `TestCaseSource` do definicji wszystkich typów klas, których konstruktory chcemy sprawdzić.
2. Klasy `GuardClauseAssertion`, która pomoże nam w tytułowym rozwiązaniu problemu.
3. Klasy `AutoMoqCustomization`, której celem jest automockowanie parametrów konstruktora.

Kod testowy wygląda następująco:

```csharp
public class Tests
{
    public static IEnumerable<Type> CtorsWithGuardClauses
    {
        get
        {
            return new List<Type>
            {
                typeof(SomeClass),
                // add more guys here...
            };
        }
    }

    [TestCaseSource(nameof(CtorsWithGuardClauses))]
    public void AllConstructorsMustBeGuardClaused(Type type)
    {
        var fixture = new Fixture().Customize(new AutoMoqCustomization());

        var assertion = new GuardClauseAssertion(fixture);

        assertion.Verify(type.GetConstructors());
    }
}
```

Gdzie klasa `SomeClass` to:

<sup>(dla poniższego kodu powyższy test będzie zielony ✅)</sup>

```csharp
public class SomeClass
{
    private readonly I1 _i1;
    private readonly I2 _i2;
    private readonly I3 _i3;
    private readonly int? _nullablei;
    private readonly string _s;

    public SomeClass(I1 i1, I2 i2, I3 i3)
    {
        _i1 = i1 ?? throw new ArgumentNullException(nameof(i1));
        _i2 = i2 ?? throw new ArgumentNullException(nameof(i2));
        _i3 = i3 ?? throw new ArgumentNullException(nameof(i3));
    }

    public SomeClass(I1 i1)
    {
        _i1 = i1 ?? throw new ArgumentNullException(nameof(i1));
    }
}

public interface I1 { }
public interface I2 { }
public interface I3 { }
```

Kilka uwag:
- Analizowane są wszystkie publiczne konstruktory.
- W analizie nie są brane _null-checki_ dla typów `Nullable<>`.

Jeśli zabraknie nam _null-checków_, to test wyrzuci bardzo jasny komunikat:

```
AutoFixture.Idioms.GuardClauseException : An attempt was made to assign the value null to the parameter "i1" of the method ".ctor", and no Guard Clause prevented this. Are you missing a Guard Clause?
```

Jak widać, możemy sobie w bardzo prosty zautomatyzować testy konstruktorów i definiować wszystkie klasy z konstruktorami wymagającymi _null-checki_ w jednym miejscu.

Pytanie — Czy aby na pewno potrzebujemy testować _null-checki_ w konstruktorach? Tak! _Guard clause_ mają na celu tzw. _fail fast_, czyli zgłosić błąd jak najszybciej. Jeśli parametr konstruktora będzie nullem, a kod będzie wymagać tego parametru, to dzięki defensywnemu podejściu, dostaniemy szybciej informację zwrotną o błędzie. Późniejszy błąd może mieć również skutki uboczne, jeśli na przykład przed wystąpieniem błędu zmutowano stan obiektu. A zatem, skoro sam kod jest potrzebny, testy tym bardziej!

Kod źródłowy do przykładu znajduje się na GitHubie: https://github.com/dariusz-wozniak/AutoTestGuardClausesDemo
