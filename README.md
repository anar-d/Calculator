# Calculator

iOS-калькулятор с сохранением истории операций и демонстрацией модифицированной VIPER-архитектуры.

## Что умеет приложение
- Базовые операции: `+`, `−`, `×`, `÷`.
- Вычисление процентов (`%`) от введённого числа.
- Отображение текущего ввода и результата.
- Сохранение выполненных операций в локальное хранилище (Core Data).
- Просмотр истории операций в отдельном модальном окне.
- Отдельное инфо-окно с описанием проекта.

## Быстрый запуск
1. Откройте `Calculator.xcodeproj` в Xcode.
2. Выберите iOS Simulator (например, iPhone 14/15).
3. Нажмите **Run** (`⌘R`).

> Минимальная версия iOS и версия Xcode берутся из настроек проекта в `project.pbxproj`.

## Архитектура
Архитектурой является улучшенный VIPER за счёт использования наследования протоколов,
а также паттерна «Фабричный метод».

![Architecture](https://psv4.userapi.com/c856332/u90917369/docs/d13/0cdee5b6553e/Group_1.png?extra=om8n-lhsqxe7R9ImY1KOF8pwE57Rj7hd3h3mG1geakwQNDY7dKQbdsW3KPE2fSPFAQ4B-jsMEgFlRWW416z8mNnSmy1z_Y-wTO9-8-BJrkGUO6P3rolqjk1tllQ2yYyY8PS01NsRoBTMBjJw3alBuQ)

### View
В приложении несколько View-компонентов:
- главный экран калькулятора;
- `UICollectionView` с кнопками калькулятора;
- окно `Info`;
- окно `Operations`.

Чтобы не плодить множество отдельных Presenter-классов, используется один Presenter,
разделённый протоколами на интерфейсы под конкретные View.

```swift
final class CalculatorViewController: UIViewController, CalculatorViewProtocol {
  var presenter: CalculatorPresenterProtocol!
}
 
final class InfoViewController: UIViewController, InfoViewProtocol {
  var presenter: CalculatorInfoViewPresenterProtocol!
}

final class OperationsViewController: UIViewController, OperationsViewProtocol {
  var presenter: CalculatorOperationsViewPresenterProtocol!
}

final class CalculatorCollectionViewCell: UICollectionViewCell, CalculatorCollectionViewCellProtocol {
  var presenter: CalculatorCollectionViewCellPresenterProtocol!
}
```

### Interactor
Та же схема применяется к Interactor: один класс и несколько протокольных интерфейсов
для разных сценариев.

```swift
final class CalculatorPresenter: CalculatorPresenterGeneralProtocol {
  var calculatorViewInteractor : CalculatorInteractorProtocol!
  var infoViewInteractor       : CalculatorInfoViewInteractorProtocol!
  var operationsViewInteractor : CalculatorOperationsViewInteractorProtocol!
}

final class CalculatorInteractor: CalculatorInteractorGeneralProtocol {
  weak var calculatorViewPresenter : CalculatorPresenterProtocol!
  weak var infoViewPresenter       : CalculatorInfoViewPresenterProtocol!
  weak var operationsViewPresenter : CalculatorOperationsViewPresenterProtocol!
}
```

### Presenter + ComputingFactory
Presenter главного View имеет ссылку на `ComputingFactoryProtocol`, который выполняет расчёты.

```swift
final class CalculatorPresenter: CalculatorPresenterGeneralProtocol {
  var computingFactory: ComputingFactoryProtocol!
}

final class ComputingFactory: ComputingFactoryProtocol {
  weak var presenter: CalculatorPresenterGeneralProtocol!
}
```

### Configurator
`CalculatorConfigurator` выполняет инициализацию зависимостей и связывает компоненты между собой.

```swift
final class CalculatorConfigurator: CalculatorConfiguratorProtocol {
  weak var calculatorView : CalculatorViewProtocol!
  let infoView            : InfoViewProtocol!
  let operationsView      : OperationsViewProtocol!
  
  let presenter        : CalculatorPresenterGeneralProtocol!
  let interactor       : CalculatorInteractorGeneralProtocol!
  let computingFactory : ComputingFactoryProtocol!
}
```

### Router
Router из классического VIPER здесь не используется: в приложении один экран и нет полноценной навигации.

## Хранение данных
История операций сохраняется в Core Data (`OperationData`) при вычислениях (`=` и `%`).

## Ограничения текущей реализации
- Некоторые случаи некорректного ввода не обрабатываются (например, неполное выражение).
- Приложение рассчитано на выражение формата `число операция число` за один расчёт.
- Нет расширенных математических функций (степени, скобки, память и т.д.).

## Идеи для развития
- Добавить валидацию и защиту от некорректного ввода.
- Поддержать цепочки вычислений и приоритет операций.
- Добавить unit/UI-тесты.
- Вынести тексты в локализацию.

## Заключение
Схема с протоколами обеспечивает гибкость и масштабируемость для экрана,
сохраняя целостность компонентов и выдавая каждому слою только нужный интерфейс.
