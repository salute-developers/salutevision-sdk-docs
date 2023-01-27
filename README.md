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
implementation files('libs/salutecv-mobile.aar')
```

2. Импортируйте необходимые объекты:

```
import ru.sberdevices.salutecv.barcode.BarcodeRecognizer
```

3. Создайте распознаватель кодов:

```
private val recognizer = BarcodeRecognizer()
```

4. В обработчике фрейма камеры отправьте кадр на распознавание и получите результат:

```
// Анализ кадра
val codes = recognizer.process(
SmartVisionImage(
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
implementation files('libs/salutecv-mobile.aar')
```

2. Импортируйте необходимые объекты: 

```
import ru.sberdevices.salutecv.mrz.MrzRecognizer
```

3. Создайте экземпляр распознавателя и инициализируйте его. Не забудьте удалить подписку, например, в методе `onDestroy`:

```
private lateinit var recognizer: MrzRecognizer
 
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    recognizer = MrzRecognizer(this)
}
override fun onDestroy() {
    super.onDestroy()
    recognizer.unregisterObserver()
}
```

4. Подпишитесь на получение данных:

```
recognizer.registerObserver {
    // Обработайте it
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
    SmartVisionImage(
        baseContext,
        image,
        imageProxy.imageInfo.rotationDegrees
    )
)
```
