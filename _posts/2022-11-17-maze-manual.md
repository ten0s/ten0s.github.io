---
layout: post
title: Головоломка Лабиринт - Системный подход
date: 2022-11-18
---

Как-то мне попалась [Головоломка Лабиринт](https://www.youtube.com/watch?v=Z-bjzPv-pLg).
Это была одна из тех задач, чтобы порешать ее несколько минут и забыть.
Я считаю, что такие задачи ничему не учат и если их решать бездумно методом подбора,
то это пустая трата времени.

Но у меня был отпуск, было особо нечем заняться и я подумал - что если решить
эту задачу системно по алгоритму, чтобы ответить на следующие вопросы:

1. Есть ли вообще такой путь?
2. Сколько таких путей?
3. Как научить других решать похожие задачи?

**Задание**:

Вы входите в лабиринт в зеленый вход, а выйти нужно из красного выхода,
при этом нельзя подряд пересекать две линии одного цвета.

![Maze Task Image](/assets/images/maze-manual/maze-task.png)

**Решение**:

Прежде всего, нужно обозначить линии и комнаты.

![Maze Tagged Image](/assets/images/maze-manual/maze-tagged.png)

Обратите внимание:

1. По условию задачи нельзя пересекать две линии одного цвета,
   т.е. мы может пересекать только определенные линии и только в одном направлении.
2. Допустим мы попали в комнату В из комнаты Б через зеленую линию 6.
   Мы не можем ни вернуться назад через линию 6, ни пересечь зеленые
   линии 7, 9 и 13. Мы может только пересечь отдельно стоящую красную линию 8.
   И если мы ее пересечем мы остаемся в той же комнате В, но в новом состоянии
   и теперь можем пересекать зеленые линии 6, 7, 9 и 13. Т.е. в каждой комнате
   мы можем находится в двух состояниях - зеленом и красном.

Пересекая зеленую линию 1 мы попадаем в комнату А в зеленом состоянии. Будем записывать это так:

```
Вход  --1->  Aзел
```

Т.к. теперь нельзя пересекать зеленые линии мы можем только пересечь
красную линию 2, чтобы попасть в комнату Е в красном состоянии или
красную линию 4, чтобы попасть в комнату Б в красном состоянии.
Запишем это так:

```
Aзел --2-> Eкр
Aзел --4-> Бкр
```

Теперь, используя нотацию выше, методично проходим по всем комнатам и выписывает куда
мы можем пройти в зеленом и красном состоянии. Т.е. в каждой комнате нужно представить,
что мы зашли туда вначале через зеленую линию и выписать возможные переходы через красные
линии, а потом что мы зашли туда же через красную линию и выписать возможные переходы
через зеленые линии. Ничего сложного, требуется только внимание и аккуратность!

Комната А:

```
Aзел --2-> Eкр
Aзел --4-> Бкр
```

```
Акр --3--> Eзел
```

Комната Б:

```
Бзел --4--> Акр
Бзел --5--> Екр
```

```
Бкр --6--> Взел
```

Комната В:

```
Взел --8--> Вкр
```

```
Вкр --6--> Бзел
Вкр --7--> Гзел
Вкр --9--> Гзел
Вкр --13--> Дзел
```

Комната Г:

```
Гзел --10--> Екр
Гзел --11--> Екр
```

```
Гкр --7--> Взел
Гкр --9--> Взел
Гкр --12--> Езел
```

Комната Д:

```
Дзел --14--> Екр
```

```
Дкр --13--> Взел
```

Комната Е:

```
Езел --10--> Гкр
Езел --11--> Гкр
Езел --14--> Дкр
Езел --15--> Выход
```

```
Екр --3--> Азел
Екр --12--> Гзел
```

Теперь рисуем схему переходов. Каждое состояние рисуем как зеленый или красный эллипс,
каждую линию как зеленую или красную стрелку с соответствующим номером линии.

![Maze Graph Image](/assets/images/maze-manual/maze-graph.png)

Обратите внимание:

В зеленые состояния входят только зеленые стрелки и выходят только красные, а
в красные состояния входят только красные стрелки и выходят только зеленые.
Также, если мы следуем по стрелкам, то у нас все время чередуются цвета.
Такая вот графическая отладка получилась.

Самая трудная часть позади. Теперь осталось только найти путь.
Начинаем с выхода и двигаемся ко входу.
Смотрим какие стрелки ведут к выходу. Только одна красная стрелка 15 из Езел.
Записываем 15.
Дальше смотрим какие стрелки ведут в Езел. Только одна зеленая стрелка 3 из Акр.
Записывает 3.
И так далее.
В итоге у нас должна получиться следующая последовательность:

```
15 3 4 6 8 6 4 1
```

Перевернув которую мы получаем искомый путь:

```
1 4 6 8 6 4 3 15
```

![Maze Graph Path Image](/assets/images/maze-manual/maze-graph-path.png)

И вот найденное решение:

![Maze Task Path](/assets/images/maze-manual/maze-path.png)

Теперь можно ответить на вопросы:

1. Есть ли вообще такой путь? Да, есть.
2. Сколько таких путей? Путь единственный.
3. Как научить других решать похожие задачи? Надеюсь у меня получилось и кто-то из вас узнал что-то новое.

В заключении для полноты следует добавить, что мы использовали математическую модель
ориентированного двудольного графа и для решения задачи свели ее к поиску пути
от Входа до Выхода этом графе.