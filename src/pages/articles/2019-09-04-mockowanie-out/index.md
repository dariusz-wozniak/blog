---
title: Mockowanie out
redirect_from:
    -
date: "2019-09-04T21:01:00.000Z"
layout: post
draft: false
path: "/posts/mockowanie-out"
category: "Programowanie"
tags:
  - "TDD"
  - "C#"
  - "Moq"
description: "W jaki sposób mockować metody ze słowem kluczowym out?"
---

W jaki sposób mockować (za pomocą Moq) metody ze słowem kluczowym out? Przyjrzyjmy się bliżej problemowi...

Przypuśćmy, że posiadamy klasę, która parsuje wartość typu string na interfejs numeryczny o nazwie `INumber`. Metoda parsowania to wzorzec Try-Parse, niemal identyczny do metod typu `int.TryParse`. Obydwa interfejsy wyglądają następująco:

```csharp
public interface INumberParser
{
    bool TryParse(string numberToBeParsed, out INumber number);
}

public interface INumber
{
    int Int32 { get; set; }
}
```

Aby zamockować metodę `TryParse` klasy `INumberConverter` należy posłużyć się następującym przepisem:

1. Przy setupie mocka musimy zdefiniować odpowiedni matcher dla parametru `number`. Zwykły matcher `It.IsAny` w tym przypadku nie zadziała. Musimy posłużyć się matcherem `It.Ref<>.IsAny`.
1. Jak zwrócić wartość typu bool metody oraz przypisać wartość zmiennej out? Tutaj musimy zdefiniować odpowiedni delegat tego samego typu co metoda `TryParse`.

Cały test oraz wspomniany delegat wyglądają następująco:

```csharp
[Test]
public void Test()
{
    // Setup the mocking of out:
    var numberConverter = Mock.Of<INumberParser>();

    Mock.Get(numberConverter)
        .Setup(x => x.TryParse(It.IsAny<string>(), out It.Ref<INumber>.IsAny))
        .Returns(new TryParse((string s, out INumber n) =>
        {
            n = Mock.Of<INumber>(num => num.Int32 == 456);
            return true;
        }));
        
    // Act:
    var tryGet = numberConverter.TryParse("any number", out var number);

    // Assert that values are properly set:
    Assert.That(tryGet, Is.True);
    Assert.That(number.Int32, Is.EqualTo(456));
}

private delegate bool TryParse(string s, out INumber n);
```

Drugi sposób to utworzenie nowego mocka i przypisanie wartości w metodzie `Callback`:

```csharp
[Test]
public void Test2()
{
    // Setup the mocking of out:
    var numberConverter = Mock.Of<INumberParser>();

    var mockedNumber = Mock.Of<INumber>();

    Mock.Get(numberConverter)
        .Setup(x => x.TryParse(It.IsAny<string>(), out mockedNumber))
        .Callback(() => mockedNumber.Int32 = 456)
        .Returns(true);

    // Act:
    var tryGet = numberConverter.TryParse("any number", out var number);

    // Assert that values are properly set:
    Assert.That(tryGet, Is.True);
    Assert.That(number.Int32, Is.EqualTo(456));
}
```

Która metoda jest lepsza? Pierwsza jest nieco bardziej skomplikowana, ale bardziej ekspresywna, a z kolei druga, prostsza, nie wymaga użycia bardziej zaawansowanych technik typu matcher dla `Ref` czy definicja delegata w kontekście zwracania wartości. Wybór pozostawiam Tobie 😊

Cały przykład jest do ściągnięcia tutaj: https://github.com/dariusz-wozniak/MockingOutDemo
