#№ Отчет по лабораторной работе №3
## по курсу "Логическое программирование"

## Решение задач методом поиска в пространстве состояний

### студент: Бронников М.А.

## Результат проверки

| Преподаватель     | Дата         |  Оценка       |
|-------------------|--------------|---------------|
| Сошников Д.В. |              |               |
| Левинская М.А.|              |               |

> *Комментарии проверяющих (обратите внимание, что более подробные комментарии возможны непосредственно в репозитории по тексту программы)*


## Введение

В современном мире все популярнее становится исскуственный интеллект, который в свою очередь постоянно сталкивается с задачей нахождения решения, имея заданное пространство состояний. Именно для таких задач необходимы алгоритмы поиска в пространствах состояний. 
Для реализации таких алгоритмов очень удобен язык Пролог, в силу того, что пространство состояний обычно представляется в виде графа, в котором вершинами являются состояния, а ребрами - переходы между ними, и в тоже время поиск решений предикатов 
в прологе осуществляется поиском по всевозможным вариантам значений аргументов, при котором предикат может быть истинным. Данное сходство между Прологом и алгоритмами поиска помогает легко, в несколько строк реализовать эти алгоритмы в Прологе.

## Задание

2. Три миссионера и три каннибала хотят переправиться с левого берега реки на правый. Как это сделать за минимальное количество шагов, если в их распоряжении 3-ех местная лодка и ни при каких обстоятельствах миссионеры не должны остаться в меньшинстве.

## Принцип решения

Принцип работы заключается в алгоритмах поиска решений в пространстве состояний. Предикат move\3 - предикат перехода из одного состояния в другое. 3-ий аргумент предиката указывает на то, где в данный момент находится лодка(1 - левый берег, 2- правый).

```
checkside(X, Y) :- X >= Y, !.
checkside(X, Y) :- X = 0, !.
checkside(_, Y) :- Y = 0, !.

move(Until, After, 1) :-
  type(Between),
  minuslist(Until, Between, After),
  manyelems(After, "каннибал", N2),
  manyelems(After, "миссионер", M2),
  checkside(M2, N2),
  minuslist(["каннибал", "каннибал", "каннибал", "миссионер", "миссионер", "миссионер"], After, Another),
  manyelems(Another, "каннибал", N3),
  manyelems(Another, "миссионер", M3),
  checkside(M3, N3).
  
  move(Until, After, 2) :-
  type(Between),
  minuslist(After, Between, Until),
  manyelems(After, "каннибал", N2),
  manyelems(After, "миссионер", M2),
  checkside(M2, N2),
  minuslist(["каннибал", "каннибал", "каннибал", "миссионер", "миссионер", "миссионер"], After, Another),
  manyelems(Another, "каннибал", N3),
  manyelems(Another, "миссионер", M3),
  checkside(M3, N3).
  
```
Здесь type\1 содержит всевозможные варианты размещения в лодке, minuslist\3 осуществялет вычитание из одного списка элементов, состовляющих другой список, checkside\2 делает проверку условия безобасти миссионеров в данном состоянии.

Предикат waycreate добавляет в список последовательных состояний, требуемых для достижения цели, очередное состояние, если оно не является членом списка(в целях предотвращения зацикливания).
```
waycreate([st(X, 1)|T], [st(Y, 2), st(X, 1)|T]):-
  move(X, Y, 1),
  not(mymember(st(Y, 2), [st(X, 1)|T])).

waycreate([st(X, 2)|T], [st(Y, 1), st(X, 2)|T]):-
  move(X, Y, 2),
  not(mymember(st(Y, 1), [st(X, 2)|T])).

```
И на основе вышеописанных предикатов реализуются алгоритмы поиска в глубину, в глубину с итерационным погружением и в ширину.
```
% поиск в глубину

dsrch([st(X, 2)|T], X, [st(X, 2)|T]).

dsrch([st(R, N)|P], X, L):-
  waycreate([st(R, N)|P], P1),
  dsrch(P1, X, L).


dpth_search(Until, After):-
 get_time(Time1),
 dsrch([st(Until, 1)], After, L),
 get_time(Time2),
 T is Time2 - Time1,
 write('Time: '),
 writeln(T),
 prnt(L).

% поиск в ширину

bdth([[st(X, 2)|T]|_], X, [st(X, 2)|T]).

bdth([B|P], X, L):-
  findall(W, waycreate(B, W), Q),
  append(P, Q, QP),
  !, bdth(QP, X, L);
  bdth(P, X, L).

 bdth_search(Until, After):-
  get_time(Time1),
  bdth([[st(Until, 1)]], After, L),
  get_time(Time2),
  T is Time2 - Time1,
  write('Time: '),
  writeln(T),
  prnt(L).

% поиск с итерационным углублением

natural(1).
natural(B) :- natural(A), B is A + 1.

itdpth([st(A, 2)|T], A, [st(A, 2)|T], 0).

itdpth(P, A, L, N) :-
  N > 0,
  waycreate(P, Pl),
  Nl is N - 1,
  itdpth(Pl, A, L, Nl).

id_search(Until, After) :-
  get_time(Time1),
  natural(N),
  itdpth([st(Until, 1)], After, L, N),
  get_time(Time2),
  T is Time2 - Time1,
  write('Time: '),
  writeln(T),
  prnt(L), !.
  
```
Здесь предикат print\1 осуществляет более читабильный вывод решения, а get_time замеряет время для последующего измерения времени исполнения программы.

