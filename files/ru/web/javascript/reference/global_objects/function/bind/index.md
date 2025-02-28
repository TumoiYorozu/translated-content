---
title: Function.prototype.bind()
slug: Web/JavaScript/Reference/Global_Objects/Function/bind
---

{{JSRef("Global_Objects", "Function")}}

## Сводка

Метод **`bind()`** создаёт новую функцию, которая при вызове устанавливает в качестве контекста выполнения `this` предоставленное значение. В метод также передаётся набор аргументов, которые будут установлены перед переданными в привязанную функцию аргументами при её вызове.

## Синтаксис

```
fun.bind(thisArg[, arg1[, arg2[, ...]]])
```

### Параметры

- `thisArg`
  - : Значение, передаваемое в качестве `this` в целевую функцию при вызове привязанной функции. Значение игнорируется, если привязанная функция конструируется с помощью оператора {{jsxref("Operators/new", "new")}}.
- `arg1, arg2, ...`
  - : Аргументы целевой функции, передаваемые перед аргументами привязанной функции при вызове целевой функции.

## Описание

Метод `bind()` создаёт новую "**привязанную функцию**" (**ПФ**). **ПФ** - это "необычный функциональный объект" ( термин из **ECMAScript 6** ), который является обёрткой над исходным функциональным объектом. Вызов **ПФ** приводит к исполнению кода обёрнутой функции.

**ПФ** имеет следующие внутренние ( скрытые ) свойства:

- **\[\[BoundTargetFunction]]** - оборачиваемый (целевой ) функциональный объект
- **\[\[BoundThis]]** - значение, которое всегда передаётся в качестве значения **this** при вызове обёрнутой функции.
- **\[\[BoundArguments]]** - список значений, элементы которого используются в качестве первого аргумента при вызове оборачиваемой функции.
- **\[\[Call]]** - внутренний метод. Выполняет код (функциональное выражение), связанный с функциональным объектом.

Когда **ПФ** вызывается, исполняется её внутренний метод **\[\[Call]]** со следующими аргументами **Call(_target_, _boundThis_, _args_).**

- **_target_** - **\[\[BoundTargetFunction]]**;
- _**boundThis** - **\[\[BoundThis]]**;
- _**args** _- **\[\[BoundArguments]].**

Привязанная функция также может быть сконструирована с помощью оператора {{jsxref("Operators/new", "new")}}: это работает так, как если бы вместо неё конструировалась целевая функция. Предоставляемое значение `this` в этом случае игнорируется, хотя ведущие аргументы всё ещё передаются в эмулируемую функцию.

## Примеры

### Пример: создание привязанной функции

Простейшим способом использования `bind()` является создание функции, которая, вне зависимости от способа её вызова, вызывается с определённым значением `this`. Обычным заблуждением для новичков в JavaScript является извлечение метода из объекта с целью его дальнейшего вызова в качестве функции и ожидание того, что он будет использовать оригинальный объект в качестве своего значения `this` (например, такое может случиться при использовании метода как колбэк-функции). Однако, без специальной обработки, оригинальный объект зачастую теряется. Создание привязанной функции из функции, использующей оригинальный объект, изящно решает эту проблему:

```js
this.x = 9;
var module = {
  x: 81,
  getX: function() { return this.x; }
};

module.getX(); // 81

var getX = module.getX;
getX(); // 9, поскольку в этом случае this ссылается на глобальный объект

// создаём новую функцию с this, привязанным к module
var boundGetX = getX.bind(module);
boundGetX(); // 81
```

### Пример: частичные функции

Следующим простейшим способом использования `bind()` является создание функции с предопределёнными аргументами. Эти аргументы (если они есть) передаются после значения `this` и вставляются перед аргументами, передаваемыми в целевую функцию при вызове привязанной функции.

```js
function list() {
  return Array.prototype.slice.call(arguments);
}

var list1 = list(1, 2, 3); // [1, 2, 3]

// Создаём функцию с предустановленным ведущим аргументом
var leadingThirtysevenList = list.bind(undefined, 37);

var list2 = leadingThirtysevenList(); // [37]
var list3 = leadingThirtysevenList(1, 2, 3); // [37, 1, 2, 3]
```

### Пример: с `setTimeout`

По умолчанию, внутри {{domxref("window.setTimeout()")}} контекст `this` устанавливается в объект {{domxref("window")}} (или `global`). При работе с методами класса, требующими `this` для ссылки на экземпляры класса, вы можете явно привязать `this` к колбэк-функции для сохранения экземпляра.

```js
function LateBloomer() {
  this.petalCount = Math.ceil(Math.random() * 12) + 1;
}

// Объявляем цветение с задержкой в 1 секунду
LateBloomer.prototype.bloom = function() {
  window.setTimeout(this.declare.bind(this), 1000);
};

LateBloomer.prototype.declare = function() {
  console.log('Я прекрасный цветок с ' +
    this.petalCount + ' лепестками!');
};
```

### Пример: привязывание функций, используемых в качестве конструкторов

> **Предупреждение:** этот раздел демонстрирует возможности JavaScript и документирует некоторые граничные случаи использования метода `bind()`. Показанные ниже методы не являются лучшей практикой и, вероятно, их не следует использовать в рабочем окружении.

Привязанные функции автоматически подходят для использования вместе с оператором {{jsxref("Operators/new", "new")}} для конструирования новых экземпляров, создаваемых целевой функцией. Когда привязанная функция используется для конструирования значения, предоставляемое значение `this` игнорируется. Однако, предоставляемые аргументы всё так же вставляются перед аргументами конструктора:

