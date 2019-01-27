---
title: C# — Nested object initializer syntax
date: "2014-10-12T16:43:19.000Z"
layout: post
draft: false
path: "/posts/csharp-nested-object-initializer-syntax/"
category: "C#"
tags:
  - "C#"
description: "W C# istnieje mało znana składnia do inicjalizacji zagnieżdzonych obiektów (nested object initializer syntax)."
---

W C# istnieje mało znana składnia do inicjalizacji zagnieżdzonych obiektów (_nested object initializer syntax_). Po raz pierwszy zetknąłem się z nią przy okazji lektury _The C# Programming Language (Covering C# 4.0)_, a później przy sugestii ReSharpera. Na czym polega cała zabawa? Spójrzmy na kod, za pomocą którego inicjalizujemy zagnieżdżony obiekt: 
```csharp
Rectangle rectangle = new Rectangle
{
     P1 = { X = 0, Y = 1 },
     P2 = { X = 2, Y = 3 }
}; 
```
 Powyższy kod kompilowany jest do postaci: 
```csharp
Rectangle r = new Rectangle(); 
r.P1.X = 0;
r.P1.Y = 1;
r.P2.X = 2;
r.P2.Y = 3; 
```
 Pozostała część kodu wygląda następująco (całość na [gist](https://gist.github.com/dariusz-wozniak/3cc70aa649761d6a0076 "gist")): 
```csharp
 public class Point
 {
     public int X { get; set; }
     public int Y { get; set; }
}

public class Rectangle
{
    public Rectangle()
    {
        P1 = new Point();
        P2 = new Point();
    }
    
    public Point P1 { get; set; }
    public Point P2 { get; set; } 
} 
```
 Kiedy stosować? Podaną składnią można sobie życie ułatwić, bo pozwoli nam to na skrót przy pisaniu struktur drzewiastych i zagnieżdżonych obiektów. Można też utrudnić, bo należy wziąć pod uwagę, że nie jest to powszechna technika i może budzić konsternację wśród innych osób przeglądających kod. * – Dla uproszczenia pominięto tymczasową zmienną ukrytą (hidden temporary variable)

### Źródła

\- [Dokumentacja C# - 7.6.10.2 - Object Initializer](http://books.google.pl/books?id=s-IH_x6ytuQC&pg=PT434&lpg=PT434&dq=nested+7.6.10.2&source=bl&ots=ls631RfRfU&sig=v_7KJ_JymXXQklwPtdjQ5lR9hXY&hl=pl&sa=X&ei=kqE6VPeQEZOxabzpgrgI&ved=0CEAQ6AEwBA#v=onepage&q=nested%207.6.10.2&f=false)