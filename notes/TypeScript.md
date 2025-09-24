---
created at: 2025-08-19
tags: " "
zero-links:
---
# Объектные типы (Object Types)

Типы объекта состоят из типов всех входящих в него свойств.

Выводятся типы автоматически:

```
// Тип: { firstName: string, pointsCount: number }
const user = {
  firstName: 'Mike',
  pointsCount: 1000,
};

// Поменять тип свойств нельзя
// Type 'number' is not assignable to type 'string'.
user.firstName = 7;
```

TypeScript не позволяет обращаться к несуществующим свойствам. Это значит, что структура любого объекта должна быть задана при его инициализации:

```
// Property 'age' does not exist on type '{ firstName: string, pointsCount: number; }'.
user.age = 100;
```

Чтобы принять такой объект в функцию как параметр, нужно указать его структуру в описании функции:

```
// Свойства в описании типа разделяются через запятую (,)
function doSomething(user: { firstName: string, pointsCount: number }) {
  // ...
}
```

Теперь внутрь можно передавать любой объект, который совпадает по свойствам:

```
doSomething({ firstName: 'Alice', pointsCount: 2000 });
doSomething({ firstName: 'Bob', pointsCount: 1800 });

// Так нельзя
doSomething({ firstName: 'Bob' });
// И так тоже
doSomething({ firstName: 'Bob', pointsCount: 1800, key: 'another' });
```

Как и в случае примитивных типов данных, ни null, ни undefined по умолчанию не разрешены. Чтобы изменить это поведение, нужно добавить опциональность:

```
// firstName может быть undefined
// pointsCount может быть null
function doSomething(user: { firstName?: string, pointsCount: number | null }) {
  // ...
}
```

Объекты могут быть полезными инструментами при разработке программного обеспечения.

# Перечисления (Enums)

Перечисления используют вместо строк для постоянных значений:

```
enum OrderStatus {
  Created,
  Paid,
  Shipped,
  Delivered,
}

const order = {
  items: 3,
  status: OrderStatus.Created,
};
```

Самый распространенный пример использования перечислений — хранение разных статусов. Но есть и другие случаи. Например, с их помощью хранят различные справочные данные и избавляются от магических значений:

- Направления движения
- Стороны света
- Дни недели
- Месяцы

```
enum CardinalDirection {
  North,
  South,
  East,
  West,
}

const direction = CardinalDirection.North;
```

Перечисление — это и значение, и тип. Его можно указывать как тип в параметрах функции:

```
setStatus(status: OrderStatus)
```

Также перечисления после компиляции превращаются в JavaScript-объект, в котором каждому значению соответствует свойство. У этого свойства есть тип `number` и начинается он с `0`:

```
const status = OrderStatus.Created;
console.log(status); // 0
```

Это позволяет в дальнейшем удобно использовать стандартные методы — например, `Object.keys` и `Object.values`:

```
const statuses = Object.keys(OrderStatus);
console.log(statuses); // ['0', '1', '2', '3', 'Created', 'Paid', 'Shipped', 'Delivered']
```

Среди ключей мы видим числа `'0', '1', '2', '3'`. Компилятор создает такие числовые ключи автоматически, а созданный объект выглядит так:

```
console.log(OrderStatus); // =>
// {
//   '0': 'Created',
//   '1': 'Paid',
//   '2': 'Shipped',
//   '3': 'Delivered',
//   'Created': 0,
//   'Paid': 1,
//   'Shipped': 2,
//   'Delivered': 3
// }
```

Но можно избавиться от создания дополнительных ключей, если указать строковые значения:

```
enum OrderStatus {
  Created = '0',
  Paid = '1',
  Shipped = '2',
  Delivered = '3',
}

const statuses = Object.keys(OrderStatus);
console.log(statuses); // ['Created', 'Paid', 'Shipped', 'Delivered']

console.log(OrderStatus); // =>
// {
//   'Created': '0',
//   'Paid': '1',
//   'Shipped': '2',
//   'Delivered': '3'
// }
```

# Псевдонимы типов (Type Aliases)
Представим программу, в которой есть объект пользователя. Этот объект используется повсеместно. В такой ситуации описание типа этого объекта будет повторяться в каждом определении функции:

```
function doSomething(user: { firstName: string, pointsCount: number }) {}
function doSomethingElse(user: { firstName: string, pointsCount: number }) {}
function doSomethingAnother(user: { firstName: string, pointsCount: number }) {}
```

Во-первых, здесь много дублирования. Во-вторых, значительно усложняется изменение структуры, так как придется руками править все места, где встречается это определение. В этом уроке разберем, как избежать таких проблем.

## Задаем псевдоним типа

Чтобы не делать одну и ту же работу, TypeScript позволяет задавать псевдоним для составных типов. Так мы не будем повторяться:

```
type User = {
  firstName: string;
  pointsCount: number;
}
```

Теперь можно провести замену во всех функциях:

```
function doSomething(user: User) {
  // ...
}
```

Псевдоним — это не создание нового типа данных. Это способ сокращенно записать определение типа. Поэтому следующие примеры будут работать без проблем:

```
const user = {
  firstName: 'Mike',
  pointsCount: 1000,
};

// Оба вызова работают
doSomething(user);
doSomething({ firstName: 'Bob', pointsCount: 1800 });
```

При этом разработчики на TypeScript говорят «создал тип», а не «создал псевдоним типа». Поэтому в этом курсе мы будем придерживаться общепринятого формата.

Типы можно задавать для любых типов данных, например, для простых:

```
type SomeType = string;
```

А также для составных:

```
// union тип из трех возможных значений
type SomeType = string | number | null;

// Функция
type Countable = (coll: number[]) => number
```

## Объекты и функции

Описание типа функции вне объекта и внутри отличается. Когда функция записывается самостоятельно, используется формат стрелочной функции:

```
type Countable = (coll: number[]) => number
```

Внутри типа, который описывает объект, формат меняется на используемый для обычных свойств:

```
type User = {
  firstName: string;
  pointsCount: number;
  count(coll: number[]): number;
}
```

Но это не касается колбеков, которые могут быть использованы внутри:

```
type User = {
  firstName: string;
  pointsCount: number;
  // Типы взяты для примера
  count(coll: (v: string) => string): number;
}
```