```js
function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function() {
  return this.x + ',' + this.y;
};

var p = new Point(1, 2);
p.toString(); // '1,2'


var emptyObj = {};
var YAxisPoint = Point.bind(emptyObj, 0/*x*/);
// не поддерживается полифилом, приведённым ниже,
// но отлично работает с родным bind:
var YAxisPoint = Point.bind(null, 0/*x*/);

var axisPoint = new YAxisPoint(5);
axisPoint.toString(); // '0,5'

axisPoint instanceof Point; // true
axisPoint instanceof YAxisPoint; // true
new Point(17, 42) instanceof YAxisPoint; // true
```

Обратите внимание, что вам не нужно делать ничего особенного для создания привязанной функции, используемой с оператором {{jsxref("Operators/new", "new")}}. В итоге, для создания явно вызываемой привязанной функции, вам тоже не нужно делать ничего особенного, даже если вам требуется, чтобы привязанная функция вызывалась только с помощью оператора {{jsxref("Operators/new", "new")}}.

```js
// Пример может быть запущен прямо в вашей консоли JavaScript
// ...продолжение примера выше

// Всё ещё можно вызывать как нормальную функцию
// (хотя обычно это не предполагается)
YAxisPoint(13);

emptyObj.x + ',' + emptyObj.y;
// >  '0,13'
```

Если вы хотите поддерживать использование привязанной функции только с помощью оператора {{jsxref("Operators/new", "new")}}, либо только с помощью прямого вызова, целевая функция должна предусматривать такие ограничения.

### Пример: создание сокращений

Метод `bind()` также полезен в случаях, если вы хотите создать сокращение для функции, требующей определённое значение `this`.

Возьмём, например, метод {{jsxref("Array.prototype.slice")}}, который вы можете использовать для преобразования массивоподобного объекта в настоящий массив. Вы можете создать подобное сокращение:

```js
var slice = Array.prototype.slice;

// ...

slice.call(arguments);
```

С помощью метода `bind()`, это сокращение может быть упрощено. В следующем куске кода `slice` является функцией, привязанной к функции {{jsxref("Function.prototype.call()", "call()")}} объекта {{jsxref("Function.prototype")}}, со значением `this`, установленным в функцию {{jsxref("Array.prototype.slice()", "slice()")}} объекта {{jsxref("Array.prototype")}}. Это означает, что дополнительный вызов `call()` может быть устранён:

```js
// Тоже самое, что и slice в предыдущем примере
var unboundSlice = Array.prototype.slice;
var slice = Function.prototype.call.bind(unboundSlice);

// ...

slice(arguments);
```

## Полифил

Функция `bind` является дополнением к стандарту ECMA-262 5-го издания; поэтому она может присутствовать не во всех браузерах. Вы можете частично обойти это ограничение, вставив следующий код в начало ваших скриптов, он позволяет использовать большую часть возможностей `bind()` в реализациях, не имеющих его родной поддержки.

```js
if (!Function.prototype.bind) {
  Function.prototype.bind = function(oThis) {
    if (typeof this !== 'function') {
      // ближайший аналог внутренней функции
      // IsCallable в ECMAScript 5
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }

    var aArgs = Array.prototype.slice.call(arguments, 1),
        fToBind = this,
        fNOP    = function() {},
        fBound  = function() {
          return fToBind.apply(this instanceof fNOP && oThis
                 ? this
                 : oThis,
                 aArgs.concat(Array.prototype.slice.call(arguments)));
        };

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();

    return fBound;
  };
}
```

Некоторые из многих отличий (так же могут быть и другие, данный список далеко не исчерпывающий) между этой реализацией и реализацией по умолчанию:

- Частичная реализация предполагает, что методы {{jsxref("Array.prototype.slice()")}}, {{jsxref("Array.prototype.concat()")}}, {{jsxref("Function.prototype.call()")}} и {{jsxref("Function.prototype.apply()")}} являются встроенными, имеют своё первоначальное значение.
- Частичная реализация создаёт функции, не имеющие неизменяемых свойств «отравленной пилюли» — {{jsxref("Function.caller", "caller")}} и `arguments` — которые выбрасывают исключение {{jsxref("Global_Objects/TypeError", "TypeError")}} при попытке получить, установить или удалить эти свойства. (Такие свойства могут быть добавлены, если реализация поддерживает {{jsxref("Object.defineProperty")}}, либо частично реализованы \[без поведения исключение-при-попытке-удаления], если реализация поддерживает расширения [`Object.prototype.__defineGetter__()`](/ru/docs/Web/JavaScript/Reference/Global_Objects/Object/__defineGetter__) и [`Object.prototype.__defineSetter__()`](/ru/docs/Web/JavaScript/Reference/Global_Objects/Object/__defineSetter__).)
- Частичная реализация создаёт функции, имеющие свойство `prototype`. (Правильная привязанная функция его не имеет.)
- Частичная реализация создаёт привязанные функции, чьё свойство {{jsxref("Function.length", "length")}} не соответствует с определением в ECMA-262; оно равно 0, в то время, как полная реализация, в зависимости от значения свойства `length` целевой функции и количества предопределённых аргументов, может вернуть значение, отличное от нуля.

Если вы решили использовать частичную реализацию, **не рассчитывайте на корректную работу в тех случаях, когда реализация отклоняется от спецификации ECMA-262 5-го издания!** Однако, в определённых случаях (и, возможно, с дополнительными модификациями для отдельных нужд), применение данной частичной реализации может быть вполне оправданным до тех пор, пока `bind()` не станет широко реализован в соответствии со спецификацией.

## Спецификации

{{Specifications}}

## Совместимость с браузерами

{{Compat}}

## Смотрите также

- {{jsxref("Function.prototype.apply()")}}
- {{jsxref("Function.prototype.call()")}}
- {{jsxref("Functions_and_function_scope", "Функции и их область видимости", "", 1)}}
