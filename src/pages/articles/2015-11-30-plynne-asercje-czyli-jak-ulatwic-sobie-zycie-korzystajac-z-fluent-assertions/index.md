---
title: „Płynne asercje”, czyli jak ułatwić sobie życie korzystając z Fluent Assertions?
redirect_from: 
  - "/2015/11/30/plynne-asercje-czyli-jak-ulatwic-sobie-zycie-korzystajac-z-fluent-assertions/"
date: "2015-11-30T20:21:19.000Z"
layout: post
draft: false
path: "/posts/plynne-asercje-czyli-jak-ulatwic-sobie-zycie-korzystajac-z-fluent-assertions"
category: "Programowanie"
tags:
  - "TDD"
  - "C#"
  - "FluentAssertions"
description: "Wzorzec projektowy fluent interface (polski odpowiednik... płynny interfejs?) przyjął się w środowisku .NET-owym bardzo dobrze. I słusznie! \"Płynna syntaktyka\" znacznie poprawia czytelność pisanego kodu. Jednym z sztandarowych przykładów jej użycia są asercje w testach, np. przy wsparciu biblioteki Fluent Assertions"
---

Wzorzec projektowy "[fluent interface](https://en.wikipedia.org/wiki/Fluent_interface)" (polski odpowiednik... _płynny interfejs_?) przyjął się w środowisku .NET-owym bardzo dobrze. I słusznie! "Płynna syntaktyka" znacznie poprawia czytelność pisanego kodu.

Jednym z sztandarowych przykładów jej użycia są asercje w testach. Pytanie który z poniższych zapisów jest bardziej naturalny?

Zwykły proszek, ekhm, asercja: 

```csharp
 Assert.AreEqual("Jan", firstName); 
```

czy odpowiednik "fluent": 

```csharp
 firstName.ShouldBe("Jan"); 
```
 
 Dla C# pojawiło się [kilka bibliotek wspierających zapis asercji w sposób "płynny"](https://github.com/dariusz-wozniak/List-of-Testing-Tools-and-Frameworks-for-.NET#assertion-frameworks), m.in. [Fluent Assertions](http://www.fluentassertions.com/), [Shouldly](https://github.com/shouldly/shouldly), [Should Assertion Library](https://github.com/erichexter/Should). Wszystkie biblioteki są do siebie podobne, polecam zaznajomić się z każdą by wyrobić sobie opinię i samemu wybrać najodpowiedniejszą. Najczęściej korzystam z dwóch pierwszych, natomiast w tym wpisie chciałbym przyjrzeć się bibliotece Fluent Assertions.

# Proste asercje

Nawiązując do pierwszego przykładu, zapis w Fluent Assertions będzie wyglądać następująco: 

```csharp
 firstName.Should().Be("Jan"); 
```

 Co więcej, nasze asercje możemy łączyć za pomocą koniunkcji: 
