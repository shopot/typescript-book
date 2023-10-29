## Связывание это плохо

Это определение `bind` в `lib.d.ts`:

```ts
bind(thisArg: any, ...argArray: any[]): any;
```

Как видите, возвращается **any**! Это означает, что вызов `bind` для функции приведет к полной потере проверки типа исходной сигнатуры функции.

Например, следующий пример:

```ts
function twoParams(a:number,b:number) {
    return a + b;
}
let curryOne = twoParams.bind(null,123);
curryOne(456); // Okay, но тип не проверен!
curryOne('456'); // Разрешено, потому что тип не проверен!
```

Лучше написать это с помощью простой [стрелочной функции](../arrow-functions.md) с явным описанием типа:
```ts
function twoParams(a:number,b:number) {
    return a + b;
}
let curryOne = (x:number)=>twoParams(123,x);
curryOne(456); // Okay и тип проверен!
curryOne('456'); // Ошибка!
```

Но если вы ожидаете каррированную функцию [для этого есть образец получше](./currying.md).

### Члены класса
Другое распространенное использование - использование `bind` для обеспечения правильного значения `this` для функций класса. Не делай этого!

Следующее демонстрирует то, что вы теряете проверку типа параметра, если используете `bind`:

```ts
class Adder {
    constructor(public a: string) { }

    add(b: string): string {
        return this.a + b;
    }
}

function useAdd(add: (x: number) => number) {
    return add(456);
}

let adder = new Adder('у Мэри была маленькая 🐑');
useAdd(adder.add.bind(adder)); // Нет ошибки компиляции!
useAdd((x) => adder.add(x)); // Ошибка: число нельзя присвоить строке
```

Если у вас есть функция-член класса, которую вы **ожидаете** передать, [используйте в первую очередь стрелочную функцию](../arrow-functions.md), например, можно было бы написать тот же класс `Adder` как:

```ts
class Adder {
    constructor(public a: string) { }

    // This function is now safe to pass around
    add = (b: string): string => {
        return this.a + b;
    }
}
```

Другой альтернативой является *вручную* указать тип связываемой переменной, например:

```ts
const add: typeof adder.add = adder.add.bind(adder);
```
