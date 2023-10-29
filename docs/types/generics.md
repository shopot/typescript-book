## Обобщения (generics)

Ключевой причиной использования обобщений является предоставление выразительных ограничителей типов используемых в нескольких частях кода. Частями могут быть:

* члены экземпляра класса
* методы класса
* параметры функции
* возвращаемое значение функции

## Мотивация и примеры

Рассмотрим простую реализацию структуры данных `Очередь` (первым пришел, первым вышел).  Простая в TypeScript / JavaScript выглядит так:

```ts
class Queue {
  private data = [];
  push = (item) => this.data.push(item);
  pop = () => this.data.shift();
}
```

Одной из проблем этой реализации является то, что она позволяет людям добавлять *что угодно* в очередь, а когда они это удаляют - это тоже может быть *что угодно*. Это показано ниже, где кто-то может добавить `строку` в очередь, в то время как использование фактически предполагает, чтобы были добавлены только` числа`:

```ts
class Queue {
  private data = [];
  push = (item) => this.data.push(item);
  pop = () => this.data.shift();
}

const queue = new Queue();
queue.push(0);
queue.push("1"); // Упс - ошибка

// и тут разработчик начинает
console.log(queue.pop().toPrecision(1));
console.log(queue.pop().toPrecision(1)); // ОШИБКА ВО ВРЕМЯ ВЫПОЛНЕНИЯ
```

Одно из решений (и на самом деле единственное в языках, которые не поддерживают обобщения) состоит в том, чтобы пойти дальше и создать *специальные* классы с этими ограничениями. Например очередь номеров по-быстрому:

```ts
class QueueNumber {
  private data = [];
  push = (item: number) => this.data.push(item);
  pop = (): number => this.data.shift();
}

const queue = new QueueNumber();
queue.push(0);
queue.push("1"); // ОШИБКА : не может добавить строку. Разрешены только номера

// ^ если эта ошибка исправлена, остальное тоже будет хорошо
```

Конечно, это может быстро превратится в боль, например, если вам нужна очередь строк, вам придется написать всё заново. На самом деле вы просто хотите чтобы независимо от того, какой тип элемента *добавляется*, он должен быть одинаковым для всего, что *удаляется*. Это легко сделать с помощью параметра *обобщение* (в данном случае на уровне класса):

```ts
/** Определение класса с обобщённым параметром */
class Queue<T> {
  private data = [];
  push = (item: T) => this.data.push(item);
  pop = (): T => this.data.shift();
}

/** Снова пример использования */
const queue = new Queue<number>();
queue.push(0);
queue.push("1"); // ОШИБКА : не может добавить строку. Разрешены только номера

// ^ если эта ошибка исправлена, остальное тоже будет хорошо
```

Другой пример, который мы уже видели - это функция *reverse*, здесь есть общий ограничитель на то, что передается в функцию, и то, что возвращает функция:

```ts
function reverse<T>(items: T[]): T[] {
    var toreturn = [];
    for (let i = items.length - 1; i >= 0; i--) {
        toreturn.push(items[i]);
    }
    return toreturn;
}

var sample = [1, 2, 3];
var reversed = reverse(sample);
console.log(reversed); // 3,2,1

// Защита!
reversed[0] = '1';     // Ошибка!
reversed = ['1', '2']; // Ошибка!

reversed[0] = 1;       // Okay
reversed = [1, 2];     // Okay
```

В этом разделе вы видели примеры определения обобщений *на уровне класса* и *на уровне функций*. Незначительное дополнение: вы можете создавать обобщения только для методов класса. В качестве мини-примера рассмотрим следующий, где мы перемещаем функцию «reverse» в класс «Utility»:

```ts
class Utility {
  reverse<T>(items: T[]): T[] {
      var toreturn = [];
      for (let i = items.length - 1; i >= 0; i--) {
          toreturn.push(items[i]);
      }
      return toreturn;
  }
}
```

> СОВЕТ: Вы можете называть параметр обобщения как хотите. Обычно используется `T`,` U`, `V`, когда у вас есть простые обобщения. Если у вас более одного параметра попробуйте использовать значимые имена, например, `TKey` и `TValue` (обычно обобщения с префиксом `T` также называют *шаблонами* на других языках, например C ++).

## Бесполезные обобщения

Я видел, как люди используют обобщения просто так. Вопрос, который нужно задать, это *какое ограничение вы пытаетесь описать*. Если вы не можете ответить на него легко, у вас может быть бесполезное обобщение. Например. следующая функция

```ts
declare function foo<T>(arg: T): void;
```
Здесь обобщение `T` совершенно бесполезно, поскольку оно используется только в *единственном* месте - в параметре. Это могло бы быть просто:

```ts
declare function foo(arg: any): void;
```

### Шаблон проектирования: целесообразные обобщения

Рассмотрим функцию:

```ts
declare function parse<T>(name: string): T;
```

В этом случае вы можете видеть, что тип `T` используется только в одном месте. Таким образом, нет никаких ограничителей переиспользуемых *между* частями функции или класса. Это эквивалентно утверждению типа с точки зрения проверок:

```ts
declare function parse(name: string): any;

const something = parse('something') as TypeOfSomething;
```

Обобщения, используемые *только один раз*, не лучше, чем утверждение с точки зрения надежности проверки типов. Тем не менее, они обеспечивают *удобство* для вашего API.

Более очевидный пример - функция, которая загружает ответ json. Она возвращает промис *любого типа, который вы передадите*:
```ts
const getJSON = <T>(config: {
    url: string,
    headers?: { [key: string]: string },
  }): Promise<T> => {
    const fetchConfig = ({
      method: 'GET',
      'Accept': 'application/json',
      'Content-Type': 'application/json',
      ...(config.headers || {})
    });
    return fetch(config.url, fetchConfig)
      .then<T>(response => response.json());
  }
```

Обратите внимание, что вам все равно нужно явно описывать то, что вы хотите, но сигнатура `getJSON<T>`  `(config) => Promise<T>` сохраняет вам несколько нажатий клавиш (вам не придётся описывать возвращаемый тип `loadUsers`):

```ts
type LoadUsersResponse = {
  users: {
    name: string;
    email: string;
  }[];  // массив пользователей
}
function loadUsers() {
  return getJSON<LoadUsersResponse>({ url: 'https://example.com/users' });
}
```

Также `Promise<T>` в качестве возвращаемого значения определенно лучше, чем альтернативы, такие как `Promise<any>`.