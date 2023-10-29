# Миксины (примеси)

TypeScript (и JavaScript) классы поддерживают строго одиночное наследование. К примеру, вы *не можете* сделать следующее:

```ts
class User extends Tagged, Timestamped { // ОШИБКА : невозможно множественное наследование
}
```

Еще один способ создания классов из повторно используемых компонентов - путем объединения более простых частичных классов, называемых миксины.

Идея проста: вместо *класса A, расширяющего класс B* для получения своей функциональности, *функция B принимает класс A* и возвращает новый класс с этой добавленной функциональностью. Функция `B` - это миксин.

> [Миксин] - это функция, которая

> 1. берет конструктор
> 2. создает класс, расширяющий этот конструктор новыми функциями
> 3. возвращает новый класс

Подробный пример:
```ts
// Требуется для всех миксинов
type Constructor<T = {}> = new (...args: any[]) => T;

////////////////////
// Примеры миксинов
////////////////////

// Миксин, который добавляет свойство
function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    timestamp = Date.now();
  };
}

// миксин, который добавляет свойство и методы
function Activatable<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    isActivated = false;

    activate() {
      this.isActivated = true;
    }

    deactivate() {
      this.isActivated = false;
    }
  };
}

////////////////////
// Использование для создания классов
////////////////////

// Простой класс
class User {
  name = '';
}

// Пользователь с отметкой времени
const TimestampedUser = Timestamped(User);

// Пользователь с отметкой времени и доступный для активации
const TimestampedActivatableUser = Timestamped(Activatable(User));

////////////////////
// Использование созданных классов
////////////////////

const timestampedUserExample = new TimestampedUser();
console.log(timestampedUserExample.timestamp);

const timestampedActivatableUserExample = new TimestampedActivatableUser();
console.log(timestampedActivatableUserExample.timestamp);
console.log(timestampedActivatableUserExample.isActivated);

```

Разложим этот пример на части.

## Возьмите конструктор

Миксины берут класс и расширяют его новой функциональностью. Итак, нам нужно определить, что такое *конструктор*. Просто как:

```ts
// Требуется для всех миксинов
type Constructor<T = {}> = new (...args: any[]) => T;
```

## Расширить класс и вернуть его

Довольно просто:

```ts
// Миксин, который добавляет свойство
function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    timestamp = Date.now();
  };
}
```

И это все 🌹
