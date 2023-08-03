# SaluteVision Mobile iOS SDK

SDK алгоритмов компьютерного зрения для iOS.

SaluteVision Mobile SDK — это набор инструментов для решения задач с использованием компьютерного зрения.

iOS SDK доступен в виде `salutevision-mobile.xcframework`.

`xcframework` содержит зависимости под все целевые платформы и архитектуры в едином bundle.

**Поддерживаемые архитектуры:**

- ios-arm64,
- ios-x86_64-simulator,
- ios-arm64-simulator.



## Работа с изображениями

Работа с изображениями в iOS SDK осуществляется через класс `SaluteVisionImage`.

Чтобы создать экземпляр класса `SaluteVisionImage`, передайте данные в конструктор в формате `CVPixelBuffer`.

В большинстве случаев `CVPixelBuffer` — это стандартная сущность для получения фрейма с камеры телефона на платформе iOS.

Также `SaluteVisionImage` поддерживает конвертацию в `UIImage`.

## Подключение

Для подключения библиотеки и доступа к моделям SDK используется стандартная команда:

```
import SaluteVision
```

## Использование SDK

Вся функциональность SDK поставляется в виде независимых классов. Например: `BarcodeReader`, `MRZReader`.

Каждый из классов может быть использован как независимо, так и в сочетании с другими. 

## Использование BarcodeReader

1. Создаем экземляр класса
```swift
private let reader = BarcodeReader(formats: [
    .qr,
    .aztec
])
```
2. Создаем экземпляр класса `SaluteVisionImage`
```swift
let pixelBuffer = ... // получить CVPixelBuffer
let image = SaluteVisionImage(pixelBuffer: pixelBuffer)
```

3. Можно использовать прямоугольник с областью интереса (ROI). Это та область изображения, в которой должен находится штрих-код.
```swift
let roi: CGRect = ... // получить CGRect, обозначающий область изображения со штрих-кодом
```

4. Запускаем процесс распознавания и обрабатываем результат
```swift
// C использованием ROI
guard let results = reader.read(from: image, inside: roi) else {
    print("Empty result")
    return
}
// Без использования ROI
guard let results = reader.read(from: image) else {
    print("Empty result")
    return
}

// вывод информации о типе первого распознанного штрих-кода и результат распознования
print("Result did obtain, type: \(results.first?.info?.format.rawValue), result: \(results.first?.info?.text)")
```
## Использование MRZReader

1. Создаем экземпляр класса
```swift
private let reader = MRZReader()
```

2. Добавляем наблюдателя (observer), который должен реализовывать `MRZReaderObserver` протокол.
```swift
reader.registerObserver(self)
```

3. Создаем экземпляр класса `SaluteVisionImage`
```swift
let pixelBuffer = ... // получить CVPixelBuffer
let image = SaluteVisionImage(pixelBuffer: pixelBuffer)
```

4. Запускаем процесс распознавания и получаем геометрию локализации (синхронно)
```swift
let geometries = reader.read(image: image)
```
5. Результат распознования приходит наблюдателю в методе `func infoDidObtain(_ info: MRZInfo)` (асинхронно)
```swift
 func infoDidObtain(_ info: MRZInfo) {
     print("His name is \(info.name) and he is from \(info.country)")   
 }
```

## Использование DocumentLocalization

1. Создаем экземляр класса
```swift
private let localization = DocumentLocalization()
```
2. Создаем экземпляр класса `SaluteVisionImage`
```swift
let pixelBuffer = ... // получить CVPixelBuffer
let image = SaluteVisionImage(pixelBuffer: pixelBuffer)
```

3. Запускаем процесс локализации и обрабатываем результат
```swift
let result = localization.predict(image: image, shouldCorrectAR: true)

if result.isStable, let cropImage = result.cropImage {
    // Работаем с cropImage
}
```
