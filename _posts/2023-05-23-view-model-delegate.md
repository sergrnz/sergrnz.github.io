---
layout: post
title: "Делегирование логики ViewModel"
tags:  [Android, kotlin, mvvm, delegates]
comments: false
---

Использование базовых классов во ViewModel довольно частое явление. Это позволяет обобщить логику, добавить общие поля. 
Например, если мы хотим создать свой scope для корутин во ViewModel, то базовый класс будет хорошим местом, чтобы это сделать.
Но если общее поведение будет разростаться, а то и местами различаться, то базовый класс будет становиться всё больше и больше.
Избежать этого могут помочь [делегаты](https://kotlinlang.org/docs/delegation.html) kotlin.

Мы можем вынести обощённую логику ViewModel в интерфейс, реализовать его отдельном классе и передать объект этого класса в конструкторе.

``` kotlin
interface CustomLogic {
    fun doLogic()
}

open class BaseClass(customLogic: CustomLogic): CustomLogic by customLogic {
    init {
        customLogic.doLogic()
    }
}

class TestLogic: CustomLogic {
    override fun doLogic() { /* magic */ }
}

class TestClass() : BaseViewModel(TestLogic())
```

Подобным образом организуем базовое поведение для ViewModel.

Рассмотрим пример. В приложении имеется механизм переключения учётных записей пользователей без повторного выхода и входа.
Базовая логика должна содержать механизм отслеживания и реакции на переключение пользователей.

``` kotlin
interface UserObserver {
    val userChangeFlow: Flow<String>
}

interface UserChangeDelegate {
    fun onUserChange()
}

open class BaseViewModel(
    userObserver: UserObserver,
    userChangeDelegate: UserChangeDelegate
) : ViewModel(), UserChangeDelegate by userChangeDelegate {

    init {
        viewModelScope.launch {
            userObserver.userChangeFlow.collectLatest {
                userChangeDelegate.onUserChange()
            }
        }
    }
}
```

UserObserver оповещает о переключении, а userChangeDelegate выполняет базовую логику.
Пользователь может измениться в любой момент на любом экране.
Добавили базовый сценарий и далее добавляем обработку на экране с абстрактным списком.

``` kotlin
interface ListRepository {
    suspend fun loadData(): List<String>
}

class ListUserChangeDelegate(private val listRepository: ListRepository): UserChangeDelegate {

    private var scope: CoroutineScope? = null
    val dataFlow = MutableSharedFlow<List<String>>()

    fun registerScope(scope: CoroutineScope) {
        this.scope = scope
    }

    override fun onUserChange() {
        scope?.launch {
            val data = listRepository.loadData()
            dataFlow.tryEmit(data)
        }
    }
}

class ListViewModel(
    userObserver: UserObserver,
    private val userChangeDelegate: ListUserChangeDelegate
): BaseViewModel(userObserver, userChangeDelegate) {

    private val _listLiveData = MutableLiveData<List<String>>()
    private val listLiveData: LiveData<List<String>>
        get() = _listLiveData

    init {
        userChangeDelegate.registerScope(viewModelScope)

        viewModelScope.launch {
            userChangeDelegate.dataFlow.collect {
                _listLiveData.postValue(it)
            }
        }
    }
}
```

В данном примере при изменении пользователя будет заново запрошен список и обновлены данные через LiveData.
Подобное поведение можно повторять для других экранов, тем самым добавляя как разную, так и одинаковую обработку.
Есть возможно расширять или переопределять реализацию делегатов. Например:

``` kotlin
open class ListUserChangeDelegate(private val listRepository: ListRepository): UserChangeDelegate

class SpecificListUserChangeDelegate(
    listRepository: ListRepository
): ListUserChangeDelegate(listRepository) {
    
    override fun onUserChange() {
        super.onUserChange()
        doSpecificLogic()
    }
}
```

Формально в данном примере мы могли бы создать метод для переопределения в BaseViewModel и использовать его, однако
класс-делегат может содержать в себе гораздо больше методов и полей. Так же мы получаем механизм поставки специфической логики
через инъекцию зависимостей, если есть такая необходимость.