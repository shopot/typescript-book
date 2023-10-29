## Замыкания

Лучшее, что когда-либо было реализовано в JavaScript - это замыкания. Функция в JavaScript имеет доступ к любой переменной, определенной во внешнем контексте. Это показано на примере ниже:

```ts
function outerFunction(arg) {
    var variableInOuterFunction = arg;

    function bar() {
        console.log(variableInOuterFunction); // Обращение к переменной внешнего контекста
    }

    // Вызов локальной функции демонстрирует, что arg доступен
    bar();
}

outerFunction("hello closure"); // выводит "hello closure"
```

Как вы видите, внутренняя функция обращается к переменной (variableInOuterFunction) внешнего контекста. Переменные из внешней функции замкнуты(или связаны) во внутренней функции. Отсюда и термин **замыкания**. Сама концепция достаточно проста и интуитивна.

А теперь потрясающая часть: Внутренняя функция имеет доступ к переменным внешнего контекста *даже после того, как внешняя функция вернула результат выполнения*. Это происходит потому что переменные все еще замкнуты во внутренней функции и не зависят от внешней функции. Давайте посмотрим на примере:

```ts
function outerFunction(arg) {
    var variableInOuterFunction = arg;
    return function() {
        console.log(variableInOuterFunction);
    }
}

var innerFunction = outerFunction("hello closure!");

// Обратите внимание, что outerFunction уже вернула результат выполнения
innerFunction(); // выводит "hello closure"
```

### Почему это круто
Это позволяет легко создавать объекты, например, для паттерна раскрывающегося модуля

```ts
function createCounter() {
    let val = 0;
    return {
        increment() { val++ },
        getVal() { return val }
    }
}

let counter = createCounter();
counter.increment();
console.log(counter.getVal()); // 1
counter.increment();
console.log(counter.getVal()); // 2
```

На высоком уровне это похоже на то, что делает Node.js (Не волнуйтесь, если это не стало понятным прямо сейчас. Понимание придет позже 🌹):

```ts
// Псевдо-код для объяснения концепции
server.on(function handler(req, res) {
    loadData(req.id).then(function(data) {
        // `res` замкнут и доступен
        res.send(data);
    })
});  
```
