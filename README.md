Руководство по языку Jsonata
============================

- [Введение](#Jsonata-Description)
- [Рекомендуемая литература](#Jsonata-Links)

Введение
--------
Данное руководство появилось при описании грамматики языка JSONata для плагина Dochub.

Руководство содержит краткое изложение оригинальной документации https://docs.jsonata.org/.
Примеры призваны ускорить погружение в язык jsonata и помочь в описании архитектуры при использовании инструмента
Dochub.

JSONata появилась в 2016 году под руководством Andrew Coleman и его коллегами!
Данный язык призван помочь при работе с большими JSON-объектами, для их переформатирования и реструктуризации.
Данный языка позволят создавать сложные запросы, выраженные в компактной и интуитивно понятной форме. JSONata
включает в себе лучшие подходы которые используются в SQL, XPath и XQuery, что делает его универсальным и
выразительным языком для работы со структурами JSON. Результаты запросов можно отформатировать в любую структуру вывода
JSON, используя знакомый синтаксис объектов JSON.
В сочетании с возможностью создания пользовательских функций можно создавать расширенные выражения для обработки
любых запросов и задач преобразования JSON, например:

- Манипулировать строками;
- Извлечение значений;
- Создание сложных структур вывода JSON-объекта;
- позволяет осуществлять выборку, фильтрацию, сортировку;

По своей сути JSONata — это легкий язык запросов и преобразования данных.

Исходный пример описания сущности прикладного компонента архитектуры, который используется на протяжении всего
руководства.

```json
{
  "title": "Компонент прикладной архитектуры",
  "config": {
    "root_menu": "Архитектура/Прикладная"
  },
  "description": "Прикладные компоненты являются составными частями прикладного сервиса",
  "schema": {
    "type": "object",
    "additionalProperties": false,
    "seaf.app.sber.component_type": {
      "title": "Тип компонента",
      "type": "string",
      "default": "component",
      "enum": [
        "component",
        "service"
      ]
    },
    "seaf.app.sber.profile": {
      "title": "Базовое описание компонента",
      "type": "object",
      "properties": {
        "id": {
          "title": "Уникальный идентификатор компонента",
          "type": "string",
          "format": "uuid"
        }
      }
    },
    "live_stage": {
      "title": "Этап жизненного цикла",
      "enum": [
        "Эскиз",
        "В разработке / приобретение",
        "Внедрение / Не введена в эксплуатацию",
        "Опытная эксплуатация",
        "Промышленная эксплуатация",
        "Архивная"
      ]
    }
  }
}
```

Создание запросов
-----------------

JSON объекты представляется как ассоциативный массив, например: map.
Для того чтобы получить доступ к нужному элементу объекта, используйте оператор '.'.
Например:

```jsonata
entities.components.title

```
```text
Оператор . позволяет осуществлять доступ к элементам нашего объекта
```

Вывод:

На выходе получаем тип string.
```json
  "Компонент прикладной архитектуры" 
```

В зависимости от запроса может возвращаться числовой тип, null, undefined или массив и т.д.

Если запрашиваемого значения нет, то вернется null.

Если требуется обратиться к элементам массива, то вы можете это сделать следующим образом:

```jsonata
    entities.components.schema.'$defs'.'seaf.app.sber.component_type'.enum[-1]
```

Элементы массива индексируются с 0, также есть возможность обращаться к элементам в обратном порядке, 
например: [-1] и т.д. Обратите внимание, если свойство объекта содержит пробелы, ключевые слова языка 
jsonata или составное имя, то его требуется экранировать через ковычки ''.

```text
    entities...enum[[0..1]] - в качестве индекса массива, можно использовать диапазоны
    [0..1] - диапазон значений от 0 до 1, интервалы включены.
```

Если у нас в качестве объекта определен массив, например:

```json
[
  {
    "object 1": [
      1,
      2
    ]
  },
  {
    "object 1": [
      3,
      4
    ]
  }
]
```

В этом случае у нас нет имени свойства и мы не можем обратиться к элементу нашего массива!
Для того чтобы получить доступ к элементам массива требуется сделать следующее:

```jsonata
$[0] /* Оператор '$' - получить доступ ко всему документу */
```

Вывод:
```json
1
```

```jsonata
$[0].'object 1' /*Получили плоскую структуру.*/
```

Вывод:
```json
[
  1,
  2,
  3,
  4
]
```

```text
JSonata позволяет работать с большими входными объектами и эффективно
трансформировать в нужные нам объект.
Jsonata предоставляет возможность создания новых объектов
```

```jsonata
{
    "my-object": entities /* где entities поле существующего объекта */
}
```

```text
Мы можем добавлять новые поля
```

```jsonata
{
    "my-object": entities,
    "a_new_property": "a value"
}
```

```text
Создание новой структуры объекта
```

```jsonata
{
    "data": {
        "device": components,
        "when": entities
    }
}
```

Предикаты
---------

```json
{
  "Phone": [
    {
      "type": "home",
      "number": "0203 544 1234"
    },
    {
      "type": "office",
      "number": "01962 001234"
    },
    {
      "type": "office",
      "number": "01962 001235"
    },
    {
      "type": "mobile",
      "number": "077 7700 1234"
    }
  ]
}
```

Примеры:

```jsonata
Phone[type='office'].number /* [type='office'] - указываем предикат, для фильтрации элементов массива*/
```

```text
[выражение] - если выражение возвращает true, то значение остается, если false, то значение удаляется из результирующей выборки.
```

Вывод
```json
[
  "01962 001234",
  "01962 001235"
]
```

```text
Если при фильтрации элементов массива возвращается один элемент, но мы хотим в результате получить массив, то можно сделать следующееЖ
```

```jsonata
Phone[type='home'][] /*[] можно использовать префексную форму, так и постфиксную, например: Phone[][type='home']*/
```

Вывод:
```json
[
  {
    "type": "home",
    "number": "0203 544 1234"
  }
]
```

Использование Wildcards

```json
{
  "live_stage": {
        "title": "Этап жизненного цикла",
        "enum": [
                   "Эскиз",
                   "В разработке / приобретение",
                   "Внедрение / Не введена в эксплуатацию",
                   "Опытная эксплуатация",
                   "Промышленная эксплуатация",
                   "Архивная"
                  ]
  }
}
```

```jsonata
    live_stage.* /* получить значения всех полей объекта live_stage */
```

Вывод:
```json
[
  "Этап жизненного цикла",
  "Эскиз",
  "В разработке / приобретение",
  "Внедрение / Не введена в эксплуатацию",
  "Опытная эксплуатация",
  "Промышленная эксплуатация",
  "Архивная"
]
```

Получение значений всех дочерних объектов

```jsonata
    *.components
```

Для получения значений объекта на произвольной глубине вложенности можно использовать Multi-level wildcard!
```jsonata
    **.enum
```
Вывод:

```json
[
  "component",
  "service",
  "Централизованная",
  "Нецентрализованная",
  "Отсутствует",
  "Mission Critical",
  "Business Critical",
  "Business Operational",
  "Office Productivity",
  "Эскиз",
  "В разработке / приобретение",
  "Внедрение / Не введена в эксплуатацию",
  "Опытная эксплуатация",
  "Промышленная эксплуатация",
  "Архивная"
]
```

Преобразование данных с помощью функций и выражений
---------------------------------------------------
```text
Литерал - это то что мы можем выразить односложно, например: число, строка.

Строковые литералы в jsonata создаются следующим образом: "тестовая строка" или 'тестовая строка'.

```

```jsonata
entities.components.title & ' AC'
```
Вывод
```json
"Компонент прикладной архитектуры AC"
```

```jsonata
entities.components.schema.(type & ' ' & additionalProperties)
```

Вывод
```json
"object false"
```

```text
Язык jsonata поддерживает работу с числами как и во многих других языках.
    + сложение
    - вычитание
    * умножение
    / деление
    % деление по модулю
Операции сравнения:
    = equals
    != not equals
    < less than
    <= less than or equal
    > greater than
    >= greater than or equal
    in value is contained in an array
Логические выражения:
    and
    or  
```

Формирование структуры результирующего объекта
-------------------------------------

```text
На языке jsonata можно формировать выходные объекты различного типа, например:
    - string - 'Привет Мир!'
    - number - 23.5
    - Boolean - true, false
    - null - null
    - object - {"key1": "value1"}
    - array - ["value1", value2]

Рассмотрим 2 варианта, часто встречающихся на практике:

В качестве примера возьмем простой объект
```
```json
{
  "Email": [
    {
      "type": "work",
      "address": [
        "fred.smith@my-work.com",
        "fsmith@my-work.com"
      ]
    },
    {
      "type": "home",
      "address": [
        "freddy@my-social.com",
        "frederic.smith@very-serious.com"
      ]
    }
  ]
}
```

```text
1) Создание массивов  
```
```jsonata
Email.address
```
Вывод:

```json
[
  "fred.smith@my-work.com",
  "fsmith@my-work.com",
  "freddy@my-social.com",
  "frederic.smith@very-serious.com"
]
```
Видим, что 2 массива объединяются в один
Если мы хотим, чтобы каждный элемент находился в отдельном массиве, то нужно сделать следующее

```jsonata
Email.[address]
```

Вывод:

```json
[
  [ "fred.smith@my-work.com",  "fsmith@my-work.com" ],
  [ "freddy@my-social.com", "frederic.smith@very-serious.com" ]
]
```
Можем помещать различные элементы нашего объекта помещать в массив, например:

```jsonata
[property1, property2].Object
```

```text
1) Создание объектов 
```
```jsonata
Phone.{type: number}
```
Вывод:

```json
[
  { "home": "0203 544 1234" },
  { "office": "01962 001234" },
  { "office": "01962 001235" },
  { "mobile": "077 7700 1234"  }
]
```

Комбинирование пар ключ/значение в одно объекте

```jsonata
Phone{type: number}
```

Вывод:

```json
{
  "home": "0203 544 1234",
  "office": [
    "01962 001234",
    "01962 001235"
  ],
  "mobile": "077 7700 1234"
}
```

Комбинирование пар ключ/значение в одно объекте и группирование всех чисел в массиве

```jsonata
Phone{type: number[]}
```

Вывод:

```json
{
  "home": [
    "0203 544 1234"
  ],
  "office": [
    "01962 001234",
    "01962 001235"
  ],
  "mobile": [
    "077 7700 1234"
  ]
}
```

Композиция запросов
-------------------

```text
 В JSONata все является выражением.
 Выражение состоит из значений, функций и операторов, которые при вычислении дают результирующее значение.
 Функции и операторы применяются к значениям, которые сами могут быть результатами вычисления подвыражений. 
 
 При арифметических опрерациях возможно задавать приоритет с помощью круглых скобок, например:
        
        (5 + 7) * 3
 
 Для более сложных выражений можно использовать следующий подход:       
            
            Product.(Price * Quantity) - Price b Quantity являются свойствами объека Product
            
 Можно выделять в блок набор выражений разделенных точкой с запятой, например:
            (expr1; expr2; expr3) - каждое выражение в блоке вычисляется последовательно, результат последнего выражения возвращается как результат блока.
```


Сортировка, Группировка и Агрегация данных
------------------------------------------

```text
Язык предоставляет несколько способов для сортировки массивов.
    1) Использование функции $sort()
    2) order-by operator
    
Группировка данных



Account.Order.Product{`Product Name`: Price}

Result	
{
  "Bowler Hat": [ 34.45, 34.45 ],
  "Trilby hat": 21.67,
  "Cloak": 107.99
}

Account.Order.Product {
  `Product Name`: {"Price": Price, "Qty": Quantity}
}
Result	
{
  "Bowler Hat": {
    "Price": [ 34.45, 34.45 ],
    "Qty": [ 2, 4 ]
  },
  "Trilby hat": { "Price": 21.67, "Qty": 1 },
  "Cloak": { "Price": 107.99, "Qty": 1 }
}
```







Операторы
---------


String Functions
Numeric Functions
Aggregation Functions
Boolean Functions
Array Functions
Object Functions
Date/Time Functions
Higher Order Functions

Функции высших порядков
-----------------------



```text
Jsonata позволяет создавать функции

Для создании функии нужно придерживаться простых правил
    - указывать область видимисти с помощью ().
```

```jsonata
(
    $localizeTemperature := function($degrees_celsius, $country) {(
        $conversion_ratio := (9 / 5);
        ($country = "US") ? (($degrees_celsius * $conversion_ratio) + 32)
                : $degrees_celsius;
        )};

    {
        "temp": $localizeTemperature(body.temp, best_country)
    }
)
```
Инструкция - это синтаксическая единица языка, выражающее действие.




Расширение JSONata
------------------



Функциональное программирование
-------------------------------

Регулярные выражения
--------------------

Работа с Date/Time
------------------

Операторы
---------
Path Operators
Numeric Operators
Comparison Operators
Boolean Operators
Other Operators


Обработка модели
----------------




Библиотека Функций
------------------
<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;
  overflow:hidden;padding:10px 5px;word-break:normal;}
.tg th{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;
  font-weight:normal;overflow:hidden;padding:10px 5px;word-break:normal;}
.tg .tg-wzsp{font-size:14px;font-style:italic;font-weight:bold;text-align:left;vertical-align:top}
.tg .tg-62xo{font-size:14px;font-weight:bold;text-align:center;vertical-align:top}
.tg .tg-13pz{font-size:18px;text-align:center;vertical-align:top}
.tg .tg-6nwz{font-size:14px;text-align:center;vertical-align:top}
.tg .tg-ltad{font-size:14px;text-align:left;vertical-align:top}
.tg .tg-6t3r{font-style:italic;font-weight:bold;text-align:left;vertical-align:top}
.tg .tg-0lax{text-align:left;vertical-align:top}
</style>
<table class="tg">
<thead>
  <tr>
    <th class="tg-13pz"><span style="font-weight:bold">Функция</span></th>
    <th class="tg-13pz"><span style="font-weight:bold">Описание</span></th>
    <th class="tg-13pz"><span style="font-weight:bold">Пример</span></th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-6nwz" colspan="3"><span style="font-weight:bold">Функции работы со строками</span></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$string()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$length()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$substring()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$substringBefore()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$substringAfter()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$uppercase()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$lowercase()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$trim()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$pad()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$contains()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$split()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$join()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$match()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$replace()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$eval()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$base64encode()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$base64decode()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$encodeUrlComponent()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$encodeUrl()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$decodeUrlComponent()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$decodeUrl()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-62xo" colspan="3">Функции работы с числами</td>
  </tr>
  <tr>
    <td class="tg-wzsp">$number()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$abs()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$floor()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$ceil()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$round()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$power()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$sqrt()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$random()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$formatNumber()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$formatBase()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$formatInteger()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$parseInteger()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-6nwz" colspan="3"><span style="font-weight:bold">Агрегатные функции</span></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$sum()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$max()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$min()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$average()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-62xo" colspan="3">Логические функции</td>
  </tr>
  <tr>
    <td class="tg-wzsp">$boolean()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$not()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$exists()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-62xo" colspan="3">Функции работы с массивами</td>
  </tr>
  <tr>
    <td class="tg-wzsp">$count()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$append()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$sort()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$reverse()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$shuffle()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$distinct()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$zip()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-62xo" colspan="3">Функции работы с объектами</td>
  </tr>
  <tr>
    <td class="tg-wzsp">$keys()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$lookup()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$spread()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$merge()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$sift()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$each()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$error()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$assert()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$type()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-62xo" colspan="3">Date/Time функции</td>
  </tr>
  <tr>
    <td class="tg-wzsp">$now()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$millis()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$fromMillis()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$toMillis()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-62xo" colspan="3">Функции высшего порядка</td>
  </tr>
  <tr>
    <td class="tg-wzsp">$map()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$filter()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$single()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-wzsp">$reduce()</td>
    <td class="tg-ltad"></td>
    <td class="tg-ltad"></td>
  </tr>
  <tr>
    <td class="tg-6t3r">$sift()</td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
  </tr>
</tbody>
</table>


Dochub примеры
--------------

```jsonata
(
        /* Определяем шаблон для идентификаторов */
        $isl2 := /^([^\.]*)(\.([^\.]*))?$/;
        /* Генерируем список компонентов L1 ровня */
        $result := $mergedeep($.components.$spread().(
          $id := $keys()[0];
          $node := $.*;
          $levels := $isl2($id).groups;
          /* Формируем ноду */
          $node :=
            /* Если это компонент уровня L2 */
            $levels[2]
            /* Создаем виртуальный компонент L1 с информацией о связях */
            ? $merge([
              {
                $levels[0]: {
                  "links":
                    $distinct($node.links.{
                      /* Преобразуем идентификатор связи к L1 */
                      "id": $split(id, ".")[0],
                      "title":  title,
                      "direction": "--"
                    })[id != $levels[0]]
                }
              }
            ])
            /* Иначе, преобразуем связи компонента к уровню L1 */
            : $merge([
              {
                $id: $merge([$.*, {
                  "links":
                    $distinct($node.links.{
                      /* Преобразуем идентификатор связи к L1 */
                      "id": $split(id, ".")[0],
                      "title":  title,
                      "direction": "--"
                    })[id != $levels[0]]
                }])
              }
            ])
        ));
        /* Перезаписываем данные манифеста новыми компонентами */
        $merge([$,
          { "components": $result}
        ]);
)
```

```jsonata
(
        $business_entities := $.business_entities;
        $makeLocation := function($id) {(
            $arrleft := function($arr ,$count) {
                $map($arr, function($v, $i) {
                $i <= $count ? $v
                })
            };
            $domains := $split($id, ".");
            "Документы/Моё болото/Бизнес-сущности/" & $join($map($domains, function($domain, $index) {(
                $lookup(business_entities, $join($arrleft($domains, $index), ".")).title
            )}), "/");
        )};
        $append(
        [{
            "icon": *.icon,                                                 /* Получаем иконку */
            "link": "entities/business_entities/business_entities_list",
            "location": "Документы/Моё болото/1. Бизнес-сущности"
        }],
        [{
            "icon": *.icon,                                                 /* Получаем иконку */
            "link": "entities/business_entities/business_entities_in_systems",
            "location": "Документы/Моё болото/2. Список бизнес-сущностей в системах"
        }]
        );

        $entities := [$.business_entities.$spread().$merge([$.*, {"id": $keys($)}])];
        $entities [id=$params.id];
        $[system_id=$params.system_id]
)
```

```jsonata
(
        [{
          /* Размещение в пользовательском меню */
          "location": "C4 Model",
          /* URI представления context см. ниже.*/
          "link": "entities/c4model/context"
        }];

       /* В переменной $parent содержатся переданные параметры из URI */
       $parent_id := $params.parent;
       /* Определяем уровень диаграммы */
       $levels := $split($parent_id, ".");
       $level := $count($levels);
       /* Сохраняем корень данных */
       $c4model := c4model;
       /* Получаем компонент, который ходим раскрыть. Для L1 = undefined */
       $parent := $lookup($c4model, $parent_id);
       /* Генерируем шаблон для выбора компонентов диаграммы */
       $pattern := $eval("/^" & ($parent_id != "" ? $parent_id & "." : "") & "[a-zA-Z0-9\\_]*$/");

       /* Обходим все компоненты сущности C4Model */
       $nodes := $c4model.$spread().(
         $id := $keys()[0];
         /* Обрабатываем компонент, если идентификатор компонента соответствует искомому */
         $match($id, $pattern) ? $merge([$.*, {
           /* Сохраняем идентификатор компонента в массиве компонентов диаграммы */
           "$id": $id
         }])
       );

       /* Находим связь выбранных копонентов с внешними */
       $extraNodes := $distinct([$nodes.links.(
         $not($exists($match(id, $pattern))) ? $merge([
           $lookup($c4model, id),
           {
             "$id": id,
             "links": [],
             "boundary": "*"
           }
         ])
       )]);

       /* Генерируем код Mermaid компонентов */
       $node2code := function($nodes, $bid) {(
         $join([$nodes[$bid ? boundary = $bid : $not($exists(boundary))].(
           /* Формируем заголовок компонента */
           $title := $level < 2
               /* Если уровень выше L2, даем возможность "провалиться" глубже */
               ? "![" & title & "](/entities/c4model/context?parent=" & $."$id" & ")"
               /* иначе нет */
               : title;
           entity & "(" & $."$id" & ", \"" & $title & "\", \"" & description & "\")"
         )], "\n") & "\n"
       )};

       /* Генерируем код Mermaid границ и их наполнения */
       $components := function($parent) {(
         $pattern := $eval("/^" & $parent & "[a-zA-Z0-9\\_]*$/");
         $boundaries := $distinct($nodes.boundary)^(boundary);
         $join($boundaries[$match($, $pattern)].(
           "Boundary(" & $ & ", \"" & $ & "\") {\n"
           & $node2code($nodes, $)
           & $join($components($ & "."), "\n")
           & "\n}"
         ), "\n") & "\n"
       )};

       /* Парсим связи компонентов */
       $relations := function() {(
         $join([$nodes.(
             $from := $."$id";
             links.{
               "from": $from,
               "to": id,
               "direction": direction,
               "title": title,
               "description": description
             }
           )].(
             (direction = "<->" ? "BiRel" : "Rel")
             & "("
             & from
             & ", " & to
             & ", \"" & title & "\""
             & ", \"" & description & "\""
             & ")"
           )
         , "\n") & "\n"
       )};

       /* Генерируем код диаграммы Mermaid */
       $code := function () {(
         /* Сначала выводим вгешние компонент обнаруженные в связях */
         $node2code($extraNodes, "*")
         /* Если уровень глубже L1, создаем контейнер */
         & ($level > 0 ? "Container_Boundary(" & $parent_id & ", \"\") {\n" : "")
         /* Выводим обнаруженные без областей */
         & $node2code($nodes)
         /* Выводим обнаруженные компоненты разложенные в области */
         & $components("")
         /* Если нужно, завершаем контейнер */
         & ($level > 0 ? "}\n" : "")
         /* Генерируем связи компонентов на диаграмме */
         & $relations()
       )};

       /* Возвращаем результирующие данные */
       {
         /* Определяем уровень представления */
         "notation": $lookup({
           "0": "C4Context",
           "1": "C4Container",
           "2": "C4Component"
         }, $string($level)),

         /* Генерируем код */
         "code":
             /* Если количество компонентов на диаграмме есть */
             $count($nodes) > 0
             /* выводом */
             ? $code()
             /* иначе сообщаем, что внутри пусто */
             : "\nBoundary(bempty, \"\") {\nSystem(sempty, \"Здесь пусто\")\n}"
             ,

         /* Генерируем заголовок диаграммы */
         "title":
           /* Если есть компонент верхнего уровня */
           $parent
           /* Возвращаем его название */
           ? $parent.title
           /* Иначе идентификатор, а если он пустой, то считаем диаграмму L1 */
           : ($parent_id ? $parent_id : "Context")
       }
)
```


Рекомендуемая литература
------------------------

1. "Jsonata: Query and Transformation Language for JSON Data".
2. "Mastering JSONata" by Bikram Kundu.
3. "JSON At Work" by Tom Marrs.
4. "Querying JSON with JSONata".
5. "Introducing JSONata – A Query and Transformation Language for JSON".

Авторы
-----
- Николай Темняков
- Илья Караваев