---
layout: post
title: "Разница и общее между двумя коллекциями"
tags:  [kotlin, collections]
comments: false
---

Рассмотрим варианты нахождения различий или общего в двух коллекциях.

## Различия

Для примера возьмём два простых списка. Подойдёт любая коллекция с совпадающим типом параметра.

``` kotlin
val first = listOf(1,2,3,4,5)
val second = listOf(4,5,6,7,8)
```

Для получения списка элементов, уникальных только для первой коллекции,
воспользуемся вычитанием. Причём можно использовать как оператор, так и метод **minus**.

``` kotlin
val differenceFirst = first - second // first.minus(second)
println("differenceFirst = $differenceFirst")

// differenceFirst = [1, 2, 3]
```
Результатом будет список уникальных элементов.

Аналогично можем воспользоваться фильтрацией:
``` kotlin
val filterFirst = first.filterNot { second.contains(it) }
println("filterFirst = $filterFirst")

// filterFirst = [1, 2, 3]
```

Следующий способ делает похожее, но с одним исключением - возвращаемый объект будет типа **Set<T>**.
Это тип коллекции, не содержащий повторяющиеся элементы. На примере будем вычитать из второй коллекции, но разницы
нет какую коллекцию брать за основу.
``` kotlin
val differenceSecond = second.subtract(first)
println("differenceSecond = $differenceSecond")

// differenceSecond = [6, 7, 8]
```

Даже если вторая коллекция содержала, скажем, несколько элементов 8, то в результирующей коллекции такое значение
было бы одно. В случае с вычитанием ситуация обратная - повторящиеся элементы могут присуствовать.

## Обощения

Для поиска общих элементов можем обратиться к фильтрации и мы получим список элементов первой коллекции, которые есть
так же и во второй. В этом случае если в первой коллекции будут повторы, то после фильтрации они и останутся.
``` kotlin
val firstFilterIntersect = first.filter { second.contains(it) }
println("filterIntersect = $firstFilterIntersect")

// firstFilterIntersect = [4, 5]
```

Для получения общих элементов без повторения воспользуемся методом **intersect**. Метод выдаёт в качестве результата
**Set<T>** без повторовяющихся элементов.
``` kotlin
val intersect = first.intersect(second)
println("intersect = $intersect")

// intersect = [4, 5]
```