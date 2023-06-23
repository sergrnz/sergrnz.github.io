---
layout: post
title: "Запрос разрешения для работы с файлами"
tags:  [kotlin, mvvm, delegates]
comments: false
---

Начиная с Android 11 (API 30) добавлены изменения в работе с файлами, в частности изменился способ запроса к файловой системе, для работы с файлами напрямую.

Проверяем наличие разрешения. В более новой версии API наличие проверяется с помощью **Environment**

``` kotlin
fun hasFileAccessPermission(): Boolean {
    return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
        Environment.isExternalStorageManager()
    } else ContextCompat.checkSelfPermission(
        applicationContext,
        Manifest.permission.READ_EXTERNAL_STORAGE
    ) == PackageManager.PERMISSION_GRANTED
}
```

Разрешение через RequestMultiplePermissions работает стандартно: если пользователь дал согласие, то работаем дальше,
иначе можем показать сообщение о необходимости для дальнейшей работы:

``` kotlin
private val requestPermissionLauncher =
    registerForActivityResult(
        ActivityResultContracts.RequestMultiplePermissions()
    ) {
        if (it[Manifest.permission.READ_EXTERNAL_STORAGE] == true) {
            onFileAccessPermissionGranted()
        } else {
            showPermissionMessage()
        }
    }
```

В случае с более новой версией API надо использовать обработку *activity for result*:

``` kotlin
    @RequiresApi(Build.VERSION_CODES.R)
    private val manageFileAccessResultLauncher = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) {
        if (Environment.isExternalStorageManager()) {
            onFileAccessPermissionGranted()
        } else {
            showPermissionMessage()
        }
    }

```
Проверка наличия разрешения повторно проверяется через Environment.

Соответсвенно запуск запроса начиная с Android 11 делается через **Intent**:

``` kotlin
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
    manageFileAccessResultLauncher.launch(
        Intent(
            Settings.ACTION_MANAGE_APP_ALL_FILES_ACCESS_PERMISSION,
            Uri.parse("package:" + BuildConfig.APPLICATION_ID)
        ))
} else {
    requestPermissionLauncher.launch(arrayOf(Manifest.permission.READ_EXTERNAL_STORAGE))
}
```