---
layout: post
title: "Создание коллекций с помощью builder функций"
tags: [kotlin, collections]
comments: false
---

Помощником в создании коллекций могут послужить builder функции *buildList*, *buildMap* и *buildSet*. 
С помощью них можно сгенерировать коллекцию для чтения не создавая для этого отдельно её изменяемый вариант.

Для добавления в коллекцию нужно вызвать соответствующий метод. Например, для **List** - это *add*.

``` kotlin
fun getIntList() = buildList {
    val a = 10
    val b = 20
    val c = 30
    add(a)
    add(b + c)
}
```

Это оказывается удобным, когда не все данные нужно будет добавлять. 

``` kotlin
val ifSettings = buildMap<String, Int> {
    put("firstParameter", 5)
    put("valParameter", paramValue)
    if (externalValue != null) put("external", externalValue)
}

val whenSettings = buildMap<String, Int> {
    put("firstParameter", 5)
    put("valParameter", paramValue)
    externalValue?.let {
        when(type) {
            Type.A -> it * 2
            Type.B -> it + 10
        }
    }
}
```

Таким образом мы можем воспользоваться удобным механизмом, особенно если у нас есть дополнительные условия для добавления элементов в коллекцию.