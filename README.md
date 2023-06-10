# 1. Понятия анализа и верификации ПО. Классификация методов анализа ПО. Непротиворечивость, полнота и точность анализа. Проблемы анализа ПО: теорема Тьюринга, теорема Райса.

# 2. Основные подходы к статическому анализу ПО. Сигнатурный поиск. Понятие абстрактного синтаксического дерева, другие модели ПО.

# 3. Анализ ПО на основе типов. Связь классических систем типов и анализа ПО. Системы типов как простейший вид анализа ПО. Полнота и точность анализа на основе систем типов. Понятие анализа на основе ограничений, типичная структура такого анализа.

# 4. Способы решения задачи присвоения типов. Задача унификации. Алгоритмы решения задачи унификации. Рекурсивные типы и регулярная унификация.

# 5. Понятие графа потока управления и моделей на его основе. Понятие чувствительности к потоку управления в анализе ПО. Виды потокочувствительных анализов. Абстрактный домен и абстрактное состояние. Потокочувствительный анализ знаков.

# 6. Введение в теорию решёток. Понятие частично упорядоченного множества. Верхние и нижние грани. Примеры решёток, применимых в анализе ПО. Анализ знака на основе теории решёток.

# 7. Теорема о наименьшей неподвижной точке. Понятие монотонного фреймворка. Алгоритмы поиска неподвижной точки. May- и must- анализ.

### Наименьшая неподвижная точка:

Обычно в результате анализа мы получаем набор переменных $x_1, x_2, \dots, x_n$ и ограничений:


$x_1 = f_1(x_1, x_2, \dots, x_n)$ </br>
$x_2 = f_2(x_1, x_2, \dots, x_n)$</br>
$\dots$</br>
$x_n = f_n(x_1, x_2, \dots, x_n)$</br>

Эти ограничения можно собрать в одну функцию, тогда они примут вид:


$X = f(f_1(x_1, x_2, \dots, x_n),$</br>
$\dots, f_n(x_1, x_2, \dots, x_n))$</br>


В итоге наша задача при решении - найти наименьший (наиболее точный) $X$, который всё ещё будет удовлетворять поставленным условиям - это значение и называется наименьшей неподвижной точкой.

Сама теорема о неподвижной точке звучит так: Для любой монотонной функции $f : L → L$, определенной на решетке $L$ конечной длины, существует наименьшая неподвижная точка.

### Монотонный фреймворк

В теореме о неподвижной точке есть понятие "монотонная функция". Функция $f : L → L$ является монотонной на решетке $L$, если
$∀x, y ∈ L : x <= y ⇒ (f(x) <= f(y))$ - если совсем упрощать: "при уточнении входных данных результат не должен становиться менее точным".

Функция нескольких аргументов является монотонной, если она монотонна по каждому аргументу. Проверить функцию 
(в контексте анализа знаков функция - каждый используемый оператор) можно путём перебора всех таблиц:

