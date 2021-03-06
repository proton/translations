# Итак, вы хотите научиться функциональному программированию (Часть 2)

*Перевод статьи [Charles Scalfani](https://medium.com/@cscalfani): [So You Want to be a Functional Programmer (Part 2)](https://medium.com/@cscalfani/so-you-want-to-be-a-functional-programmer-part-2-7005682cec4a) с [наилучшими пожеланиями от автора](https://twitter.com/cscalfani/status/933052963781722112).*

![Эволюция парадигм программирования](https://cdn-images-1.medium.com/max/800/1*AM83LP9sGGjIul3c5hIsWg.png)

Первый шаг к пониманию идей функционального программирования – самый важный и иногда самый сложный шаг. Но с правильным подходом никаких трудностей быть не должно.

Предыдущая часть: [Часть 1](https://medium.com/devschacht/charles-scalfani-so-you-want-to-be-a-functional-programmer-part-1-6ef98e90d58d).

## Дружеское напоминание

![Палец с бантиком](https://cdn-images-1.medium.com/max/800/1*RYgiClt6s_Xj9OUK9qapIw.png)

Пожалуйста, читайте код медленно. Перед тем, как двигаться дальше, убедитесь, что вы всё поняли. Каждая следующая часть главы отталкивается от предыдущей.

Если вы будете спешить, то наверняка упустите некоторые нюансы, которые могут быть важны в дальнейшем.

## Рефакторинг

![Если бы человек был рефакторингом...](https://cdn-images-1.medium.com/max/800/1*_GBlt7_8aD19rxHh6f2Uow.png)

Давайте чуть-чуть подумаем о рефакторинге. Вот пример JavaScript-кода:

```js
function validateSsn(ssn) {
    if (/^\d{3}-\d{2}-\d{4}$/.exec(ssn))
        console.log('Valid SSN');
    else
        console.log('Invalid SSN');
}

function validatePhone(phone) {
    if (/^\(\d{3}\)\d{3}-\d{4}$/.exec(phone))
        console.log('Valid Phone Number');
    else
        console.log('Invalid Phone Number');
}
```

Мы все писали такой код и лишь потом начинали понимать, что эти две функции практически одинаковые и отличаются только несколькими моментами (выделены **жирным** шрифтом).

Вместо копирования `validateSsn` и последующим её редактированием для получения `validatePhone`, нам лучше создать отдельную функцию и параметризировать данные.

В этом примере мы будет параметризировать ***входное значение***, ***регулярное выражение*** и ***сообщение*** (по крайней мере, последнюю его часть).

Код после рефакторинга:

```js
function validateValue(value, regex, type) {
    if (regex.exec(value))
        console.log('Invalid ' + type);
    else
        console.log('Valid ' + type);
}
```

Параметры `ssn` и `phone` из старого примера теперь представлены как `value`.

Регулярные выражения `/^\d{3}-\d{2}-\d{4}$/` и `/^\(\d{3}\)\d{3}-\d{4}$/` – как `regex`.

И наконец, последние части сообщения `'SSN'` и `'Phone Number'` – как `type`.

Всегда лучше иметь одну функцию вместо двух. Или и того хуже: трёх, четырёх или десяти функций. Это делает ваш код чистым и удобным в поддержке.

Для примера: если возникает ошибка, вам нужно исправить её в одном-единственном месте в сравнении с тем, чтобы искать по всему исходному коду, куда эта функция МОГЛА БЫТЬ вставлена и переделана.

Но что происходит, когда у нас появляется следующая ситуация:

```js
function validateAddress(address) {
    if (parseAddress(address))
        console.log('Valid Address');
    else
        console.log('Invalid Address');
}

function validateName(name) {
    if (parseFullName(name))
        console.log('Valid Name');
    else
        console.log('Invalid Name');
}
```

Здесь `parseAddress` и `parseFullName` – функции, принимающие строку и возвращающие `true`, если она парсится.

Как произвести рефакторинг в этом случае?

Что ж, мы можем использовать `value` для `adress` и `name` и `type` для `'Address'` и `'Name'`, как мы делали это раньше, но вместо регулярного выражения здесь функция.

Единственный выход – передавать функцию параметром...

## Функции высшего порядка

![Функции высшего порядка, как матрёшки](https://cdn-images-1.medium.com/max/800/1*hZyWFJAiDDiqci0ygBLeoA.png)

Многие языки не поддерживают передачу функций в качестве параметров. Некоторые – поддерживают, но проще от этого не становится.

> *В функциональном программировании функция – это полноправный гражданин языка. Иными словами, функция – всего лишь другое значение.*

Пока функции являются просто значениями, мы можем передавать их как параметры.

Хотя JavaScript – это не чистый функциональный язык, вы можете выполнять с помощью него некоторые функциональные операции. Вот последние две функции, преобразованные в одну отдельную с помощью передачи ***функции-парсера*** в качестве параметра, называющегося `parseFunc`.

```js
function validateValueWithFunc(value, parseFunc, type) {
    if (parseFunc(value))
        console.log('Invalid ' + type);
    else
        console.log('Valid ' + type);
}
```

Наша новая функция называется ***функцией высшего порядка***.

> *Функции высшего порядка либо принимают функции как параметры, либо возвращают их, либо и то, и другое одновременно.*

Теперь мы можем вызвать нашу функцию высшего порядка для четырёх предыдущих функций (это работает в JavaScript, потому что `Regex.exec` возвращает истинное значение, если находится совпадение):

```js
validateValueWithFunc('123-45-6789', /^\d{3}-\d{2}-\d{4}$/.exec, 'SSN');
validateValueWithFunc('(123)456-7890', /^\(\d{3}\)\d{3}-\d{4}$/.exec, 'Phone');
validateValueWithFunc('123 Main St.', parseAddress, 'Address');
validateValueWithFunc('Joe Mama', parseName, 'Name');
```

Это гораздо лучше, чем иметь четыре практически идентичных функций.

Но обратите внимание на регулярные выражения. Они немного пространные. Давайте приведём наш код в порядок, реорганизовав его таким образом:

```js
var parseSsn = /^\d{3}-\d{2}-\d{4}$/.exec;
var parsePhone = /^\(\d{3}\)\d{3}-\d{4}$/.exec;

validateValueWithFunc('123-45-6789', parseSsn, 'SSN');
validateValueWithFunc('(123)456-7890', parsePhone, 'Phone');
validateValueWithFunc('123 Main St.', parseAddress, 'Address');
validateValueWithFunc('Joe Mama', parseName, 'Name');
```

Так-то лучше. Теперь, когда мы хотим пропарсить телефонный номер, нам не нужно копировать и вставлять регулярные выражения.

Но представьте, что у нас гораздо больше регулярных выражений для парсинга, а не только `parseSsn` и `parsePhone`. Каждый раз, когда мы создаем парсер регулярным выражением, мы должны помнить о том, чтобы добавить `.exec` в конце. И уж поверьте мне, это легко забыть.

Мы можешь застраховаться от этого, создав функцию высшего порядка, возвращающую метод `exec`.

```js
function makeRegexParser(regex) {
    return regex.exec;
}

var parseSsn = makeRegexParser(/^\d{3}-\d{2}-\d{4}$/);
var parsePhone = makeRegexParser(/^\(\d{3}\)\d{3}-\d{4}$/);

validateValueWithFunc('123-45-6789', parseSsn, 'SSN');
validateValueWithFunc('(123)456-7890', parsePhone, 'Phone');
validateValueWithFunc('123 Main St.', parseAddress, 'Address');
validateValueWithFunc('Joe Mama', parseName, 'Name');
```

В этом примере `makeRegexParser` принимает регулярное выражение и возвращает метод `exec`, который в свою очередь принимает строку. `validateValueWithFunc` будет передавать строку, `value`, методу-парсеру, то есть `exec`.

`parseSsn` и `parsePhone` эффективны также, как и раньше, и также, как и метод `exec` регулярных выражений.

Честно говоря, это незначительное улучшение, но оно показано здесь, чтобы привести пример функции высшего порядка, возвращающей функцию (*прим. пер., методы - тоже функции*).

Несмотря на это, вы можете представить пользу от подобных изменений, если бы `makeRegexParser` была гораздо более комплексной.

Вот другой пример функции высшего порядка, возвращающей функцию:

```js
function makeAdder(constantValue) {
    return function adder(value) {
        return constantValue + value;
    };
}
```

Здесь у нас есть `makeAdder`, принимающая `constantValue` и возвращающая `adder` - функцию, которая будет добавлять значение-константу к любой переданной ей переменной.

Вот как её можно использовать:

```js
var add10 = makeAdder(10);
console.log(add10(20)); // печатает 30
console.log(add10(30)); // печатает 40
console.log(add10(40)); // печатает 50
```

Мы создаём функцию, `add10`, передавая константу `10` функции `makeAdder`, которая возвращает функцию, которая в свою очередь будет добавлять `10`.

Заметьте, что функция `adder` имеет доступ к `constantValue` даже после того, как `makeAdder` возвращает своё значение. Это потому, что `constantValue` уже находилась в её области видимости в тот момент, когда `adder` была создана.

Такое поведение очень значимо, так как без него функции, возвращающие функции, не были бы настолько полезными. Так что важно понимать, как они работают и как такое поведение называется.

А называется оно ***замыканием***.

## Замыкания

![Принцип замыкания в форме пирамиды](https://cdn-images-1.medium.com/max/800/1*0phT7qIAPVxG7KXcL-6B5g.png)

Вот специально придуманный пример функций, использующих замыкание:

```js
function grandParent(g1, g2) {
    var g3 = 3;
    return function parent(p1, p2) {
        var p3 = 33;
        return function child(c1, c2) {
            var c3 = 333;
            return g1 + g2 + g3 + p1 + p2 + p3 + c1 + c2 + c3;
        };
    };
}
```

В этом примере `child` имеет доступ к своим локальным переменным, к переменным `parent` и к переменным `grandParent`.

`parent` имеет доступ к своим переменным и к переменным `grandParent`.

`grandParent` имеет доступ только к своим переменным.

(Смотрите на пирамиду выше для ясности.)

Вот пример использования всего этого:

```js
var parentFunc = grandParent(1, 2); // возвращает parent()
var childFunc = parentFunc(11, 22); // возвращает child()
console.log(childFunc(111, 222)); // печатает 738
// 1 + 2 + 3 + 11 + 22 + 33 + 111 + 222 + 333 == 738
```

Здесь `parentFunc` хранит область видимости `parent`, потому что `grandParent` возвращает `parent`.

Таким же образом `childFunc` хранит область видимости `child`, потому что `parentFunc`, по сути являющийся `parent`, возвращает `child`.

Когда создана функция, все переменные в её области видимости в момент создания доступны ей на время жизни. Функция существует, пока на неё есть ссылка. Для примера, область видимости `child` существует, пока `childFunc` продолжает ссылаться на неё. 

> *Замыкание – область видимости функции, которая сохраняется благодаря ссылке на эту функцию.*

Обратите внимание, что замыкания в JavaScript – сомнительное удовольствие из-за изменяемости переменных, то есть из-за того, что они могут менять своё значение с момента определения и до тех пор, пока вызываемая функция не возвратится.

К счастью, переменные в функциональных языках программирования неизменяемые, что устраняет этот общий источник ошибок и неопределённости.

## Мой мозг!!!!

![Условная инструкция что делать, если вы вправду так сказали](https://cdn-images-1.medium.com/max/800/1*IK5485-iZaHeZRfP8aWmYg.png)

Пока что достаточно.

В последующих частях этой статьи я расскажу про функциональную композицию, каррирование, стандартные функции в функциональном программировании (такие как `map`, `filter`, `fold` и так далее) и ещё много о чём.

---

*Слушайте наш подкаст в [iTunes](https://itunes.apple.com/ru/podcast/%D0%B4%D0%B5%D0%B2%D1%88%D0%B0%D1%85%D1%82%D0%B0/id1226773343) и [SoundCloud](https://soundcloud.com/devschacht), читайте нас на [Medium](https://medium.com/devschacht), контрибьютьте на [GitHub](https://github.com/devSchacht), общайтесь в [группе Telegram](https://t.me/devSchacht), следите в [Twitter](https://twitter.com/DevSchacht) и [канале Telegram](https://t.me/devSchachtChannel), рекомендуйте в [VK](https://vk.com/devschacht) и [Facebook](https://www.facebook.com/devSchacht).*

[Статья на Medium](https://medium.com/devschacht/charles-scalfani-so-you-want-to-be-a-functional-programmer-part-2-ae095d9807b3)
