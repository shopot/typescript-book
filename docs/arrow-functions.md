* [Стрелочная функция](#arrow-functions)
* [Совет: Необходимость стрелочных функций](#tip-arrow-function-need)
* [Совет: Опасность стрелочных функций](#tip-arrow-function-danger)
* [Совет: Стрелочные функции и библиотеки, которые используют `this`](#tip-arrow-functions-with-libraries-that-use-this)
* [Совет: Стрелочные функции и наследование](#tip-arrow-functions-and-inheritance)
* [Совет: Быстрый возврат объекта](#tip-quick-object-return)

### <a name='arrow-functions'>Стрелочная функция</a>

Заботливо называемая *толстой стрелкой* (потому что `->` это тонкая стрелка и `=>` это толстая стрелка), а также называемая *лямбда-функцией* (как в других языках). Наиболее часто используемый вариант `()=>something`. Мотивация для использования *толстой стрелки* - это:
1. Вам не нужно печатать `function`
2. Лексически отражает значение `this`
2. Лексически отражает значение `arguments`

Для языка, который претендует на функциональность, в JavaScript вы обычно набираете `function` достаточно часто. Стрелочная функция упрощает создание функции
```ts
var inc = (x)=>x+1;
```
`this` традиционно был болезненным пунктом в JavaScript. Как однажды сказал мудрый человек "Я ненавижу JavaScript, так как он слишком легко теряет значение `this`". Стрелочная функция решает это проблему, фиксируя значение `this` из окружающего контекста. Рассмотрим чистый JavaScript класс:

```ts
function Person(age) {
    this.age = age;
    this.growOld = function() {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 1, должен быть 2
```

Если вы запустите этот код в браузере, `this` внутри функции будет указывать на `window`, потому что `window` будет контекстом выполнения функции `growOld`. Исправить это можно использованием стрелочной функции:

```ts
function Person(age) {
    this.age = age;
    this.growOld = () => {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```

Это работает, потому что в стрелочной функции `this` ссылается на окружающий контекст. Это равнозначно следующему коду на JavaScript (вы бы написали это сами, если бы у вас не было TypeScript):

```ts
function Person(age) {
    this.age = age;
    var _this = this;  // сохраняем this
    this.growOld = function() {
        _this.age++;   // используем сохраненный this
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```

Обратите внимание, что используя TypeScript, вы можете писать более приятный код и комбинировать стрелочные функции и фичи классов:

```ts
class Person {
    constructor(public age:number) {}
    growOld = () => {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```

> [Видео об этом паттерне 🌹](https://egghead.io/lessons/typescript-make-usages-of-this-safe-in-class-methods)

#### <a name='tip-arrow-function-need'>Совет: Необходимость стрелочных функций</a>
Помимо краткого синтаксиса, вам *следует* использовать стрелочную функцию только в том случае, если вы хотите передать функцию кому-то для вызова.  Фактически:
```ts
var growOld = person.growOld;
// Позже кто-то вызовет ее:
growOld();
```

Если же вы сами ее вызываете:

```ts
person.growOld();
```
тогда `this` будет использовать корректный контекст вызова (в примере это `person`).

#### <a name='tip-arrow-function-danger'>Совет: Опасность стрелочных функций</a>

На самом деле, если вы хотите, чтобы *именно `this` был контекстом вызова*, тогда *не нужно использовать стрелочные функции*. Это относится к обратным вызовам, используемым библиотеками, типа jquery, underscore, mocha и другими. Если в документации указан вызов функции для `this`, тогда вам следует использовать обычную `function` вместо стрелочной функции. Точно также, если вы планируете использовать `arguments`, то не прибегайте к стрелочным функциям.

#### <a name='tip-arrow-functions-with-libraries-that-use-this'>Совет: Стрелочные функции и библиотеки, которые используют `this`</a>
Многие библиотеки используют `this`, например итератор в `jQuery` (один из примеров https://api.jquery.com/jquery.each/) будет использовать `this`, чтобы передать вам объект, который он в данный момент перебирает. В данном случае, если вам нужен доступ и к окружающему контексту, и к `this`, переданному из библиотеки, просто используйте дополнительную переменную, например `_self`,
 для хранения ссылки на окружающий контекст.

```ts
let _self = this;
something.each(function() {
    console.log(_self); // внешний контекст
    console.log(this); // контекст функции .each()
});
```

#### <a name='tip-arrow-functions-and-inheritance'>Совет: Стрелочные функции и наследование</a>
Стрелочные функции как методы классов прекрасно работают с наследованием:

```ts
class Adder {
    constructor(public a: number) {}
    add = (b: number): number => {
        return this.a + b;
    }
}
class Child extends Adder {
    callAdd(b: number) {
        return this.add(b);
    }
}
// Демонстрация работы
const child = new Child(123);
console.log(child.callAdd(123)); // 246
```

Однако, они не работают с `super`, когда вы пытаетесь переопределить метод базового класса в дочернем. Свойства копируются на `this`. Поскольку существует только один `this`, такие функции не могут участвовать в вызове `super` (`super` работает только с членами-прототипами). Вы можете легко обойти это, создав копию метода перед тем, как переопределить его.

```ts
class Adder {
    constructor(public a: number) {}
    // Теперь эту функцию можно безопасно передавать
    add = (b: number): number => {
        return this.a + b;
    }
}

class ExtendedAdder extends Adder {
    // Создание копии метода родителького класса
    private superAdd = this.add;
    // Переопределение
    add = (b: number): number => {
        return this.superAdd(b);
    }
}
```

### <a name='tip-quick-object-return'>Совет: Быстрый возврат объекта</a>

Иногда вам нужна функция, которая возвращает простой объект. Что-то вроде

```ts
// неправильный способ сделать это
var foo = () => {
    bar: 123
};
```
парсится как *блок*, содержащий *JavaScript Label*, во время выполнения (в соответствии со спецификацией JavaScript).

>  Вы в любом случае получите ошибку компилятора TypeScript о "неиспользуемой метке (label)". Метки - это старая (и в основном неиспользуемая) функция JavaScript, которую вы можете игнорировать как современный GOTO (который опытные разработчики считают плохим 🌹).

Вы можете исправить ошибку, обернув объект в `()`:

```ts
// корректно 🌹
var foo = () => ({
    bar: 123
});
```
