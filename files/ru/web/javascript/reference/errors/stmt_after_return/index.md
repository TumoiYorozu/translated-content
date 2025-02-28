---
title: "Warning: unreachable code after return statement"
slug: Web/JavaScript/Reference/Errors/Stmt_after_return
---

{{jsSidebar("Errors")}}

## Сообщение

```
Warning: unreachable code after return statement (Firefox)
```

```
Предупреждение: недоступный код после оператора return (Firefox)
```

## Тип ошибки

Предупреждение

## Что пошло не так?

Недоступный код после оператора `return` может возникнуть в следующих ситуациях:

- когда в коде программы есть какие-либо выражения после оператора {{jsxref("Statements/return", "return")}}
- когда используется оператор `return` без точки с запятой, но далее непосредственно за ним следует выражение.

Когда присутствует выражение после оператора `return`, то выдаётся предупреждение о том, что код программы после `return` недоступен, то есть он никогда не запустится и не выполнится.

Почему нужно ставить точку с запятой после оператора `return`? В случае оператора `return` без точки с запятой, совсем неясно, хотел ли разработчик вернуть результат, вычисляемый в следующей строке, или же он хочет остановиться сейчас и выйти из подпрограммы. Предупреждение указывает на неопределённость результата работы оператора `return`.

Предупреждение не появится для оператора `return` без точки с запятой, если за данной строкой следуют:

- {{jsxref("Statements/throw", "throw")}}
- {{jsxref("Statements/break", "break")}}
- {{jsxref("Statements/var", "var")}}
- {{jsxref("Statements/function", "function")}}

## Примеры

### Неверные варианты

```js example-bad
function f() {
  var x = 3;
  x += 4;
  return x;   //return завершает функцию немедленно,
  x -= 3;     //поэтому эта строка никогда не запустится; она недоступна
}

function f() {
  return     //эта строка трактуется как завершение функции оператором `return;`,
    3 + 4;   //поэтому происходит выход из функции, и эта строка не выполнится
}
```

### Верные варианты

```js example-good
function f() {
  var x = 3;
  x += 4;
  x -= 3;
  return x;  //OK: return находится после всех остальных выражений
}

function f() {
  return 3 + 4  //OK: return без точки с запятой и вычисляемое выражение находятся на одной строке
}
```

## Смотрите также

- {{jsxref("Statements/return", "Automatic Semicolon Insertion", "#Automatic_Semicolon_Insertion", 1)}}
