* [Начинаем работу с TypeScript](#getting-started-with-typescript)
* [Версия TypeScript](#typescript-version)

# Начинаем работу с TypeScript

TypeScript компилируется в чистый JavaScript. JavaScript используется для написания пользовательских скриптов ( как в браузере так и на сервере ). Таким образом для работы вам понадобится:

* Компилятор TypeScript (доступен OSS в [исходниках](https://github.com/Microsoft/TypeScript/) и в [NPM](https://www.npmjs.com/package/typescript))
* Редактор для написания TypeScript кода ( вы можете использовать любой, какой вам нравится. Я использую [vscode 🌹](https://code.visualstudio.com/) с [расширением которое я написал](https://marketplace.visualstudio.com/items?itemName=basarat.god). Также [множество IDES имеют отличную поддержку]( https://github.com/Microsoft/TypeScript/wiki/TypeScript-Editor-Support))


## Версия TypeScript

Вместо использования *стабильной* версии компилятора TypeScript мы будет использовать ночную сборку компилятора, поскольку в данной книге рассматривается функционал который еще не реализован ни в одной стабильной версии, также ночная сборка **обрабатывает больше ошибок чем стабильный релиз**.

Установить её можно с помощью команды

```
npm install -g typescript@next
```

После этого ваша консольная утилита `tsc` будет работать наиболее полным набором фич доступным на данный момент. Большая часть IDEs тоже поддерживает эту возможность.

* Вы можете переопределить используемую версию в *vscode* создав файл `.vscode/settings.json` и добавив в него следующие строки:

```json
{
  "typescript.tsdk": "./node_modules/typescript/lib"
}
```

## Исходный код
Примеры кода описанные в данной книге доступны в github репозитории https://github.com/basarat/typescript-book/tree/master/code большая часть примеров кода может быть запущена через vscode и вы можете поиграть с ним как вам угодно. Для примеров кода в которых требуется предварительная установка ( например npm модулей ), мы добавляем ссылки перед ним, например.

`this/will/be/the/link/to/the/code.ts`
```ts
// This will be the code under discussion
```

После необходимой подготовки мы можем перейти к рассмотрению синтаксиса TypeScript.
