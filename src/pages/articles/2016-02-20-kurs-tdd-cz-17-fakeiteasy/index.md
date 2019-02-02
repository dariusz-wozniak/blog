---
title: Kurs TDD cz. 17 — FakeItEasy
date: "2016-02-20T18:27:29.000Z"
layout: post
draft: false
path: "/posts/kurs-tdd-17-fakeiteasy"
category: "Programowanie"
tags:
  - "TDD"
  - "Kurs TDD"
  - "C#"
  - "FakeItEasy"
description: "Dziś w kursie TDD przyjrzymy się frameworkowi do tworzenia atrap, konkurencyjnemu do wcześniej poznanego Moq. FakeItEasy, bo o nim mowa, jest darmowy, łatwy w nauce, ma wsparcie dla C# i VB.NET, różni się od innych bibliotek nie tylko semantyką, ale także nieco innym podejściem do tematu tworzenia atrap."
---

Dziś w kursie TDD przyjrzymy się frameworkowi do tworzenia atrap, konkurencyjnemu do wcześniej poznanego Moq. **FakeItEasy**, bo o nim mowa, jest darmowy, łatwy w nauce, ma wsparcie dla C# i VB.NET, różni się od innych bibliotek nie tylko semantyką, ale także nieco innym podejściem do tematu tworzenia atrap.

![fakeiteasy](c7551327-d950-45aa-bb70-09a1cf4b93f6.png)

Co więcej można powiedzieć o FakeItEasy?

*   W FakeItEasy wszystkie atrapy są fake'm. Nie ma rozróżnienia na mocki, stuby, itp.
*   Korzysta z API opartego na semantyce fluent i wyrażeniach lambda.
*   Jest rozszerzalny.
*   Posiada wsparcie dla bardziej zaawansowanych technik, m.in. strict fakes, argument matching, atrapy asynchronicznych metod.

Części kursu TDD [15.](/posts/kurs-tdd-15-wstep-do-moq) i [16.](/posts/kurs-tdd-16-zaawansowane-techniki-moq-argument-matching-verify-callback) powstały z myślą o wprowadzeniu technik tworzenia atrap. W tej części kursu dowiemy się jak, w oparciu o wiedzę i kod zawarty w poprzednich częściach, posługiwać się biblioteką FakeItEasy. Zalecane jest zatem zaznajomienie się z dwoma poprzednimi częściami kursu.

# Wstęp

Zanim zaczniemy przygodę z frameworkiem, należy wiedzieć że:

*   Po pierwsze, aby cokolwiek zrobić w FakeItEasy – czy to utworzyć atrapę, czy to ustawić żądane dane wyjściowe – korzystamy z klasy o nazwie "A", która posiada statyczne metody służące do wszystkich potrzebnych nam operacji. Niekonwencjonalne, oryginalne, ale na swój sposób bardzo sympatyczne podejście do tematu.
*   A po drugie, FakeItEasy posiada dość nietypową na swój sposób semantykę. Potrzeba trochę czasu, aby się przyzwyczaić...

Tyle tytułem wstępu, a więc zaczynamy!

## Tworzenie atrap

Do tworzenia atrap służy metoda `Fake`, która zwraca typ `T`: 

```csharp
 ICustomer customer = A.Fake<ICustomer>(); 
```
 Aby przesłać parametry do konstruktora, korzystamy z metody `WithArgumentsForConstructor`: 

```csharp
 A.Fake<ICustomerRepository>(x =>
  x.WithArgumentsForConstructor(() =>
    new CustomerRepository(A.Fake<ICustomerValidator>()))); 
```

 Domyślnie fake'i w FakeItEasy są tworzone w trybie "loose", co oznacza że metoda lub właściwość, która nie ma zdefiniowanej wartości żądanej, zwróci wartość automatyczną (w tej bibliotece: string.Empty, default(T) lub Fake<T>). Aby stworzyć atrapę "strict", a więc taką, która musi posiadać definicję wartości oczekiwanych, korzystamy z metody Strict. W przypadku, gdy nastąpi odwołanie do metody lub właściwości klasy fake'owej, która nie ma przypisanej żądanej wartości, kod wyrzuci wyjątek. Przykład: 

```csharp
var customer = A.Fake<ICustomer>(x => x.Strict());
string firstName = customer.FirstName; // To wywołanie wyrzuci wyjątek
```

 Próba dostępu do właściwości `FirstName` skończy się wyjątkiem typu `FakeItEasy.ExpectationException` i komunikatem:

```
Additional information: Call to non configured method "get_FirstName" of strict fake.
```

## Definicja zachowania

