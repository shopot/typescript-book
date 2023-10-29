- [Совместимость типов](#совместимость-типов)
- [Разумность](#разумность)
- [Структурность](#структурность)
- [Расхождение](#расхождение)
- [Функции](#функции)
  - [Тип возвращаемого значения](#тип-возвращаемого-значения)
  - [Количество параметров](#количество-параметров)
  - [Необязательные и остальные параметры](#необязательные-и-остальные-параметры)
  - [Типы параметров](#типы-параметров)
- [Перечисления](#перечисления)
- [Классы](#классы)
- [Обобщения](#обобщения)
- [Примечание: инвариантность](#примечание-инвариантность)

## Совместимость типов

Совместимость типов (как мы здесь обсуждаем) определяет, можно ли назначить один другому. Например, `string` и `number` несовместимы:

```ts
let str: string = "Hello";
let num: number = 123;

str = num; // ОШИБКА: число не может быть присвоено строке
num = str; // ОШИБКА: строка не может быть присвоена числу
```

## Разумность

Система типов TypeScript устроена так, чтобы быть удобной и допускать *нерациональное* поведение, например что угодно может быть присвоено для типа `any`, что означает сказать компилятору разрешить вам делать все, что вы захотите:

```ts
let foo: any = 123;
foo = "Hello";

// Позже
foo.toPrecision(3); // Разрешено, потому что вы описали foo как `any`
```

## Структурность

Объекты TypeScript структурно типизированы. Это означает, что *имена типов* не имеют значения, пока структуры совпадают.

```ts
interface Point {
    x: number,
    y: number
}

class Point2D {
    constructor(public x:number, public y:number){}
}

let p: Point;
// всё в порядке из-за структурной типизации
p = new Point2D(1,2);
```

Это позволяет вам сходу создавать объекты (как в ванильном JS) и при этом сохранять проверку типов всякий раз, когда это можно логически вывести.

Также *лишние* данные не считаются ошибкой:

```ts
interface Point2D {
    x: number;
    y: number;
}
interface Point3D {
    x: number;
    y: number;
    z: number;
}
var point2D: Point2D = { x: 0, y: 10 }
var point3D: Point3D = { x: 0, y: 10, z: 20 }
function iTakePoint2D(point: Point2D) { /* сделать что-то */ }

iTakePoint2D(point2D); // точное совпадение - okay
iTakePoint2D(point3D); // дополнительная информация - okay
iTakePoint2D({ x: 0 }); // Ошибка: отсутствует `y`
```

## Расхождение

Расхождение - это простая и важная для понимания концепция анализа совместимости типов.

Для простых типов `Base` и `Child`, если `Child` является дочерним по отношению к `Base`, то экземпляры `Child` могут быть присвоены переменной типа `Base`.

> Это полиморфизм 101

Совместимость сложных типов зависит от *расхождения*:

* Ковариантный: (ко === совместный) только в одном направлении.
* Контравариантный: (контра === обратный) только в противоположном направлении.
* Бивариантный: (би === оба) как ко, так и контра.
* Инвариантный: если типы не совпадают абсолютно полностью, то они несовместимы.

> Примечание: для максимально безопасной системы типов при присутствии мутабельности данных, как в JavaScript, `инвариантный` - единственный правильный вариант. Но, как уже упоминалось, *удобство* заставляет нас выбирать менее безопасный вариант.

## Функции

При сравнении двух функций следует учитывать несколько важных моментов.

### Тип возвращаемого значения

`Ковариантный`: Тип возвращаемого значения должен содержать хотя бы необходимые данные.

```ts
/** Иерархия типов */
interface Point2D { x: number; y: number; }
interface Point3D { x: number; y: number; z: number; }

/** Два примера-функции */
let iMakePoint2D = (): Point2D => ({ x: 0, y: 0 });
let iMakePoint3D = (): Point3D => ({ x: 0, y: 0, z: 0 });

/** Присвоение */
iMakePoint2D = iMakePoint3D; // Okay
iMakePoint3D = iMakePoint2D; // ОШИБКА: Point2D не может быть присвоен Point3D
```

### Количество параметров

Допускается меньшее количество параметров (т.е. функции могут игнорировать дополнительные параметры). Ведь они гарантированно вызываются хотя бы с необходимыми параметрами.

```ts
let iTakeSomethingAndPassItAnErr
    = (x: (err: Error, data: any) => void) => { /* сделать что-то */ };

iTakeSomethingAndPassItAnErr(() => null) // Okay
iTakeSomethingAndPassItAnErr((err) => null) // Okay
iTakeSomethingAndPassItAnErr((err, data) => null) // Okay

// ОШИБКА: параметр типа '(err: any, data: any, more: any) => null' не может быть назначен параметру типа '(err: Error, data: any) => void'.
iTakeSomethingAndPassItAnErr((err, data, more) => null);
```

### Необязательные и остальные параметры

Необязательные (предварительно определенное количество) и остальные параметры (любое количество параметров) совместимы, опять же для удобства.

```ts
let foo = (x:number, y: number) => { /* сделать что-то */ }
let bar = (x?:number, y?: number) => { /* сделать что-то */ }
let bas = (...args: number[]) => { /* сделать что-то */ }

foo = bar = bas;
bas = bar = foo;
```

> Примечание: необязательные (в нашем примере `bar`) и обязательные (в нашем примере `foo`) совместимы, только если strictNullChecks имеет значение false.

### Типы параметров

`бивариантный`: разработан для поддержки общих сценариев обработки событий

```ts
/** Иерархия событий */
interface Event { timestamp: number; }
interface MouseEvent extends Event { x: number; y: number }
interface KeyEvent extends Event { keyCode: number }

/** Пример слушателя событий */
enum EventType { Mouse, Keyboard }
function addEventListener(eventType: EventType, handler: (n: Event) => void) {
    /* ... */
}

// Неидеально, но полезно и распространено. Работает как двувариантная функция сравнения параметров
addEventListener(EventType.Mouse, (e: MouseEvent) => console.log(e.x + "," + e.y));

// Нежелательные альтернативы для достижения идеальности
addEventListener(EventType.Mouse, (e: Event) => console.log((<MouseEvent>e).x + "," + (<MouseEvent>e).y));
addEventListener(EventType.Mouse, <(e: Event) => void>((e: MouseEvent) => console.log(e.x + "," + e.y)));

// Не допускается (явная ошибка). Проверка типов применена для полностью несовместимых типов
addEventListener(EventType.Mouse, (e: number) => console.log(e));
```

Также делает `Array<Child>` присваиваемым  `Array<Base>` (ковариационным), поскольку функции совместимы. Ковариационный массив требует, чтобы все функции `Array<Child>` могли быть присвоены `Array<Base>`, например `push(t:Child)` назначается `push(t:Base)`, что стало возможным благодаря двувариантным параметрам функции.

**Это может сбивать с толку людей, пришедших из других языков**, которые ожидали бы следующей ошибки, но не в TypeScript:

```ts
/** Иерархия типов */
interface Point2D { x: number; y: number; }
interface Point3D { x: number; y: number; z: number; }

/** Два примера-функции */
let iTakePoint2D = (point: Point2D) => { /* сделать что-то */ }
let iTakePoint3D = (point: Point3D) => { /* сделать что-то */ }

iTakePoint3D = iTakePoint2D; // Okay : Разумно
iTakePoint2D = iTakePoint3D; // Okay : ЧЕГО?
```

## Перечисления

* Перечисления совместимы с числами, а числа совместимы с перечислениями.

```ts
enum Status { Ready, Waiting };

let status = Status.Ready;
let num = 0;

status = num; // OKAY
num = status; // OKAY
```

* Значения перечислений из разных типов перечислений считаются несовместимыми. Это делает перечисления пригодными для  *формального* использования(в отличие от структурных типов)

```ts
enum Status { Ready, Waiting };
enum Color { Red, Blue, Green };

let status = Status.Ready;
let color = Color.Red;

status = color; // ОШИБКА
```

## Классы

* Сравниваются только члены экземпляра и методы. *конструкторы* и *статика* роли не играют.

```ts
class Animal {
    feet: number;
    constructor(name: string, numFeet: number) { /** сделать что-то */ }
}

class Size {
    feet: number;
    constructor(meters: number) { /** сделать что-то */ }
}

let a: Animal;
let s: Size;

a = s;  // OK
s = a;  // OK
```

* `private` и` protected` члены *должны происходить из одного класса*. Такие члены по сути делают класс *именным*.

```ts
/** Иерархия классов */
class Animal { protected feet: number; }
class Cat extends Animal { }

let animal: Animal;
let cat: Cat;

animal = cat; // OKAY
cat = animal; // OKAY

/** Похож на Animal */
class Size { protected feet: number; }

let size: Size;

animal = size; // ОШИБКА
size = animal; // ОШИБКА
```

## Обобщения

Поскольку TypeScript имеет систему структурных типов, параметры типа влияют на совместимость только когда используются. Например, в следующем примере `T` не влияет на совместимость:

```ts
interface Empty<T> {
}
let x: Empty<number>;
let y: Empty<string>;

x = y;  // okay, y соответствует структуре x
```

Однако, если используется `T`, он будет играть роль в совместимости на основе его *конкретизации*, как показано ниже:

```ts
interface NotEmpty<T> {
    data: T;
}
let x: NotEmpty<number>;
let y: NotEmpty<string>;

x = y;  // ошибка, x и y несовместимы
```

В случаях, когда общие параметры не были *созданы*, они заменяются на `any` перед проверкой совместимости:

```ts
let identity = function<T>(x: T): T {
    // ...
}

let reverse = function<U>(y: U): U {
    // ...
}

identity = reverse;  // Okay, потому что (x: any)=>any совпадает с (y: any)=>any
```

Обобщения, включающие классы, сопоставляются по совместимости на уровне классов, как мы упоминали ранее. Например:

```ts
class List<T> {
  add(val: T) { }
}

class Animal { name: string; }
class Cat extends Animal { meow() { } }

const animals = new List<Animal>();
animals.add(new Animal()); // Okay 
animals.add(new Cat()); // Okay 

const cats = new List<Cat>();
cats.add(new Animal()); // Ошибка 
cats.add(new Cat()); // Okay
```

## Примечание: инвариантность

Мы сказали, что инвариантность - самый разумный вариант. Вот пример, в котором показывается, что `контравариантный` и `ковариантный` небезопасны для массивов.

```ts
/** Иерархия */
class Animal { constructor(public name: string){} }
class Cat extends Animal { meow() { } }

/** По одному экземпляру каждого */
var animal = new Animal("animal");
var cat = new Cat("cat");

/**
 * Демонстрация: полиморфизм 101
 * Animal <= Cat
 */
animal = cat; // Okay
cat = animal; // ОШИБКА: cat наследуется от animal

/** Массив экземпляров каждого для демонстрации расхождения */
let animalArr: Animal[] = [animal];
let catArr: Cat[] = [cat];

/**
 * Очевидно плохо: Контравариантность
 * Animal <= Cat
 * Animal[] >= Cat[]
 */
catArr = animalArr; // Okay, если контравариантный
catArr[0].meow(); // Разрешено, но БЭМС 🔫 во время выполнения


/**
 * Также плохо: ковариантный
 * Animal <= Cat
 * Animal[] <= Cat[]
 */
animalArr = catArr; // Okay, если ковариантный
animalArr.push(new Animal('another animal')); // Просто добавили animal в catArr!
catArr.forEach(c => c.meow()); // Разрешено, но БЭМС 🔫 во время выполнения
```