## Результаты

Приведите результаты работы программы: найденные пути, время, затраченное на поиск тем или иным алгоритмом, длину найденного первым пути. Используйте таблицы,
если необходимо.

```
?- id_search(["миссионер", "миссионер", "миссионер", "каннибал", "каннибал", "каннибал"], []).
Time: 0.006711244583129883
Берег 1:[миссионер,миссионер,миссионер,каннибал,каннибал,каннибал]~~Берег 2:[]
Берег 1:[миссионер,миссионер,каннибал,каннибал]~~Берег 2:[каннибал,миссионер]
Берег 1:[миссионер,миссионер,миссионер,каннибал,каннибал]~~Берег 2:[каннибал]
Берег 1:[миссионер,каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер]
Берег 1:[каннибал,миссионер,миссионер,каннибал]~~Берег 2:[каннибал,миссионер]
Берег 1:[каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер,миссионер]
Берег 1:[миссионер,каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер]
Берег 1:[]~~Берег 2:[каннибал,каннибал,каннибал,миссионер,миссионер,миссионер]
true.

?- dpth_search(["миссионер", "миссионер", "миссионер", "каннибал", "каннибал", "каннибал"], []).
Time: 0.0010268688201904297
Берег 1:[миссионер,миссионер,миссионер,каннибал,каннибал,каннибал]~~Берег 2:[]
Берег 1:[миссионер,миссионер,каннибал,каннибал]~~Берег 2:[каннибал,миссионер]
Берег 1:[миссионер,миссионер,миссионер,каннибал,каннибал]~~Берег 2:[каннибал]
Берег 1:[миссионер,каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер]
Берег 1:[каннибал,миссионер,миссионер,каннибал]~~Берег 2:[каннибал,миссионер]
Берег 1:[каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер,миссионер]
Берег 1:[миссионер,каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер]
Берег 1:[]~~Берег 2:[каннибал,каннибал,каннибал,миссионер,миссионер,миссионер]
true ;
Time: 7.977422475814819
Берег 1:[миссионер,миссионер,миссионер,каннибал,каннибал,каннибал]~~Берег 2:[]
Берег 1:[миссионер,миссионер,каннибал,каннибал]~~Берег 2:[каннибал,миссионер]
Берег 1:[миссионер,миссионер,миссионер,каннибал,каннибал]~~Берег 2:[каннибал]
Берег 1:[миссионер,каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер]
Берег 1:[каннибал,миссионер,миссионер,каннибал]~~Берег 2:[каннибал,миссионер]
Берег 1:[каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер,миссионер]
Берег 1:[каннибал,каннибал]~~Берег 2:[каннибал,миссионер,миссионер,миссионер]
Берег 1:[]~~Берег 2:[каннибал,каннибал,каннибал,миссионер,миссионер,миссионер]
true ;
Time: 8.748473405838013
Берег 1:[миссионер,миссионер,миссионер,каннибал,каннибал,каннибал]~~Берег 2:[]
Берег 1:[миссионер,миссионер,миссионер,каннибал]~~Берег 2:[каннибал,каннибал]
Берег 1:[каннибал,миссионер,миссионер,миссионер,каннибал]~~Берег 2:[каннибал]
Берег 1:[миссионер,каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер]
Берег 1:[каннибал,миссионер,миссионер,каннибал]~~Берег 2:[каннибал,миссионер]
Берег 1:[каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер,миссионер]
Берег 1:[миссионер,каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер]
Берег 1:[]~~Берег 2:[каннибал,каннибал,каннибал,миссионер,миссионер,миссионер]
true ;
Time: 9.090486526489258
Берег 1:[миссионер,миссионер,миссионер,каннибал,каннибал,каннибал]~~Берег 2:[]
Берег 1:[миссионер,миссионер,миссионер,каннибал]~~Берег 2:[каннибал,каннибал]
Берег 1:[каннибал,миссионер,миссионер,миссионер,каннибал]~~Берег 2:[каннибал]
Берег 1:[миссионер,каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер]
Берег 1:[каннибал,миссионер,миссионер,каннибал]~~Берег 2:[каннибал,миссионер]
Берег 1:[каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер,миссионер]
Берег 1:[каннибал,каннибал]~~Берег 2:[каннибал,миссионер,миссионер,миссионер]
Берег 1:[]~~Берег 2:[каннибал,каннибал,каннибал,миссионер,миссионер,миссионер]
true ;
false.

?- bdth_search(["миссионер", "миссионер", "миссионер", "каннибал", "каннибал", "каннибал"], []).
Time: 0.0035555362701416016
Берег 1:[миссионер,миссионер,миссионер,каннибал,каннибал,каннибал]~~Берег 2:[]
Берег 1:[миссионер,миссионер,каннибал,каннибал]~~Берег 2:[каннибал,миссионер]
Берег 1:[миссионер,миссионер,миссионер,каннибал,каннибал]~~Берег 2:[каннибал]
Берег 1:[миссионер,каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер]
Берег 1:[каннибал,миссионер,миссионер,каннибал]~~Берег 2:[каннибал,миссионер]
Берег 1:[каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер,миссионер]
Берег 1:[миссионер,каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер]
Берег 1:[]~~Берег 2:[каннибал,каннибал,каннибал,миссионер,миссионер,миссионер]
true ;
Time: 0.8852996826171875
Берег 1:[миссионер,миссионер,миссионер,каннибал,каннибал,каннибал]~~Берег 2:[]
Берег 1:[миссионер,миссионер,каннибал,каннибал]~~Берег 2:[каннибал,миссионер]
Берег 1:[миссионер,миссионер,миссионер,каннибал,каннибал]~~Берег 2:[каннибал]
Берег 1:[миссионер,каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер]
Берег 1:[каннибал,миссионер,миссионер,каннибал]~~Берег 2:[каннибал,миссионер]
Берег 1:[каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер,миссионер]
Берег 1:[каннибал,каннибал]~~Берег 2:[каннибал,миссионер,миссионер,миссионер]
Берег 1:[]~~Берег 2:[каннибал,каннибал,каннибал,миссионер,миссионер,миссионер]
true ;
Time: 1.3531155586242676
Берег 1:[миссионер,миссионер,миссионер,каннибал,каннибал,каннибал]~~Берег 2:[]
Берег 1:[миссионер,миссионер,миссионер,каннибал]~~Берег 2:[каннибал,каннибал]
Берег 1:[каннибал,миссионер,миссионер,миссионер,каннибал]~~Берег 2:[каннибал]
Берег 1:[миссионер,каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер]
Берег 1:[каннибал,миссионер,миссионер,каннибал]~~Берег 2:[каннибал,миссионер]
Берег 1:[каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер,миссионер]
Берег 1:[миссионер,каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер]
Берег 1:[]~~Берег 2:[каннибал,каннибал,каннибал,миссионер,миссионер,миссионер]
true ;
Time: 1.696690559387207
Берег 1:[миссионер,миссионер,миссионер,каннибал,каннибал,каннибал]~~Берег 2:[]
Берег 1:[миссионер,миссионер,миссионер,каннибал]~~Берег 2:[каннибал,каннибал]
Берег 1:[каннибал,миссионер,миссионер,миссионер,каннибал]~~Берег 2:[каннибал]
Берег 1:[миссионер,каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер]
Берег 1:[каннибал,миссионер,миссионер,каннибал]~~Берег 2:[каннибал,миссионер]
Берег 1:[каннибал]~~Берег 2:[каннибал,каннибал,миссионер,миссионер,миссионер]
Берег 1:[каннибал,каннибал]~~Берег 2:[каннибал,миссионер,миссионер,миссионер]
Берег 1:[]~~Берег 2:[каннибал,каннибал,каннибал,миссионер,миссионер,миссионер]
true ;
false.

```

