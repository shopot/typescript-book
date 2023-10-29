## readonly
Система типов TypeScript позволяет помечать отдельные элементы интерфейса `доступными только для чтения`. Это позволяет вам работать в функциональном стиле (в котором неожиданные мутации это плохо):

```ts
function foo(config: {
    readonly bar: number,
    readonly bas: number
}) {
    // ..
}

let config = { bar: 123, bas: 123 };
foo(config);
// Вы можете быть уверены, что `config` не изменился 🌹
```

Вы также можете использовать `readonly` в определениях `interface` и `type`, например:

```ts
type Foo = {
    readonly bar: number;
    readonly bas: number;
}

// Инициализация в порядке
let foo: Foo = { bar: 123, bas: 456 };

// Мутация нет
foo.bar = 456; // Ошибка: Выражение присваивания не может осуществлено для константы или свойства, доступного только для чтения
```

Вы даже можете объявить свойство класса как `readonly`. Вы можете инициализировать их в момент объявления или в конструкторе, как показано ниже:

```ts
class Foo {
    readonly bar = 1; // OK
    readonly baz: string;
    constructor() {
        this.baz = "hello"; // OK
    }
}
```

## Тип Readonly
Существует тип `Readonly`, который принимает тип `T` и помечает все его элементы как `readonly`. Вот демонстрация использования этого на практике:

```ts
type Foo = {
  bar: number;
  bas: number;
}

type FooReadonly = Readonly<Foo>; 

let foo:Foo = {bar: 123, bas: 456};
let fooReadonly:FooReadonly = {bar: 123, bas: 456};

foo.bar = 456; // Okay
fooReadonly.bar = 456; // ОШИБКА: bar только для чтения
```

### Различные варианты использования

#### ReactJS
ReactJS - это библиотека, которая любит иммутабельность, вы *можете* пометить ваши `Props` и `State` как неизменяемые, например:

```ts
interface Props {
    readonly foo: number;
}
interface State {
    readonly bar: number;
}
export class Something extends React.Component<Props,State> {
  someMethod() {
    // Вы можете быть уверены, что никто не сможет сделать следующее
    this.props.foo = 123; // ОШИБКА: (props неизменяемые)
    this.state.baz = 456; // ОШИБКА: (следует использовать this.setState)
  }
}
```

Однако вам не нужно это делать, поскольку определения типов для React уже помечают их как `readonly` (внутренняя оболочка универсального типа соответствует типу `Readonly`, упомянутому выше). Поэтому, достаточно:

```ts
export class Something extends React.Component<{ foo: number }, { baz: number }> {
  // Вы можете быть уверены, что никто не сможет сделать следующее
  someMethod() {
    this.props.foo = 123; // ОШИБКА: (props неизменяемые)
    this.state.baz = 456; // ОШИБКА: (следует использовать this.setState)
  }
}
```

#### Обратно-совместимая иммутабельность

Вы даже можете пометить сигнатуры индекса только для чтения:

```ts
/**
 * Объявление
 */
interface Foo {
    readonly[x: number]: number;
}

/**
 * Использование
 */
let foo: Foo = { 0: 123, 2: 345 };
console.log(foo[0]);   // Okay (чтение)
foo[0] = 456;          // Ошибка (изменение): Readonly
```

Это замечательно, если вы хотите использовать нативные массивы JavaScript в *иммутабельном* виде. На самом деле TypeScript поставляется с интерфейсом `ReadonlyArray<T>`, позволяющим вам сделать именно это:

```ts
let foo: ReadonlyArray<number> = [1, 2, 3];
console.log(foo[0]);   // Okay
foo.push(4);           // Ошибка: не возможен в ReadonlyArray, поскольку он изменяет массив
foo = foo.concat([4]); // Okay: создать копию
```

#### Автоматический логический вывод
В некоторых случаях компилятор может автоматически делать предположение, что определенный элемент доступен только для чтения, например, внутри класса, если у вас есть свойство, у которого есть только геттер, но нет сеттера, оно предполагается только для чтения, например:

```ts
class Person {
    firstName: string = "John";
    lastName: string = "Doe";
    get fullName() {
        return this.firstName + this.lastName;
    }
}

const person = new Person();
console.log(person.fullName); // John Doe
person.fullName = "Dear Reader"; // Ошибка! fullName только для чтения
```

### Отличие от `const`
`const`

1. для ссылки на переменную
2. переменная не может быть переназначена ни на что другое

`readonly` это

1. для свойства
2. свойство может быть изменено из-за ссылочности

Пример, объясняющий 1:

```ts
const foo = 123; // ссылка на переменную
var bar: {
    readonly bar: number; // для свойства
}
```

Пример, объясняющий 2:

```ts
let foo: {
    readonly bar: number;
} = {
        bar: 123
    };

function iMutateFoo(foo: { bar: number }) {
    foo.bar = 456;
}

iMutateFoo(foo); // Параметр функции - foo ссылается на переменную foo
console.log(foo.bar); // 456!
```

По сути, `readonly` гарантирует, что свойство *не может быть изменено мной*, но если вы дадите его кому-то, кто не даёт такой гарантии (кому разрешено по причинам совместимости типов), они могут его изменить. Но если сам `iMutateFoo` сказал, что они не изменяют `foo.bar`, компилятор правильно пометит его как ошибку, как показано ниже:

```ts
interface Foo {
    readonly bar: number;
}
let foo: Foo = {
    bar: 123
};

function iTakeFoo(foo: Foo) {
    foo.bar = 456; // Ошибка! bar только для чтения
}

iTakeFoo(foo); // Параметр функции - foo ссылается на переменную foo
```

[](https://github.com/Microsoft/TypeScript/pull/6532)