$∀x, y, x' ∈ L : x <= x' ⇒ (x op y <= x' op y) $</br>
$∀x, y, y' ∈ L : y <= y' ⇒ (x op y <= x op y')$</br>

Иными словами: "при более точных аргументах результат функции не должен быть менее точным".

Сам монотонный фреймворк представляет из себя:
- Набор вершин CFG программы;
- Решётку конечной длины L (представление интересующего нас домена);
- Набор переменных для каждой вершины CFG;
- Набор ограничений для различных видов узлов CFG.

Используя всё вышеперечисленное, можно извлечь ограничения из CFG и решить их, используя алгоритм поиска неподвижной точки.

### Поиск неподвижной точки

##### Простейший алгоритм

Для начального $X = (⊥, \dots, ⊥)$ расчитываем $X = f(X)$, пока $X$ не перестанет изменяться. Алгоритм корректный, 
но долгий - никак не учитывает ни структуру CFG/используемой решётки.

##### Round-robin / хаотический

Для начального $(x_1,\dots, x_n) = (⊥, \dots, ⊥)$ расчитываем $x_i = f_i(x_1,\dots, x_n)$, пока $(x_1,\dots, x_n)$ не перестанет изменяться.
В round-robin $i$ изменяется в пределах $1..n$ (перебираем $x_i$ по очереди), в хаотическом - принимает случайное значение в тех же пределах.

##### Структурный алгоритм

Используем структуру CFG: будем итеративно проходиться по всем вершинам графа, для каждой вершины в списке $WL$:
- Расчитывая $y = f_i(x_1,\dots, x_n)$;
- Если $x_i != y$, то $x_i = y$ и добавляем потомков текущей вершины в $WL$.

Можно и дальше улучшить алгоритм, особым образом приоретизируя вершины или анализируя иные зависимости вершин друг от друга.

На условной решётке ход работы алгоритма выглядит примерно следующим образом:

![fixPointVisual](images/7/fixPointVisual.png)

### May- и Must- анализы

В моментах, когда нам необходим ответ, который точно будет включать в себя истинный, используется May-анализ, он же переаппроксимация:
- Решётка обходится снизу вверх, от частных результатов к общим;
- Например, анализ знаков, поиск живых переменных (live variables), обнаружение зависимостей значения текущей переменной (reaching definitions).

И, напротив, Must-анализ, он же недоаппроксимация, используется в обратных случаях:
- На практике рассматривают решётку не с $<=$, а с $>=$;
- Используется при поиске предрасчитанных выражений (availible expressions) и анализе занятости (very busy expressions).

# 8. Анализ живости. Анализ готовых выражений.

### Анализ живости

Анализ живости (live variables) служит для поиска живых переменных в различных точках программы, а "мёртвые" переменные 
можно просто игнорировать во многих задачах анализа/оптимизации, что уменьшает фактическую размерность задач.

Так как живость переменной в какой-либо момент выполнения программы - нетривиальное свойство, его необходимо аппроксимировать: 
будем искать в первую очередь те переменные, которые ТОЧНО мертвы (May-анализ). 

В качестве решётки будем использовать булеан от всех переменных программы - таким образом, самым безопасным для нас 
ответом будет живость всех переменных в рассматриваемой точке программы.

В качестве точек программы будем рассматривать состояния программы ДО вершин CFG с использованием следующих правил:

$⟦var x;⟧ = JOIN(n) \ {x} $</br>
$⟦x = E;⟧ = JOIN(n) \ {x} ∪ vars(E) $</br>
$⟦n⟧ = JOIN(n) ∪ vars(n)$</br>

Иными словами:
- При инициализации переменной удаляем её из списка живых;
- При назначении переменной какого-либо значения удаляем её из списка живых, затем добавляем в список все использованные в расчёте переменные;
- При любых других выражениях добавляем в список живых все используемые переменные.

$JOIN(n)$ в данном анализе - объединение полученных от всех потомков значений -> сам анализ выполняется в обратном порядке (Backward analysis)

### Анализ готовых выражений

Анализ готовых выражений (Available expressions) служит для поиска рассчитанных ранее выражений (уже рассчитанные выражения можно не пересчитывать повторно).
Так как "готовность" выражения в какой-либо момент выполнения программы - нетривиальное свойство, его необходимо аппроксимировать:
будем искать в первую очередь те переменные, которые уже точно считались (Must-анализ).

В качестве решётки будем использовать перевёртнутый булеан от всех переменных программы - таким образом, самым безопасным для нас
ответом будет отсутствие расчитанных переменных в рассматриваемой точке программы.

В качестве точек программы будем рассматривать состояния программы после вершин CFG с использованием следующих правил:

$⟦x = E;⟧ = removerefs(JOIN(n) ∪ exprs(n), x) $</br>
$⟦n⟧ = JOIN(n) ∪ exprs(n)$</br>

Иными словами:
- При назначении переменной какого-либо значения удаляем все выражения, в которых эта переменная использовалась;
- При наличии любых других нетривиальных выражений добавляем их в список.

$JOIN(n)$ в данном анализе - пересечение полученных от всех предшественников значений -> сам анализ выполняется в прямом порядке (Forward analysis)

# 9. Анализ занятости. Анализ достижимых значений. Распространение констант.

### Анализ занятости

Анализ занятости (Very busy expressions) служит для поиска выражений, которые были гарантированно использованы до интересующей нас точки в программе.
Так как "готовность" выражения в какой-либо момент выполнения программы - нетривиальное свойство, его необходимо аппроксимировать:
будем искать в первую очередь те переменные, которые уже точно использовались (Must-анализ).

В качестве решётки будем использовать перевёртнутый булеан от всех переменных программы - таким образом, самым безопасным для нас
ответом будет отсутствие использованных до рассматриваемой точки программы выражений.

В качестве точек программы будем рассматривать состояния программы ДО вершин CFG с использованием следующих правил:

$⟦x = E;⟧ = removerefs(JOIN(n), x) ∪ exprs(n) $</br>
$⟦n⟧ = JOIN(n) ∪ exprs(n) $</br>

Иными словами:
- При назначении переменной какого-либо значения удаляем все выражения, в которых эта переменная использовалась, затем добавляем все используемые при расчёте присваиваемого значения переменные;
- При наличии любых других нетривиальных выражений добавляем их в список.

$JOIN(n)$ в данном анализе - пересечение полученных от всех потомков значений -> сам анализ выполняется в обратном порядке (Backward analysis)

### Анализ достижимых значений

Анализ достижимых значений (Reaching definitions) служит для поиска тех присваиваний/объявлений, которые отвечают за значения 
переменных в текущей точке программы.
Так как достижимость выражений в какой-либо момент выполнения программы - нетривиальное свойство, его необходимо аппроксимировать:
будем искать наиболее полный набор определений, пусть среди них и будут лишние(Must-анализ).

В качестве решётки будем использовать булеан от всех присваиваний в программе - таким образом, самым безопасным для нас
ответом будут все присваивания.

В качестве точек программы будем рассматривать состояния программы после вершин CFG с использованием следующих правил:

$⟦var x;⟧ = {var x} $</br>
$⟦x = E;⟧ = = removedefs(JOIN(n), x) ∪ {x = E} $</br>
$⟦n⟧ = JOIN(n)$</br>

Иными словами:
- При объявлении переменной включаем её в список;
- При присвоении значения переменной удаляем из списка предыдущее объявление/присвоение ей значения и вносим новое;
- Для всех остальных значений оставляем список без изменений.

$JOIN(n)$ в данном анализе - объединение полученных от всех предшествнников значений -> сам анализ выполняется в прямом порядке (Forward analysis)

### Распространение констант

Распространение констант (Constant propagation) служит для предрасчёта тех значений, которые точно являются константами. 

В простейшем виде используется плоская решётка с перечислением всех возможных констант, однако, такая реализация может быть расширена решёткой для знаков.

При дальнейшем расширении (например, для возможности предрасчёта выражений `x >= 1` (x в домене 1+) ) уже нужна интервальная 
решётка - решётка, ячейки которой являются интервалами значений. При этом данная решётка бесконечна как в высоту, так и в ширину.

# 10. Интервальный анализ. Тривиальная интервальная решётка. Проблемы реализации анализов на основе бесконечных решёток. Widening в применении к решёткам и ограничения, связанные с его применением.

### Интервальный анализ

Интервальный анализ - анализ над интервалами значений. Используется в многих полезных вещах:
- Распространение констант;
- Поиск выходов за границы массива;
- Упрощение выражений;
- т.д.

### Тривиальная интервальная решётка

Простейшая интервальная решётка - булеан от всех возможных интервалов в $\[-\inf, +\inf\]$:

${\[l, u\] |l, u ∈ N ∧ l ⩽ u} ∪ {⊥} $</br>
$\[l, u\] <= \[l', u'\] ⇔ l' ⩽ l ∧ u ⩽ u'$</br>

![intervalLattice](images/10/intervalLattice.png)

### Проблемы реализации анализов на основе бесконечных решёток

Основная проблема тривиальной решётки - то, что она бесконечная. Помимо того, что это несёт за собой проблемы с временем работы, 
все алгоритмы по поиску неподвижной точки не работают (у решётки бесконечная высота). Решение "в лоб", с ограничением размера целых чисел, 
на практике работает плохо, поэтому необходимо иным образом формировать решётку, "загрубляя" её.

### Widening

Widening - способ представления бесконечной интервальной решётки в конечном виде путём загрубления:

$ω(L) = {y ∈ L|∃x ∈ L : y = ω(x)} $</br>
$fix(ω ◦ f) -> fix(f)$</br>

Иными словами:
- $ω$ - некая "загрубляющая" функция, возвращающая конечную решётку, такую, что её стационарная точка - переаппроксимация стационарной точки исходной решётки;
- $ω(L)$ - конечная, "загрублённая" решётка от бесконечной решётки $L$.

Один из примеров реализации - загрубление интервалов до ближайших интервалов над $B$ - множеством констант в программе.
Также $B$ дополняется $-\inf, +\inf$.

![wideningVisual](images/10/wideningVisual.png)

Однако, результаты после использования widening могут получиться слишком неточными из-за переаппроксимации - их можно и нужно уточнить, и для этого используется narrowing.

# 11. Narrowing в применении к решёткам и ограничения, связанные с его применением. Методы объединения widening и narrowing. Модификации алгоритма решения задачи поиска неподвижной точки с применением widening и narrowing.

### Narrowing

Narrowing - Способ уточнения результатов анализа путём применения функции $f$:

$fix <= fixω $</br>
$fix <= f(fixω) <= fixω$</br>

Иными словами:
- "Спускаемся" по решётке вниз, не выходя за истинную стационарную точку;
- Можем всё же попасть прямо в стационарную точку, можем остаться там же, можнм очутиться где-то посередине.

![narrowingVisual](images/11/narrowingVisual.png)

Однако, у narrowing тоже имеется ряд проблем:
- Результат, скорее всего, всё ещё будет переаппроксимацией, хоть и более точной, чем сразу после widening;
- Если исходное B подобрано плохо, то narrowing будет работать очень долго.

### Методы объединения widening и narrowing

Классический вариант - сначала использовать только widening, затем - только narrowing.

Однако, можно пробовать применять информацию о структуре программы, и, в соответствии с ней, чередовать widening и 
narrowing - результат, скорее всего, будет точнее и быстрее.

### Модификации алгоритма решения задачи поиска неподвижной точки с применением widening и narrowing

##### Общий widening

Можно ввести общий widening-оператор:

$∇: LxL → L $</br>
$l_1 <= (l_1 ∇ l_2) >= l_2 ∀l_1, l_2 ∈ L$</br>

Используя тот факт, что для любой возрастающей последовательности $y_(i+1) = y_i ∇ x_(x+1)$ сходится, можем искать стационарную точку 
через $y_(i+1) = y_i ∇ f(y_i)$

##### Общий narrowing

Можно ввести общий narrowing-оператор:

$Δ: LxL → L $</br>
$l_1 <= (l_1 Δ l_2) <= l_2 ∀l_1, l_2 ∈ L$</br>

Используя тот факт, что для любой нисходящей последовательности $y_(i+1) = y_i Δ x_(x+1)$ сходится, можем искать стационарную точку
через $y_(i+1) = y_i Δ f(y_i)$

##### Warrowing

Объединим общий widening и общий narrowing - для $l_1 ♢ l_2$:
- Если $l_2 <= l1$, то используем narrowing ($l_1 Δ l_2$);
- Иначе используем widening ($l_1 ∇ l_2$).

Подобное совместное использование ускоряет сходимость, увеличивает точность решения, но не работает с стандартными решателями монотонных фреймворков.

# 12. Понятие чувствительности к пути исполнения. Зависимости по данным и по управлению. Предикаты путей. Интервальный анализ, чувствительный к путям исполнения, проблемы, связанные с ним. Выбор множества предикатов путей в общем случае. Метод уточнения абстракции на основе контрпримеров (CEGAR).

# 13. Понятие межпроцедурности в анализе ПО. Пессимистичный межпроцедурный анализ и проблемы, связанные с ним. Полная подстановка тела функции и проблемы, связанные с ней. Понятие полного графа потока управления для программы и анализ на его основе.

# 14. Понятие контекстной чувствительности. Метод ограниченного клонирования процедур, его ограничения и параметризация. Методы улучшения контекстной чувствительности.

# 15. Метод контекстной чувствительности на основе входных данных, его анализ в сравнении с методом ограниченного клонирования процедур. Понятие аппроксимации функции в программе. Проблема внешних функций и варианты её решения.

# 16. Анализ указателей. Анализ псевдонимов. Анализ цели. Анализ формы. Направленный и ненаправленный анализ указателей.

Указателем является ссылка на область памяти (memory cells).

Любой анализ над значениями существенно усложняется с появлением в программе указателей, т.к. любые модификации памяти
не предсказуемы.

Сами указатели являются нежелательными, т.к. подразумевают довольно долгую операцию обращения к памяти. Большинство
компиляторов пытается свести количество указателей к минимуму.

Если в программе присутствуют указатели, используются анализы указателей, которые учитывают набор возможных ячеек
памяти, используемых при выполнении программы.

Как правило, реальные анализы нечувствительны к потоку (задача чувствительности к потоку просто неразрешима).

Анализы можно подразделять на 2 подтипа (было объяснено не очень понятно):

- направленный - подразумевается, что, если указатели A и B указывают на одну ячейку памяти, и указатели A и C указывают
  на одну ячейку памяти, это не значит, что B и C указывают на одну ячейку; короче данный тип анализов использует
  включения множеств;
- ненаправленный - подразумевает, что B и C указывают на одну ячейку; грубо говоря данный тип анализов использует
  сравнение множеств.

Далее приведены возможные формулировки анализа указателей.

## Анализ псевдонимов (Alias analysis)

Анализ псевдонимов выявляет для каждой пары указателей, ссылаются ли они на одну и ту же ячейку памяти.

May-формулировка: выявляет для каждой пары указателей, могут ли они ссылаться на одну и ту же ячейку памяти.

Как альтернатива, есть менее точный анализ множеств псевдонимов (alias set analysis), разбивающий все указатели на
множества, которые могут указывать на одни и те же ячейки.

## Анализ цели

Анализ цели выявляет для каждого указателя, на какую ячейку памяти он ссылается.

May-формулировка: выявляет для каждого указателя, на какие ячейки памяти он может ссылаться.

Данный анализ эквивалентен анализу псевдонимов и может быть построен на его основе (и наоборот, на основе анализа цели
можно построить анализ цели).

## Анализ формы

Анализ формы выявляет структуру области памяти, достижимой по указателям. Данный анализ строит модель в виде графа,
где указывается примерное расположение ячеек памяти.

По сути является аналогом анализа цели с другим описанием, но с более формальным обозначением ячеек.

# 17. Алгоритм Стенсгаарда, его реализация на основе унификации.

Алгоритм реализует нахождения наборов псевдонимов (alias sets) - наборов указателей, ссылающихся на одну и ту же ячейку
памяти. Является ненаправленным анализом (направление присваивания не имеет значения).

Для анализа уравнения строятся следующим образом:

- `X = alloc` - `X` присваивается индексированный указатель на выделенную ячейку памяти; каждая аллокация
  рассматривается как отдельный индекс;
- `X = Y` - присвоение одного указателя другим; `X` и `Y` равны;
- `X = &Y` - `X` является указателем на `Y`;
- `*X = Y` - `X` является указателем на некоторую ячейку памяти, равной `Y`;
- `X = *Y` - `Y` является указателем на некоторую ячейку памяти, равной `X`;

Анализ оперирует именно ячейками памяти, т.к. указатели могут ссылаться друг на друга или самих себя, что не стабильно.

Для решения уравнений используется унифицирующий солвер (как при анализе типов). На выходе получаются равенства,
обозначающие множества псевдономов.

Также, как мы помним по анализу типов, унифицирующий солвер учитывает рекурсию, т.е. может учитывать указатели на самих
себя.

Алгоритм имеет сложность $o(N)$.

Проблемой является то, что алгоритм работает с множествами: он объединяет в одно множество указатели, которые могут
ссылаться на одну и ту же ячейку памяти или указатель; при этом, если указатель из другого множества ссылается на один
из указателей этого множества, то алгоритм посчитает, что он ссылается на каждый указатель этого множества. Это снижает
точность анализа.

# 18. Алгоритм Андерсена, его реализация на основе кубического фреймворка. Контекстно-чувствительный анализ указателей.

## Cubic framework

Cubic framework - солвер для решения уравнений следующего вида:

$$
t_i \in x_j\\
x_m \subseteq x_n\\
t_i \in x_j \rightarrow x_m \subseteq x_n
$$

где $t_1, \dots, t_N$ - набор токенов (элементы или константы), $x_1, \dots, x_K$ - набор переменных, соответствующих
множествам констант.

Задача состоит в поиске множеств, в которых находится определенный элемент.

В нашем случае рассматриваются множества указателей в программе и наборы переменных, для которых определяется, как в
нем расположены элементы.

Солвер использует универсальный алгоритм, имеющий сложность $o(n^3)$, зависящую от количества уравнений (это в худшем
случае, обычно это $~o(n^2))$.

Структура решения представляет собой ориентированный ациклический граф (DAG), в вершинах которого хранится:

- множество токенов;
- отображение из токенов в множества пар переменных.

Каждой переменной $x_i$ соответствует вершина графа.

Каждой дуге соответствует отношение включения (стрелка означает, что одно множество включает в себя другое множество).

Уравнения обрабатываются по одному, структура всегда содержит минимальное решение.

Решение уравнений вида $t_i \in x_j$:

$$
\begin{gather}
node \leftarrow nodes(x_j)\\
tokens(node) \leftarrow tokens(node) \lor {t_i}\\
if\ node(t_i) \neq []\ then\\
\quad for all {x_k,x_n} \in node(t_i) do\\
\quad \quad add\ edge\ from\ nodes(x_k)\ to\ nodes(x_n)\\
\quad end\ for\\
\quad node(t_i) \leftarrow {}\\
end\ if
\end{gather}
$$

Решение уравнений вида $t_i \in x_j \rightarrow x_m \subseteq x_n$:

$$
\begin{gather}
node \leftarrow nodes(x_j)\\
if\ t_i \in tokens(node)\ then\\
\quad add\ edge\ from\ nodes(x_m)\ to\ nodes(x_n)\\
else\\
\quad node(t_i) \leftarrow node(t_i) \lor {{x_m,x_n}}
end\ if
\end{gather}
$$

Если при добавлении дуги в граф получился цикл (что означает равенство множеств):

- Соответствующие вершины сливаются вместе
- Сливаются множества токенов
- Сливаются множества пар для каждого токена

## Алгоритм Андерсена

В отличие от алгоритма Стенсгаарда, алгоритм Андерсена является направленным анализом и не сваливает все в сеты.

Если в алгоритме Стенсгаарде использовалось равенство множеств, то для алгоритма Андерсена используется включение,
что позволяет множествам лишь частично пересекаться.

Для анализа уравнения строятся следующим образом:

- `X = alloc` - индексированный указатель на выделенную ячейку памяти относится к множеству токенов переменной `X`;
  каждая аллокация все еще рассматривается как отдельный индекс;
- `X = Y` - множество токенов `X` включает в себя множество токенов `Y`;
- `X = &Y` - указатель на `Y` относится к множеству токенов `X`;
- `*X = Y` - если указатель на некоторую ячейку памяти, относится к множеству токенов `X`, то к его множеству токенов
  относится множество токенов `Y`;
- `X = *Y` - если указатель на некоторую ячейку памяти, относится к множеству токенов `Y`, то его множество токенов
  относится к множеству токенов `X`;

Алгоритм Андерсена является более точным, но говраздо медленнее алгоритма Стенсгаарда.

# 19. Чувствительный к потоку управления анализ указателей. Анализ указателей на основе типов.

Алгоритмы Стенсгаарда и Андерсена работают на всю кодовую базу целиком, а потому они не могут делать анализ в конкретной
точке программы.

## Монотонный фреймворк

Можно использовать монотонный фреймворк, построив решетку булеан над парами всех ячеек памяти, определяемые наборами
аллокаций (можно сопоставить с дугами в points to задаче). Нужней границей решетки является граф без дуг, верхней -

На практике алгоритм является очень медленным и редко используется.

## Анализ указателей на основе типов (Type-based alias analysis)

Данный анализ основан на том, что в языке с системой типизации 2 указателя на 2 разных типа скорее всего не будут
одинаковыми.

Благодаря этому факту можно анализировать указатели на разные типы отдельно, используя любой из рассмотренных нами ранее
анализов, значительно уменьшая пространство поиска.

В реальных системах анализа строится стек анализов. Если на этапе, соответствующего одному из анализов был получен
точный ответ, что 2 указателя - разные, то анализ останавливается. Если анализ не дал точного ответа, то запускается
следующий по стеку анализ.

# 20. Выпуклые реляционные численные домены. Домен выпуклых многогранников.

Нами на лекциях был рассмотрен интервальный анализ, позволявший для каждой переменной определить набор значений, который 
они могут принимать. Данный анализ не может учитывать отношения между переменными.

Существует реляционные численные домены, которые позволяют анализировать линейные зависимости пар переменных.

## Convex Polyhedra

Convex Polyhedra – домен выпуклых многогранников, где отношения между переменными описываются 2 способами:

- системой линейных равенств или неравенств;
- перечислить все возможные значения переменных:

![convex-polyhedra-generator.png](images/20/convex-polyhedra-generator.png)

На основе ограничений составляется N-мерный многогранник, где N – размер множества анализируемых переменных ${x_N}$:

![convex-polyhedra.png](images/20/convex-polyhedra.png)

Данный домен учитывает варианты, при которых область допустимых значений может быть бесконечной (Infinity Polyhedra):

![infinity-polyhedra.png](images/20/infinity-polyhedra.png)

Представление в виде уравнений также называют матричным.

Представление в виде вершин также называют генераторным.

Генераторное представление для Infinity Polyhedra:

![infinity-polyhedra-generator.png](images/20/infinity-polyhedra-generator.png)

## Polyhedra Lattice

Элементами решетки является пара ${C_P, G_P}$, где $C_P$ – описание многогранника в виде матрицы, $G_P$ – описание 
многогранника в виде набора точек, лучей и прямых.

Отношение включения согласно геометрическому смыслу:

- Пересечение — геометрическое пересечение
- Объединение — выпуклая оболочка геометрического объединения

- Как и с диапазонами, для каждого оператора свой eval и нужно проводить widening.

## Дуализм представления многогранников

В оригинальной статье про Convex Polyhedra используется 40 операторов. На лекциях были рассмотрены 2 из них: объединение
и пересечение.

В случае с генераторным представлением можно без проблем объединить все множества вершин, дуг и прямых из 2 исходных 
доменов, и получить новый домен (возможно избыточный, но корректный).  

Пересечение из генераторного представления сделать почти невозможно (сложная операция), но зато это легко сделать при
помощи матричного представления путем слияния всех уравнений.

Поэтому составляются многогранники двумя способами: одни операции легко сделать с уравнениями, другие с множеством 
значений.

Трансформация из одной формы представления в другую возможно при помощи алгоритма Черниковой, алгоритмическая сложность 
равна $o(nc^{2^{n+1}})$.

Если у нас есть оба представления – получить хотя бы одно всегда просто.

# 21. Выпуклые реляционные численные домены. Разностные матрицы и октагоны.

## Домен разностных матриц (Hexagonal Domain)

Позволяет приводить ограничения в форме $x - y \leq c$.

Между всеми парами и 0 переменных формируется разреженная матрица имеющихся зависимостей.

Также эту матрицу можно рассматривать как матрицу смежности некоторого графа (граф потенциалов). По сути меняется просто 
отображение: переменные не столбцы и строки матрицы, а узлы графа.

Основные операции над матрицами довольно просты (например отношение меньше/больше выражается просто применение этой 
операции на соответствующие элементы матриц), таким образом на основе этого домена можно без проблем строить решетки.

Главной фишкой является то, что операции над матрицами нереально оптимизированы как программно, так и аппаратно, что 
делает данный анализ очень быстрым.

Единственным минусом является ограниченная форма, в которой нужно представить уравнения.

## Восьмигранники (Octagon Domain)

В отличие от предыдущего домена позволяет выражать ограничения в форме $\pm x \pm y \leq c$, что делает его более 
гибким.

Т.к. теперь рассматриваются одновременно порицательные и положительные значения, матрица увеличивается в 4 раза, но в 
целом это приемлемо. В остальном решение ищется также, как и для гексагонального домена.

# 22. Реляционные численные домены. Невыпуклые домены: общее представление.

Все домены названы в соответствии с тем, как они выглядят на графике. 

## Powerset над выпуклым доменом

Над любым выпуклым доменом можно построить булеан, который будет N-отдельных фигур. Соответственно, когда требуется 
провести любую операцию (например пересечение), будет проводиться операция этих фигур друг с другом.

Редко используется из-за высокой алгоритмической сложности. 

## Домен пончиков (Donut domain)

Строится на основе 2 любых доменов ${A, B}$, где $A$ - основной домен, $B$ - дырка в основном домене (а по сути 
перевернутая черной магией решетка).

Говорят в каких-то задачах она эффективна.

![donut.png](images/22/donut.png)

## Домен коробок: Boxes domain

Является булеаном над доменом диапазонов, но вместо хранения интервалов по отдельности, хранится в виде графовой 
структуры - линейных решающих диаграмм (LDD). 

Фишка в том, что существует множество эффективных алгоритмов для LDD.

![boxes.png](images/22/boxes.png)

## Конгруэнция

Элементом домена является конгруэнтное соотношение $a \equiv b[c]$.

В данном случае $a \in {\forall k \in W | b + ck}$

$k$ - набор координат.

**Дальше буквально вынос мозга, Беляев сам плохо понимает**

По сути получается набор разбросанных по пространству точек. Для каких-нибудь `for (i in 0..10 step 2)` подходит хорошо.
