---
layout: post
title: "Создание списка listOfNotNull"
tags:  [kotlin, collections]
comments: false
---

В случае необходимости создать список, который будет содержать только не null элементы полезно будет использовать фнукцию listOfNotNull.

По сути эта функция заменяет фильтрацию null элементов только без дополнительных записей.

``` kotlin
listOf(1, null, 2, null).filter { it != null }

listOf(1, null, 2, null).filterNotNull()

listOfNotNull(1, null, 2, null)

// [1, 2]
```

Немного облегчаются функции, в которых необходимо вернуть список без null значений:
``` kotlin
fun getProfileData(firstName: String, nickName: String?, region: String): List<String> = 
    listOfNotNull(firstName, nickName, region)
```

Можем использользовать корутины, если у нас асинхронный способ получения значений.
``` kotlin
// запрос в сеть
suspend fun getRemoteInfo(id: Int): String? {
    return try {
        api.getInfo(id)
    } catch(e: Exception) {
        null
    }
}

suspend fun getInfo(): List<String> = listOfNotNull(
    getRemoteInfo(ID_FIRST_NAME_INFO),
    getRemoteInfo(ID_NICKNAME_INFO),
    getRemoteInfo(ID_REGION_INFO)
)
```

Таким образом получим более коротку запись списка без null значений.