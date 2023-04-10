---
layout: post
title: "Map putIfAbsent и computeIfAbsent"
tags: [kotlin, map, collections]
comments: false
---

В основном для добавления объектов в Java Map используется метод *put*.
Есть менее популярный метод *putIfAbsent*, который устанавливает значение в случае его отсутствия и если значения не было, то возвращает null, иначе - текущее значение.

``` kotlin
val map = mutableMapOf<Int, String>()

// установит значение "first" и вернёт null
val first = map.putIfAbsent(1, "first")

// значение не будет установлено и будет возвращено "first"
val second = map.putIfAbsent(1, "second")
```

В дополнении есть ещё менее популярный метод *computeIfAbsent*, который позволяет вычислить новое значение в случае, если такого ключа нет или значение для ключа равно null. 
Если вычисленное значение будет равно null, то оно добавляться не будет.

``` kotlin
val map = mutableMapOf<Int, String?>()

// результат вычисления null, не добавляется, вернётся null
val first = map.computeIfAbsent(1) { key ->
    null
}
// добавится результат вычисления, вернётся "1 second attempt"
val second = map.computeIfAbsent(1) { key ->
    "$key second attempt"
}
// значение для данного ключа уже есть, вернётся текущее значение "1 second attempt"
val third = map.computeIfAbsent(1) { key ->
    "$key third attempt"
}
```

В отличии от *putIfAbsent*, здесь при добавлении вычисленного значение оно же и возвращается.
Данный метод можно использовать для вычисления начального значения.
Такое поведение подходит для случаев, когда по ключу у нас коллекция со множеством значений.

``` kotlin
// [first, second]
val multiMap = mutableMapOf<Int, MutableSet<String>>()
multiMap.computeIfAbsent(1) { mutableSetOf() }.add("first")
multiMap.computeIfAbsent(1) { mutableSetOf() }.add("second")
```

В данном примере создание **MutableSet**, а добавление значения два раза. Если мы не уверены, что **MutableSet** был создан, то *computeIfAbsent* как раз подходит.