```csharp
 firstName.Should().StartWith("Ag").And.EndWith("a").And.Contain("szk"); 
```

 [Dokumentacja Fluent Assertions](https://github.com/dennisdoomen/fluentassertions/wiki) przedstawia sporo przykładów zastosowania biblioteki w praktyce. Poniżej wkleiłem proste i co ciekawsze przykłady, aby pokazać w jaki sposób biblioteka radzi sobie z asercjami dla obiektów, kolekcji, liczb, dat, typów Booleanowych: 

```csharp
 string firstName = "Jan";
firstName.Should().Be("Jan");
 
firstName = "Agnieszka";
firstName.Should().StartWith("Ag").And.EndWith("a").And.Contain("szk");
 
// Obiekty:
string theObject = null;
theObject.Should().BeNull();
 
theObject = "sth";
theObject.Should().NotBeNull();
theObject.Should().BeOfType();
 
int? theInt = 5;
theInt.Should().HaveValue(); // Nullable type
 
// Boolean:
bool theBoolean = true;
theBoolean.Should().BeTrue();
 
theBoolean = false;
theBoolean.Should().BeFalse();
 
// Stringi:
string theString = " ";
theString.Should().BeNullOrWhiteSpace();
 
theString = "This is a string";
theString.Should().Contain("is a");
theString.Should().NotContain("is an");
theString.Should().BeEquivalentTo("THIS IS A STRING"); // case insensitive
theString.Should().StartWith("This");
 
string emailAddress = "someone@somewhere.com";
emailAddress.Should().Match("*@*.com"); // wildcards
emailAddress.Should().MatchEquivalentOf("*@*.COM"); // case insensitive
 
string someString = "hello world";
someString.Should().MatchRegex("h.*world");
 
// Typy numeryczne:
int number = 6;

number.Should().BeGreaterOrEqualTo(5);
number.Should().BeGreaterThan(4);
number.Should().BeLessOrEqualTo(7);
number.Should().BeLessThan(68);
number.Should().BePositive();
number.Should().Be(6);
number.Should().NotBe(10);
number.Should().BeInRange(1, 10);
 
// Daty:
DateTime theDatetime = 
  new DateTime(year: 2010, month: 3, day: 1, hour: 22, minute: 15, second: 0);

theDatetime.Should().BeAfter(1.February(2010));
theDatetime.Should().BeBefore(2.March(2010));
theDatetime.Should().BeOnOrAfter(1.March(2010));
theDatetime.Should().Be(1.March(2010).At(22, 15));
theDatetime.Should().NotBe(1.March(2010).At(22, 16));
theDatetime.Should().HaveDay(1);
theDatetime.Should().HaveMonth(3);
theDatetime.Should().HaveYear(2010);
theDatetime.Should().HaveHour(22);
theDatetime.Should().HaveMinute(15);
theDatetime.Should().HaveSecond(0);
 
// Kolekcje:
var collection = new [] {2, 5, 3};

collection.Should().NotBeEmpty()
  .And.HaveCount(3)
  .And.ContainInOrder(new[] {2, 5});
 
collection.Should().HaveCount(3);
collection.Should().ContainSingle(x => x == 3);
collection.Should().NotContain(x => x > 10);
collection.Should().NotContainNulls();
collection.Should().NotBeEmpty();
collection.Should().NotBeNullOrEmpty();
collection.Should().IntersectWith(new[] {5});
```

# Asercje dla wyjątków

Aby wykorzystać FluentAssertions w łapaniu wyjątków należy zadeklarować metodę, która rzuca wyjątkiem do zmiennej typu Action, a następnie wywołać metodę ShouldThrow. Wygląda to następująco:

```csharp
Action act = () => throw new NotImplementedException();
act.Should().Throw<NotImplementedException>();
```

Bajecznie proste i czytelne.

# Komunikaty asercji

Jednym z kluczowych czynników oceny bibliotek do testowania jest sposób komunikacji w przypadku testów czerwonych. Fluent Assertions radzi sobie na tym polu bardzo dobrze.

Przykładowy komunikat dla kolekcji, która ma mieć 4 elementy, a w rzeczywistości (wirtualnej) ma 3: 

```
Expected 4 items because we thought we put three items in the collection, but found 3.
```

# Kompatybilność

Fluent Assertions działa z bibliotekami:

*   MSTest (Visual Studio 2010, 2012 Update 2, 2013 and 2015)
*   NUnit
*   XUnit, XUnit2
*   MBUnit
*   Gallio
*   NSpec
*   MSpec

# Werdykt

Jest to kwestia czysto-subiektywna, ale mi osobiście bardziej odpowiada pod względem czytelności zapis asercji w wersji fluent niż standardowej. Przyznam szczerze, że odzwyczaiłem się już od pisania zwykłych asercji (`Assert.That`, `Assert.AreEqual`, itd.) na rzecz Fluent Assertions. Polecam zaznajomić się z tą biblioteką, a jak już będziecie w jej obsłudze "fluent", to polecam poeksperymentować z konkurencją – Shouldly, Should Assertion Library, itp.

# Źródła i linki

* [Fluent Assertions: Oficjalna strona](http://www.fluentassertions.com/)
* [Fluent Assertions: Dokumentacja](https://github.com/dennisdoomen/fluentassertions/wiki)
* [List of Automated Testing Tools and Frameworks for .NET: Assertion Frameworks](https://github.com/dariusz-wozniak/List-of-Testing-Tools-and-Frameworks-for-.NET/blob/master/README.md#assertion-frameworks)