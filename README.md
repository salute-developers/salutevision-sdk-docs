# SaluteVision Mobile Android SDK

SDK алгоритмов компьютерного зрения для Android.

## Описание проекта
SaluteVision Mobile SDK — это набор инструментов для решения задач с использованием компьютерного зрения.

## Распространение
Бинарные файлы поставляются в виде AAR-архивов.

**Поддерживаемые архитектуры:**

- `arm64-v8a`
- `armeabi-v7a`
- `x86`
- `x86_64`

## Использование

### Работа с изображениями
Вся работа с изображениями в Android SDK осуществляется через класс `SaluteVisionImage`
Для распознавания кадра небходимо создать экземпляр класса `SaluteVisionImage` - в конструктор передать `android.media.Image`,
текущий контекст и угол поворота в градусах. В большинстве случаев логика работы с камерой на Android будет выглядеть так:
```kotlin
    val imageAnalyzer = ImageAnalysis.Builder()
        .setBackpressureStrategy(ImageAnalysis.STRATEGY_KEEP_ONLY_LATEST)
        .setTargetResolution(Size(1080, 1920))
        .build()
        .also {
            it.setAnalyzer(Executors.newSingleThreadExecutor()) { imageProxy ->
                imageProxy.image?.let { image ->
                    val visionImage = SaluteVisionImage(context, image, imageProxy.imageInfo.rotationDegrees)
                    ...
                }
            }
            imageProxy.close
        }
    cameraProvider.bindToLifecycle(this, selector, preview, imageAnalyzer)
```

### Использование SDK 
Вся функциональность SDK поставляется в виде независимых классов: `BarcodeRecognizer`, `MrzRecognizer`, `DocumentLocalizer`. 
Более подробную информацию об устройстве каждого класса можно найти в документации конкретного класса.

### Подключение модулей
1. Подключите библиотеку в build.gradle:
```groovy
    implementation files('libs/salutevision-mobile-public-1.0.0.aar')
```

2. Импортируйте необходимые объекты:
```kotlin
    import ru.sberdevices.salutevision.SaluteVisionSdk
```

### Использование BarcodeRecognizer
Модуль распознавания штрих-кодов. Выполняется распознавание всех штрих-кодов, находящихся в кадре.
1. Создайте распознаватель кодов. Обратите внимание на то, что распознаватель кодов создается один раз для одного экрана
распознавания (Activity) - нельзя пересоздавать его на каждом кадре:
```kotlin
    private val recognizer = SaluteVisionSdk.createBarcodeRecognizer()
```

2. В обработчике фрейма камеры создайте экземпляр класса `SaluteVisionImage`
```kotlin
    val visionImage = SaluteVisionImage(context, imageProxy.image, imageProxy.imageInfo.rotationDegrees)
```

3. Можно использовать прямоугольник с областью интереса (ROI). Это та область изображения, в которой должен находится штрих-код.
```kotlin
    val frame = RectF(left, top, right,bottom) // получить RectF, обозначающий область изображения со штрих-кодом
```

4. Отправьте кадр на распознавание и получите результат. Если roi не используется, то можно просто передать null:
```kotlin
    val codes = recognizer.process(visionImage, frame)
     
    // Обработка результатов
    if (!codes.isNullOrEmpty()) {
        ...
    }
```

5. Результатом распознавания является структура BarcodeRecognition с опциональными полями info и geometry. Если есть геометрия, то значит объект распознавания попал в кадр. Если есть info, то объект распознан:
```kotlin
    BarcodeRecognition {
        info : BarcodeRecognitionInfo? {
            text : String
            format : BarcodeFormat
            sourceEncoding : String
            hasEci : Boolean
            rawBytes : ByteArray
        }
        geometry : List<PointF>?
    }
```

### Использование MrzRecognizer
Модуль распознавания Machine Readable Zone
1. Создайте экземпляр распознавателя и инициализируйте его. Обратите внимание на то, что распознаватель кодов создается один раз для одного экрана распознавания (Activity) - нельзя пересоздавать его на каждом кадре. Не забудьте удалить подписку, например, в методе onDestroy:
```kotlin
    private val recognizer = SaluteVisionSdk.createMrzRecognizer()
    ...
    override fun onDestroy() {
        super.onDestroy()
        recognizer.unregisterObserver()
    }
```

2. Подпишитесь на получение данных:
```kotlin
    recognizer.registerObserver {
        // Обработайте it
    }
```

3. it — это объект класса MrzRecord со следующей структурой:
```kotlin
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

4. В обработчике фрейма камеры создайте экземпляр класса `SaluteVisionImage`
```kotlin
    val visionImage = SaluteVisionImage(context, imageProxy.image, imageProxy.imageInfo.rotationDegrees)
```

5. Отправьте изображение на распознавание:
```kotlin
    recognizer.process(visionImage)
```

### Использование DocumentLocalization
Детектор документов
1. Создайте определитель документов
```kotlin
    val recognizer = SaluteVisionSdk.createDocumentLocalizer()
```

2. В обработчике фрейма камеры создайте экземпляр класса `SaluteVisionImage`
```kotlin
    val visionImage = SaluteVisionImage(context, imageProxy.image, imageProxy.imageInfo.rotationDegrees)
```

3. Отправьте кадр на распознавание и получите результат:
```kotlin
    val recognitions = recognizer.process(visionImage)
     
    if (!recognitions.isNullOrEmpty()) {
    //...
    }
```

4. Результат работы классификатора - список объектов типа DocumentRecognition:
```kotlin
    DocumentRecognition{
        info : DocumentRecognitionInfo? {
            crop : Bitmap?
        }
        geometry : List<PointF>?
    }
```
