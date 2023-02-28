# SaluteVision Mobile Android SDK

SDK алгоритмов компьютерного зрения для Android.

SaluteVision Mobile SDK — это набор инструментов для решения задач с использованием компьютерного зрения.

Бинарные файлы поставляются в виде AAR-архивов.

**Поддерживаемые архитектуры:**

- `arm64-v8a`
- `armeabi-v7a`
- `x86`
- `x86_64`


## Использование BarcodeRecognizer

Модуль распознавания штрих-кодов. Выполняется распознавание только одного штрих-кода, находящегося в кадре.

1. Подключите библиотеку в `build.gradle`: 

```
implementation files('libs/salutevision-mobile-public-1.0.0.aar')
```

2. Импортируйте необходимые объекты:

```
import ru.sberdevices.salutevision.barcode.BarcodeRecognizer
```

3. Создайте распознаватель кодов:

```
private val recognizer = BarcodeRecognizer()
```

4. В обработчике фрейма камеры отправьте кадр на распознавание и получите результат:

```
// Анализ кадра
val codes = recognizer.process(
    SaluteVisionImage(
        baseContext,
        image,
        imageProxy.imageInfo.rotationDegrees
        )
    )
    // Обработка результатов
    if (!codes.isNullOrEmpty()) {
        //...
    }
```

## Использование MrzRecognizer

1. Подключите библиотеку в `build.gradle`:

```
implementation files('libs/salutevision-mobile-public-1.0.0.aar')
```

2. Импортируйте необходимые объекты: 

```
import ru.sberdevices.salutevision.mrz.MrzRecognizer
```

3. Создайте экземпляр распознавателя:

```
private val recognizer = MrzRecognizer()
```

4. Подпишитесь на получение данных. Не забудьте удалить подписку, например, в методе `onDestroy`:

```
recognizer.registerObserver {
    // Обработайте it
}

override fun onDestroy() {
    super.onDestroy()
    recognizer.unregisterObserver()
}
```

`it` — это объект класса `MrzRecord` со следующей структурой:

```
data class MrzRecord(
    val doc_type_code: String,
    val name: String,
    val last_name: String,
    val gender: String,
    val nationality: String,
    val country: String,
    val birth_date: String,
    val personal_number: String,
    val document_number: String,
    val expiry_date: String
)
```

5. Отправьте изображение на распознавание:

```
recognizer.process(
    SaluteVisionImage(
        baseContext,
        image,
        imageProxy.imageInfo.rotationDegrees
    )
)
```
