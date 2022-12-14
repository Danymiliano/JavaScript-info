
# Опциональная цепочка '?.'

[recent browser="new"]

Опциональная цепочка `?.` — это защищённый от ошибок способ доступа к свойствам вложенных объектов, даже если какое-либо из промежуточных свойств не существует.

## Проблема

Если вы только начали читать учебник и изучать JavaScript, то, возможно, для вас не знакома эта довольно распространённая проблема.

К примеру, наши пользователи могут указать свой адрес, но лишь немногие это делают. В таком случае мы не можем полагаться, что свойство `user.address.street` всегда будет иметь значение:

```js run
let user = {}; // пользователь без адреса

alert(user.address.street); // Ошибка!
```

Или, например, нужно получить данные об HTML-элементе, который может отсутствовать на странице:

```js run
// Произойдёт ошибка, если querySelector(...) равен null.
let html = document.querySelector('.my-element').innerHTML;
```

До появления `?.` в языке для решения подобного использовался оператор `&&`.

Например:

```js run
let user = {}; // пользователь без адреса

alert( user && user.address && user.address.street ); // undefined (без ошибки)
```

Использование логического И во всей цепочке свойств гарантирует, что у каждого из свойств есть значение, но это довольно длинная и громоздкая конструкция.

## Опциональная цепочка

Опциональная цепочка `?` останавливает вычисление и возвращает `undefined`, если часть перед `?.` имеет значение `undefined` или `null`.

**Для краткости в этой статье мы будем говорить, что что-то "существует", если его значение отличается от `null` или `undefined`.**

Это безопасный способ обратиться к свойству `user.address.street`:

```js run
let user = {}; // пользователь без адреса

alert( user?.address?.street ); // undefined (без ошибки)
```

Получение адреса с помощью конструкции `user?.address` выполняется без ошибок, даже если объекта `user` не существует:

```js run
let user = null;

alert( user?.address ); // undefined
alert( user?.address.street ); // undefined
```

Обратите внимание, что синтаксис `?.` делает необязательным только свойство перед ним, а не какое-либо последующее.

В приведённом выше примере конструкция `user?.` допускает, что переменная `user` может быть содержать `null/undefined`.

С другой стороны, если объект `user` существует, то в нём должно быть свойство `user.address`, иначе выполнение `user?.address.street` вызовет ошибку из-за второй точки.

```warn header="Не злоупотребляйте опциональной цепочкой"
Используйте `?` только тогда, когда это действительно нужно, т.е. если свойства может не быть.

Например, если в коде объект `user` будет точно определён, но его свойство `address` является необязательным, то предпочтительнее использовать следующую конструкцию: `user.address?.street`.

Если же переменная `user` по ошибке окажется необъявленной, мы узнаем об этом и исправим. В противном случае, такие ошибки будет проигнорированы, хотя их лучше бы исправить, потому что они в дальнейшем усложнят отладку.
```

````warn header="Переменная перед `?.` должна быть объявлена"
Если переменная `user` вообще даже не объявлена, то выражение `user?.anything` выдаст ошибку:

```js run
// ReferenceError: user is not defined
user?.address;
```
Поэтому, чтобы этого не было, необходимо определить переменную (`let/const/var user`). Опциональная цепочка работает только с существующими переменными.
````

## Сокращённое вычисление

Как уже говорилось, `?.` немедленно останавливает вычисление (т.е. выполняется по "сокращённой схеме"), если левой части не существует.

Таким образом, последующие вызовы функций или операции не будут выполнены: 

```js run
let user = null;
let x = 0;

user?.sayHi(x++); // ничего не произойдёт

alert(x); // 0, значение не было увеличено на единицу
```

## Другие случаи применения: ?.(), ?.[]

Опциональная цепочка `?.` — это не оператор, а специальная синтаксическая конструкция, которая также работает с функциями и квадратными скобками.

Например, `?.()` используется для вызова потенциально несуществующей функции.

В следующем примере не у всех пользователей есть метод `admin`:

```js run
let user1 = {
  admin() {
    alert("Я администратор");
  }
}

let user2 = {};

*!*
user1.admin?.(); // Я администратор
user2.admin?.();
*/!*
```

В обоих вызовах сначала используем точку `.`, чтобы получить свойство `admin`, потому что объект пользователя уже определён, соответственно к нему можно обратиться без какой-либо ошибки.

Затем уже `?.()` проверяет левую часть: если функция `admin` существует, то она выполнится (для `user1`). Иначе (для `user2`) вычисление остановится без ошибок.

Также существует синтаксис `?.[]`, если значение свойства требуется получить с помощью квадратных скобок `[]`, а не через точку `.`. Как и в остальных случаях, такой способ позволяет защититься от ошибок при доступе к свойству объекта, которого может не быть.

```js run
let user1 = {
  firstName: "Иван"
};

let user2 = null; // Представим, что пользователь не авторизован

let key = "firstName";

alert( user1?.[key] ); // Иван
alert( user2?.[key] ); // undefined

alert( user1?.[key]?.something?.not?.existing); // undefined
```

Кроме этого, `?.` можно совместно использовать с `delete`:

```js run
delete user?.name; // Удалить user.name, если пользователь существует
```

```warn header="Можно использовать `?.` для безопасного чтения и удаления, но не для записи"
Опциональная цепочка `?.` не имеет смысла в левой части присваивания:

```js run
// Идея следующего кода в том, чтобы присвоить значение свойству user.name, если пользователь существует.

user?.name = "John"; // Ошибка, это не сработает
// из-за того, что код был вычислен как undefined = "John"
```

## Итого

Синтаксис `?.` имеет три формы:

1. `obj?.prop` -- возвращает `obj.prop`, если существует `obj`, и `undefined` в противном случае.
2. `obj?.[prop]` -- возвращает `obj[prop]`, если существует `obj`, и `undefined` в противном случае.
3. `obj.method?.()` -- вызывает `obj.method()`, если существует `obj.method`, в противном случае возвращает `undefined`.

Как мы видим, все они просты и понятны в использовании. `?.` проверяет левую часть выражения на `null/undefined`, и останавливает её дальнейшее выполнение, если они не существуют.

Цепочка `?.` позволяет без возникновения ошибок обратиться к вложенным свойствам.

Тем не менее, нужно разумно использовать `?.` — только там, где это уместно, если левая часть не существует.

Чтобы таким образом не скрывать возможные ошибки программирования.