Нас интересует результат первого найденного решения, его мы и запишем.

| Алгоритм поиска |          Время работы          |  Длина найденного пути  |
|-----------------|--------------------------------|-------------------------|
| В глубину       |      0.0010268688201904297     |           7             |
| В ширину        |      0.0035555362701416016     |           7             |
| ID              |      0.006711244583129883      |           7             |
## Выводы

Эта лабораторная работа научила меня решать задачи исскуственного интеллекта с помощью алгоритмов нахождения путей в пространстве состояний на одном из декларативных языков, таком как Пролог.
В процессе решения задачи мне не раз приходилось задуматься о том, насколько труден и объемен получился мой код при использовании декларативных языков. Недаром первые задичи искуственного интеллекта прошлого столетия были решены на Прологе.

Самым интересным по реализации мне показался алгоритм с итерационным погружением, который позволяет искать наикратчайший путь с относительно небольшим расходом памяти. Такой алгоритм следует использовать когда стоит задача нахождения наименьшего пути в условиях ограниченной памяти. 
Вторым алгоритмом, позволяющем искать наикратчайший путь оказался алгоритм поиска в ширину, использование которого позволяет искать нужный путь чуть быстрее, нежели поиск с итерацонным погружением, однако при этом с крупным перерасходом памяти. Этот алгоритм мне понравился менее всего ввиду его прожорливости и, как мне показалось, трудности его реализации на Прологе.
И самым быстрым, простым в реализации и наиболее пригодным к моей сегодняшней задаче(тк все решения имели длину 7) оказался алгоритм поиска в глубину, который наиболее пригоден в условиях, если не требуется искать наикратчайший путь.

Я верю, что полученные мной сегодня знания помогут мне в будущем в освоении программирования искуственного интеллекта.


