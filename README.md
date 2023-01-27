# SaluteVision Mobile iOS SDK

SDK алгоритмов компьютерного зрения для iOS.

SaluteVision Mobile SDK — это набор инструментов для решения задач с использованием компьютерного зрения.

iOS SDK доступен в виде `smartcv-mobile.xcframework`.

`xcframework` содержит зависимости под все целевые платформы и архитектуры в едином bundle.

**Поддерживаемые архитектуры:**

- ios-arm64,
- ios-x86_64-simulator.


## Работа с изображениями

Работа с изображениями в iOS SDK осуществляется через класс `SmartVisionImage`.

Чтобы создать экземпляр класса `SmartVisionImage`, передайте данные в конструктор в формате `CVPixelBuffer`.

В большинстве случаев `CVPixelBuffer` — это стандартная сущность для получения фрейма с камеры телефона на платформе iOS.

Также `SmartVisionImage` поддерживает конвертацию в `UIImage`.

## Подключение

Для подключения библиотеки и доступа к моделям SDK используется стандартная команда:

```
import SmartVision
```

## Использование SDK

Вся функциональность SDK поставляется в виде независимых классов. Например: `BarcodeReader`, `MRZReader`.

Каждый из классов может быть использован как независимо, так и в сочетании с другими. 

## Использование BarcodeReader

1. Создайте экземляр класса:

```
private let reader = BarcodeReader(formats: [
.qr,
.aztec
])
```

2. Создайте экземпляр класса `SmartVisionImage`:

```
let pixelBuffer = ... // получить CVPixelBuffer
let image = SmartVisionImage(pixelBuffer: pixelBuffer)
```

3. Запустите процесс распознавания и обработайте результат:

```
guard let result = reader.read(image: image) else {
print("Empty result")
return
}
 
// вывод информации о типе баркода и результат распознавания
print("Result did obtain, type: \(result.format.rawValue), result: \(result.text)")
```

## Использование MRZReader

1. Создайте экземляр класса:

```
private let reader = MRZReader(fromFile: ModelRepository.paddleOCRV2)
```

2. Добавьте `observer` (класс должен быть отнаследован от `MRZReaderObserver`):

```
reader.registerObserver(self)
```

3. Создайте экземпляр класса `SmartVisionImage`:

```
let pixelBuffer = ... // получить CVPixelBuffer
let image = SmartVisionImage(pixelBuffer: pixelBuffer)
```

4. Запустите процесс распознавания и получите геометрию локализации:

```
let geometries = reader.read(image: image)
Сам результат распознавания приходит наблюдателю (observer) в методе `func infoDidObtain(_ info: MRZInfo)`:

func infoDidObtain(_ info: MRZInfo) {
    print("His name is \(info.name) and he is from \(info.country)")  
}
```