Aby móc zdefiniować zachowanie naszego fake'a, korzystamy z metody CallTo, a następnie wywołujemy metodę Returns która przyjmuje wskazaną przez nas wartość. Przykładowe użycie: 

```csharp
 var customer = A.Fake<ICustomer>();
 A.CallTo(() => customer.GetAge()).Returns(20); 
```

 Właściwości fake'a możemy alternatywnie ustawić odwołując się do niej bezpośrednio: 

```csharp
 var customer = A.Fake<ICustomer>();
 customer.FirstName = "John"; 
```

 Jeśli chcemy aby nasza metoda wyrzuciła wyjątek, korzystamy z metody `Throws`:

```csharp
 var customer = A.Fake<ICustomer>();
 A.CallTo(() => customer.GetAge()).Throws<NotSupportedException>(); 
```

 Aby zdefiniować "callback", który wykona dowolną logikę, możemy posłużyć się metodą `Invokes`: 

```csharp
 int a = 0; var customer = A.Fake<ICustomer>();
 A.CallTo(() => customer.GetAge()).Invokes(() => a++);
 // a = 0
 customer.GetAge();
 // a = 1 
```

## Weryfikacja wywołań

Do weryfikacji czy dana metoda lub właściwość została wywołana lub nie, posługujemy się metodami `MustHaveHappened` oraz `MustNotHaveHappened`. Parametr `metody MustHaveHappened` pozwala na określenie oczekiwanej ilości wywołań. Przykład testu: 

```csharp
[Test]
public void WhenAddingCustomer_ThenValidateMethodOfValidatorIsCalledOnce()
{
    var customerValidator = A.Fake<ICustomerValidator>();
 
    var customerRepository = new CustomerRepository(customerValidator);
 
    var customer = A.Fake<ICustomer>();
    customer.FirstName = "John";
 
    customerRepository.Add(customer);
 
    A.CallTo(() =>
      customerValidator.Validate(customer)).MustHaveHappened(Repeated.Exactly.Once);
}
```

## Argument Matching

Dopasowanie parametrów danej metody możemy wykonać na kilka sposobów. Jeżeli zależy nam na konkretnych wartościach, to przekazujemy je w "standardowy" sposób do metody: 

```csharp
 A.CallTo(() => calculator.Add(2, 4)).MustHaveHappened(); 
```

 Jeśli chcemy ustalić regułę dla naszych parametrów, możemy posłużyć się właściwością `That` klasy `A` (oraz `That.Not`): 

```csharp
 A.CallTo(() => 
  calculator.Add(
    A<int>.That.Matches(i => i > 0),
    A<int>.That.Matches(i => i < 0))).MustHaveHappened(); 
```

 Jeśli natomiast nie zależy nam na wartości parametru, możemy przekazać właściwość Ignored (lub... podkreślnik, który jest jego odpowiednikiem) klasy `A`: 

```csharp
 A.CallTo(() => calculator.Add(A<int>.Ignored, A<int>.Ignored)).MustHaveHappened();
 A.CallTo(() => calculator.Add(A<int>._, A<int>._)).MustHaveHappened(); // Tożsame z powyższym 
```

# Podsumowanie

W tej części kursu poznaliśmy część funkcjonalności FakeItEasy, która będzie wystaraczająca w większości testów jednostkowych. Oczywiście, framework oferuje wiele więcej zaawansowanych technik. Zachęcam do odwiedzenia oficjalnej dokumentacji celem ich poznania.

Dla adeptów FakeItEasy, semantyka oparta o parametr typu `Expression<Func<T>>` lub `Expression<Action>` może budzić lekkie zamieszanie i zamęt. Z czasem jednak FakeItEasy staje się prostym i bardzo przyjaznym w użyciu frameworkiem.

Z pewnością każdy czytelnik zadał sobie pytanie o wybór frameworka: Moq czy FakeItEasy? Są to dwie bardzo mocne i ugruntowane już biblioteki i jeśli wybór ma nie być podyktowany ograniczeniami technicznymi któregoś z nich, to pozostaje kwestia preferencji względem semantyki i nieco odmiennej filozofii tworzenia atrap. Wybór pozostawiam czytelnikowi, zachęcając jednocześnie do eksperymentowania zarówno z Moq, jak i FakeItEasy (każdy projekt testowy może mieć inny framework, czemu nie?)

Tymczasem w kolejnej części kursu poznamy trzeciego konkurenta. Stay tuned!

# Linki zewnętrzne

*   [FakeItEasy: Strona internetowa](http://fakeiteasy.github.io/)
*   [FakeItEasy: GitHub](https://github.com/FakeItEasy/FakeItEasy)
*   [FakeItEasy: Dokumentacja](https://github.com/FakeItEasy/FakeItEasy/wiki)

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
4. FakeItEasy
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