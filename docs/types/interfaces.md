## Интерфейсы

Интерфейсы не оказывают *никакого* влияния на выполнение JS. Существует разные полезные возможности при объявления структуры переменных в TypeScript с помощью интерфейсов.

Следующие два являются эквивалентными объявлениями, первое использует *встроенное описание*, второе использует *интерфейс*:

```ts
// Пример A
declare var myPoint: { x: number; y: number; };

// Пример B
interface Point {
    x: number; y: number;
}
declare var myPoint: Point;
```

Однако прелесть *Примера B* заключается в том, что если кто-то пишет основанную на `myPoint` библиотеку, то ему легко расширить объявление `myPoint` просто добавив новое свойство:

```ts
// Библиотека a.d.ts
interface Point {
    x: number; y: number;
}
declare var myPoint: Point;

// Библиотека b.d.ts
interface Point {
    z: number;
}

// Ваш код
var myPoint.z; // Разрешено!
```

Это потому, что **интерфейсы в TypeScript не ограничены**. Это жизненно важный принцип TypeScript, который позволяет имитировать расширяемость JavaScript с помощью *интерфейсов*.


## Классы могут реализовывать интерфейсы

Если вы хотите использовать *классы*, следующие структуре объекта, который кто-то объявил для вас в `интерфейсе`, вы можете сделать это, используя ключевое слово `implements` для обеспечения совместимости:

```ts
interface Point {
    x: number; y: number;
}

class MyPoint implements Point {
    x: number; y: number; // Такой же как Point
}
```

По сути, при наличии этих `implements` любое изменение во внешнем интерфейсе `Point` приведет к ошибке компиляции в новом коде, что позволит вам следить за консистентностью кода:

```ts
interface Point {
    x: number; y: number;
    z: number; // Новая переменная
}

class MyPoint implements Point { // ОШИБКА : отсутствует переменная `z`
    x: number; y: number;
}
```

Обратите внимание, что `implements` ограничивает структуру *экземпляров* класса, т.е.

```ts
var foo: Point = new MyPoint();
```

И такие вещи, как `foo: Point = MyPoint` - это не одно и то же.


## Советы

### Не каждый интерфейс легко реализуем

Интерфейсы придуманы для объявления *любой произвольной* структуры, возможной в JavaScript.

Рассмотрим следующий интерфейс, где можно вызывать `new`:

```ts
interface Crazy {
    new (): {
        hello: number
    };
}
```

По сути, у вас будет что-то вроде:

```ts
class CrazyClass implements Crazy {
    constructor() {
        return { hello: 123 };
    }
}
// Так как
const crazy = new CrazyClass(); // crazy было бы {hello:123}
```

Вы можете *объявить* различные структуры с помощью интерфейсов и безопасно использовать JS код с помощью проверок TypeScript. Однако не всегда возможно реализовать эти структуры как классы TypeScript.