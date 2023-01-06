## Конспект лекций по биоинформатике, МКН

## 1. Репликация генома. Задача о часто встречающихся словах. "Скрытые послания"

### Биологическое введение

**ДНК** - дезоксирибонуклеиновая кислота

Цепочки нуклеатидов (нуклеиновые кислоты), создающих молекулу ДНК (двойную спираль)

(~  комплиментарность)
- A - аденин (~T)
- C - цитозин (~G)
- G - гуанин (~C)
- T - тимин (~A)

Между парами близкорасположенных нуклеатидов возникают водородные связи , которые удерживают вмесете 2 нити молекулы ДНК.

**РНК** - рибонуклеиновая кислота

РНК, в свою очередь, одноцепочечная молекула

**ДКН ---(транскрипция)--> РНК --(трансляция)--> белок**

Белок - цепочки цепочка аминокислотных остатков

**Реплика́ция** — процесс создания двух дочерних молекул ДНК на основе родительской молекулы ДНК.

**origin** - точка начала репликации (ori)

**The copying mechanism**

![img](http://bioinformaticsalgorithms.com/images/Replication/semiconservative_replication.png)

**Three hypotheses for DNA replication**

![img](https://3.bp.blogspot.com/_3XmsmouUXic/TOjIwh2l8BI/AAAAAAAAACU/yzrFp9BpxDI/s1600/Slide5.JPG)

semiconservative - гипотеза Ватсона и Крика.

**Эксперимент разделения гипотез**. Взяли тяжелые изотопы азота и заставили бактерий пройти сначала одну стадию репликации,
потом вторую и поместили в центрифугу получившиеся днк, они разнеслись оба раза - значит верна гипотеза
полуконсервативная, а 2 другие нет.


### Ищем ori

Геном бактерии зациклен, чтобы начался процесс репликации, где-то две нити должны раслеиться, то место, где это происходит - точка начала репликации. В этот самом месте нити расплетаются и каждой из них достраивается комплиментарная. 
Длина генома примерно 3 Mbp (base pairs), хотим найти  ori.

В геноме должно найтись *скрытое послание*, которое указывает на точку начала репликации

Есть некоторый белок, который является медиатором репликации, он ее инициирует. Называется он replication protein A; для
бактерий DnaA.

Вполне логично иметь несколько мест для связывания такого белка, одно повредилось - клетка не обречена.
*"Nothing in biology makes sense except in the light of evolution"*

#### Substring Counting Problem:

- **Input**: A string *pattern*, and a longer string *text*
- **Output**: The number if timers *pattern* occurs in text

(Создаем массив длины len(text)-k+1. Проходимся по исходному массиву и считаем количество  подстрок вида text[i, i+k], для каждого i в text. Так делаем для каждого k-мера, насчиная с 0 символа и далее находим максимум из вхождений таких k-меров, максимум из созданного массива.)

#### Frequent words problem

- **Input**: A string *text* and an integer *k*
- **Output:** All most frequent k-mers in *text*

Создаем массив, который станет выходным массивом. В качестве вспомогательной структуры используем map (словарь). Дальше рассматриваем все возможные  k-меры из нашей строки, последовательным окном длины k. Смотрим на k-мер, если его еще нет в нашей структуре данных, то добавляем новый ключ со значением 1, а если уже есть, то увеличивпем счетчик числа вхождений на 1. После этого находим максимальное значение счетчика и выбрать все эелементы с таким значением. 
    
    BetterFrequentwords():
        freq_patterns = empty array
        freq_map = empty dict
        n = len(text)
        for i in range(n-k):
            pattern = text[i,i+k]
            if freq_map[pattern] doesn't exists:
                freq_map[pattern] = 1
            else:
                freq_map[pattern] +=1 
        max_count = Max(freq_map)
        for all str pattern in freq_map:
            if freq_map[pattern] = max_count:
                freq_patterns.append(pattern)
        return freq_patterns



Имеет смысл запускать frequent words problem, учитывая reverse-complimentary

Теперь, чтобы искать скрытые послания в ori других геномов, мы можем искать часто встречающиеся подстроки в них.

### Как теперь искать окно для ori?

Будем сканировать все окна длиной примерно 500 нуклеатидов, в них искать участки, в которых содержатся частотные слова и выбирать наиболее подходящие.

k-мер формирует **(L, t) - clump** (группа) в геноме, если существует короткий участок длины L в котором этот k-мер встречается >
= t раз

#### Clump finding problem:

- **Input:** A string *genome* and integers *k, L, t*
- **Output:** All *k*-mers forming *(L, t)* - clumps in *genome*

Сканируем окном ширины L, запускаем поиск частотных слов и выдаем нужные k-меры

    FindClumbs():
        patterns = empty array
        n = len(text)
        for i in range(n-L):
            window = text[i,i+L]
            freqMap = FreqencyMap(window,k)
        for all str pattern in freqMap:
            if freq_map[pattern] >= t:
                patterns.append(pattern)
        return patterns

Такой подход обречен на провал (с биологической точки зрения):
частотные слова встречаются не только вокруг ori, но и очень много еще где. В геноме очень много повторов.
*Alu in humans is ~300bp long and occurs 1 million times.*

## 2. Асимметрия репликации. Диаграмма асимметрии (skew diagram)

Давайте окном 100k нуклеотидов посчитаем частоту вхождения нуклеотидов в геноме кишечной палочки. (считаем цитозин)

Мы могли бы ожидать 25% для каждого из нуклеотидов, но на самом деле видим, что до какого-то момента C > G, а после
какого-то момента C < G. Это особенно заметно на графике разности их частот.

Нити ДНК ориентированы (от 5' к 3' (количество углерода в сахарном каркасе ДНК)). Процесс репликации происходит несимметрично. ДНК полимераза, отвечающая за наращивание нити
может перемещаться только в направлении 3->5. Лидирующие полунити (как раз расположенные в нужном направлении (выделенные жирным)) могут
быть реплицированы сразу, их называют Leading half-strand

![img](http://bioinformaticsalgorithms.com/images/Replication/replication_fork.png)

С запаздывающими нитями полимераза работает по-другому, достраивает okazaki-фрагменты по одному выжидая некоторое время, сдвигаясь назад в
нужном направлении. После этого специальная ДНК Легаз сшивает воедино. 

![img](http://bioinformaticsalgorithms.com/images/Replication/okazaki.png)

В итоге получаем из одной молекулы ДНК две.

![img](http://bioinformaticsalgorithms.com/images/Replication/naive_replication_complete.png)

Когда ДНК спираль расплетается, возрастает вероятность мутаций. Скорость мутрирования Цитозина в Тимин возрастает на 2 порядка. Таким образом, если один из нуклеотидов имеет повышенный
mutation rate, мы заметим его недостаток на lagging half-strand, т.к он чаще расплетенный (single-stranded). Cytosine
быстро мутирует в Thymine через *deamination*.

Из-за этого происходит дизбаланс в G-C парах до и после ori. Для тех копий ДНК, которые возникли при достраивании ведущей нити как комплиментарной к запаздывающей, там в среднем Гуанина будет меньше. Потому что в момент когда нить остается одна (запаздывающая) ее Цитозин мутирует в Тимин и комплиментарным к нему достраивается Аденин, а не Гуанин. А для второй синей запаздывающей нити, будет противоположная ситуация, т.к Цитозин мутирует в Тимин их становится меньше, а Гуанин сохраняет количество.

![img](http://bioinformaticsalgorithms.com/images/Replication/increasing_decreasing_skew.png)

**Skew array:** Skew[k] = #G - #C for the first k nucleotides of Genome

Таким образом ori можно искать как минимум этого массива.

## 3. Задача о часто встречающихся словах с опечатками.

Раньше мы решали задачу о часто встречающихся словах без учета возможных мутаций и опечаток.

#### Frequent Words with Mismatches Problem: Find the most frequent k-mers with mismatches in a string.

- **Input:** A string Text as well as integers k and d.
- **Output:** All most frequent k-mers with up to d mismatches in Text.

```
FrequentWordsWithMismatches(Text, k, d)
    Patterns ← an array of strings of length 0
    freqMap ← empty map
    n ← |Text|
    for i ← 0 to n - k
        Pattern ← Text(i, k)
        neighborhood ← Neighbors(Pattern, d)
        for j ← 0 to |neighborhood| - 1
            neighbor ← neighborhood[j]
            if freqMap[neighbor] doesn't exist
                freqMap[neighbor] ← 1
            else
                freqMap[neighbor] ← freqMap[neighbor] + 1
    m ← MaxMap(freqMap)
    for every key Pattern in freqMap
        if freqMap[Pattern] = m
            append Pattern to Patterns
    return Patterns
```

## 4. Регуляторные мотивы. Задача поиска мотивов. Задача о медианной строке

У нас есть несколько строк (например 10), хотим найти паттерны которые часто встречаются в каждой из строк

**(k, d)-motif** - a k-mer that appears in each sequence with at most d mismatches

Почему такая задача нам интересна? Организмы как-то регулируют экспресию генов в зависимости от времени суток.
Регуляторные гены прикрепляются к участкам до нужных генов, какой-то префикс куда могут сесть гены активаторы. Идея в
том, что у разных генов в этом префиксе должна содержаться одна информация (площадка например для циркадных регуляторов)

Простой алгоритм: перебрать все варианты смещений на d для каждой подстроки и проверить является ли она мотивом

```
 MotifEnumeration(Dna, k, d)
    Patterns ← an empty set
    for each k-mer Pattern in the first string in Dna
        for each k-mer Pattern’ differing from Pattern by at most d mismatches
            if Pattern' appears in each string from Dna with at most d mismatches
                add Pattern' to Patterns
    remove duplicates from Patterns
    return Patterns
```

Какой-то из генов может регулироваться другим фактором. То, что мы сейчас формализовали, предполагает, что КАЖДАЯ
последовательность содержит мотив.

Будем думать над более гибкими алгоритмами. Выберем по одному k-меру из каждой строки:

![img](http://bioinformaticsalgorithms.com/images/Motifs/motifs_score_count_profile.png)

### Motif finding problem:

Given a collection of strings, find a set of k-mers, one from each string, that minimizes the score of the resulting
motif.

- **Input:** A collection of strings _Dna_ and an integer _k_
- **Output:** A collection _Motifs_ of  _k_-mers, one from each string in _Dna_, minimizing _Score(Motifs)_ among all
  posible choices of k-mers

Можем смотреть не на столбцы, а на строки

![img](http://bioinformaticsalgorithms.com/images/Motifs/motifs_score_consensus.png)

Определим расстояние между

```
[d(Pattern, Motifs) = Sum( HammingDistance(Pattern, Motifs_i))]
```

Тогда Score(Motifs) = d(Consensus(Motifs), Motifs)

#### Equivalent Motif Finding Problem:

Given a collection of strings, find a pattern and a collection of k-mers (one from each string) that minimizes the distance between
all possible patterns and all possible collections of k-mers.

- **Input:** A collection of strings Dna and an integer k.
- **Output:** A k-mer Pattern and a collection of k-mers, one from each string in Dna, minimizing d(Pattern, Motifs)
  among all possible choices of Pattern and Motifs.

Ищем теперь не только набор motifs, но и некоторый шаблон

Если нам дал шаблон, легко найти нужное множество Motifs:

Расстояние между паттерном и большой строкой 

d(Pattern, Motif_i) := min {d(pattern, motif_i[l:r])} 

минимум из всех расстояний по хеммингу

**Расстояние между k-мером и всем множестом DNA ={DNA_1,...,DNA_i}** это 

d(k-mer, Dna) = SUM(d(k-mer, Dna_i))

A **median string** for set of string Dna: a k-mer minimizing distance d(k-mer, Dna) over all possible k-mers

#### Median String Problem:

Finding a median string.

- **Input:** A set of  =sequences Dna and an integer k.
- **Output:** A k-mer minimizing distance d(k-mer, Dna) among k-mers.
- 

    Mediantring(Dna, k):
        best_k_mer = AAA..AA
        for each k-mer from AAA..AA to TTT..TT:
            if d(k-mer, Dna) < d(best_k_mer, Dna):
                best_k_mer = k-mer
        return best_k_mer


Можно решать в лоб 4^k * n * t * k (алгоритм до этого, который в лоб вычисляет Score всех возможных выборов Motifs
работает за (n)^t * k * t ) 
- t - просматриваем кажду строку
- n - сканируем окном длины k - это (n-k+1)  ассимптотически n 
- k - время вычисления расстояния 

## 5. Профили. Поиск мотивов "жадный подход"

![img](http://bioinformaticsalgorithms.com/images/Motifs/motifs_score_consensus.png)

Имеем набор из 10 k-меров - мотивов

Введем в рассмотрение матрицу профиля (частотная матрица, вероятностная)

![img](http://bioinformaticsalgorithms.com/images/Motifs/probability_random_string.png)

k - число столбцов матрицы ( длина нашего слова). Какова вероятность того, что для какого-то k-мера если мы возьмем возьмем по кубику для каждого из столбцов и подбросим его, то случайным образом будем сгинерирована именно такая строка.
Нам интересна условная вероятность ( по отношению к профилю).

Консенсус - наиболее вероятная строка по отношению к данному профилю.

Введем в рассмотрение, для заданной строки наиболее вероятный k-мер который в нее входит по отношению к данному профилю. 

Pr (k-mer| Profile)

Жадный алгоритм: итеративно ищем оптимальное решение

Последовательно формируем набор мотивов, выбирая по одному мотиву из каждой строки нашего ДНК. Переходя от  i-1 шага к i  мы рассматриваем множество мотивов из i-1 k-мера по одному из каждой i-1 строк, для него мы строим профиль и по отношению к этому профилю выбираем лучший из i строки k-мер.

У него есть недостаток: на первом шаге, когда мы выбираем всего 1 k-мер его матрица будет состоять из 0 и 1, на втором шаге, если в точности такого k-мера не будет, то оттуда с равной вероятностью может быть выбран люой k-мер 
```
GreedyMotifSearch(Dna, k, t)
        BestMotifs ← motif matrix formed by first k-mers in each string from Dna
        for each k-mer Motif in the first string from Dna
            Motif1 ← Motif
            for i = 2 to t
                form Profile from motifs Motif1, …, Motifi - 1
                Motifi ← Profile-most probable k-mer in the i-th string in Dna
            Motifs ← (Motif1, …, Motift)
            if Score(Motifs) < Score(BestMotifs)
                BestMotifs ← Motifs
        return BestMotifs
```

Алгоритм сейчас очень плохой, потому что вероятность почти у всех k-mer ов ноль

## 6. Поиск мотивов: вероятностный подход. Семплирование по Гиббсу.

Если у нас есть множество мотивов, то по нему можно построить profile. Можем сгенирировать используя матрицу профиля новый набор мотивов, выбрав в каждой строке наиболее вероятный с точки зрения этого профиля  k-мер. Так можно итерироваться бесконечно.

```
    RandomizedMotifSearch(Dna, k, t)
        randomly select k-mers Motifs = (Motif1, …, Motift) in each string from Dna
        BestMotifs ← Motifs
        while forever
            Profile ← Profile(Motifs)
            Motifs ← Motifs(Profile, Dna)
            if Score(Motifs) < Score(BestMotifs)
                BestMotifs ← Motifs
            else
                return BestMotifs
```

Алгоритм может работать, потому что мы ищем наши мотивы в тех строках, в которых ожидаем их увидеть. Останавливаемся, когда уже ничего не можем улучшить.

Если хотя бы в какой-то строке генерируя мотивы на первом шаге мы угадаем, он притянет все остальные строки.

##### Entropy of Motifs

Можно посчитать энтропию каждого столбца Profile, тогда энтропия of motifs это сумма всех экнтропий столбцов.

![img](https://github.com/Tanya519/bioinf/blob/master/Screen%20Shot%202023-01-06%20at%2012.45.35.png?raw=true)

Можно посмтроить Motif Logo - каждый столбец имеет высоту равную своей энтропии (высота столбца: 2 - энтропия), внутри каждого столбца нарисованы буквы
соответственно частоте.

##### Семплирование по Гиббсу.

Идея в том, чтобы в вероятностном алгоритме модифицировать не все k-mers за раз, а только один, и эту строку выбираем случайным образом. Это позволит плавнее
двигаться в пространстве мотивов.

В начале выбираем по одному k-меру из каждой строки случайно (нашего множество Мотифс), дальше рандомно выбираем k-мер и удаляем его из рассмотрения. Те k-меры, который остались формируют текущаю матрицу Мотифс, для которой можно построить матрицу профиля. После, возвращаемся к забытой строке, и используя текущую матрицу профиля, мы выбираем из нее самый лучший k-мер.

(Выбираем из строки все возможные k-меры и смотрим их вероятность по матрицы профиля, далее выбираем k-мер случайно)

(случайным образом выбираем строку которую будем обновлять), кроме этого выбираем не наиболее вероятный k-mer имея
profile, а семплируем k-mer имея profile


```
GibbsSampler(Dna, k, t, N)
        randomly select k-mers Motifs = (Motif1, …, Motift) in each string from Dna
        BestMotifs ← Motifs
        for j ← 1 to N
            i ← Random(t)
            Profile ← profile matrix constructed from all strings in Motifs except for Motifi
            Motifi ← Profile-randomly generated k-mer in the i-th sequence
            if Score(Motifs) < Score(BestMotifs)
                BestMotifs ← Motifs
        return BestMotifs
```
Но есть проблема, что для большой строки у многих k-меров вероятность будет нулевая, и тогда мы будем рассматривать только малое число k-меров. То, что в каком-то из столбцов мы не увидели вхождения какой-то буквы не означает, что ее вовсе нет, поэтому давайте в матрице counts прибавим к каждому числу 1,и посчитаем новую матрицу профилей,  после этого нулевых вероятностей не будет, но соотношения между друг другом сохранится. 

## 7. Сборка генома. Граф перекрытий. Граф де Брюйна

Важная нужная задача из кучи отрывков пазла собрать картинку.

Допущения: рассматриваем только одну нить, считаем, что с покрытием все в порядке и для фиксированных k-меров, каждый k-мер, который в принципе есть в нашем геноме, встретился в результать сегменирования в явном виде. 

Давайте по строке, которая нам дана сгенирируем k-мерный состав:
**k-mer Composition** - коллекция k-mer-ов которые могут быть получены по данной строке, может содержать повторения. (окном длины k вырезаем k-меры)

#### String reconstruction problem:

Reconstruct a string from its k-mer composition

- **Input:** k-mers
- **Output:** text, which k-mer composition is equal to given (if such string exists)

Жадным алгоритмом можем начать с любого k-мера и дальше начать наращивать нашу строку влево и вправо выуживая из k-меров нужные буквы (перекрытие по длине k-1) Но не все так просто! Из-за повторения могут быть неоднозначности.

**Граф перекрытий**: каждый read положим в вершину, между вершинами проведем ребро, если read-ы можно поставить друг за
другом (ребра направленные). Будем рассматривать префиксы и суффиксы:

- Prefix: first k-1 letters in k-mer
- Suffix: last k-1 letters in k-mer
  
Построим еще дополнительные ребра к тем, которые уже есть:
    
    Если суффикс первой и префикс второй совпадают, строим ребро 

![img](http://bioinformaticsalgorithms.com/images/Assembly/overlap_graph_easy.png)

Но мы не знаем правильного порядка и если отсортируем по алфавиту, то ничего не поймем :(

Геномная строка в таком графе - Гамильтонов Путь (ровно один раз по вершинам) ](может быть определен неоднозначно, привожу только один вариант)(NP hard)



![img](http://bioinformaticsalgorithms.com/images/Assembly/overlap_graph_alternate.png)

**Графе де Брюйна** - reads пометим на ребрах, в вершинах напишем prefix выходящих и suffix входящих ребер длины k - 1

![img](http://bioinformaticsalgorithms.com/images/Assembly/debruijn_path_graph_labeled.png)

## 8. Решение задачи сборки генома с помощью графа де Брюйна

Задача сборки свелась к поиску эйлерова пути

В любом сбалансированном (входящая степень = выходящей) сильно связном графе есть Ейлеров цикл.

Чтобы решить оригинальную задачу нужно найти эйлеров путь а не цикл, соединим фейковым ребром 2 нечетные вершины.

Затруднения в жизни:

- Reads have imperfect coverage
- Sequencing machines are error-prone
- Exact multiplicities of k-mers are unknown
- DNA is double-stranded

Можно искусственным образом из наших ридов уменьшить k. Тогда если покрытие было хорошее у нас что-то может получиться.
Но тогда граф становится более запутанным.

Эйлеров путь найти не получится. Можно найти контиги (contiguous genome segments). Будем искать non-branching paths,
пути у которых indegree(v) == outdegree(v) == 1 для всех вершин на пути. Разобьем наш граф на максимальные non-branching
пути.

Error-prone reads:
![img](http://bioinformaticsalgorithms.com/images/Assembly/bubble.png)

**Figure:** A correct path CGTA → GTAT → TATG → ATGG → TGGA → GGAC along with an incorrect path CGTA → GTAC → TACG →
ACGG → CGGA → GGAC form a “bubble" in a de Bruijn graph, making it difficult to identify which path is correct.

Чтобы искать такие пузыри можно ориентироваться на частоты, покрытие.

Existing assemblers remove bubbles from de Bruijn graphs. The practical challenge is that, since nearly all reads have
errors, de Bruijn graphs have millions of bubbles (see below).

![img](http://bioinformaticsalgorithms.com/images/Assembly/many_bubbles.png)

Bubble removal occasionally removes the correct path, thus introducing errors rather than fixing them. To make matters
worse, in a genome having inexact repeats, where the repeated regions differ by a single nucleotide or some other small
variation, reads from the two repeat copies will also generate bubbles in the de Bruijn graph because one of the copies
may appear to be an erroneous version of the other. Applying bubble removal to these regions introduces assembly errors
by making repeats appear more similar than they are. Thus, modern genome assemblers attempt to distinguish bubbles
caused by sequencing errors (which should be removed) from bubbles caused by variations (which should be retained).

Из-за того что DNA double-stranded, все равно все плохо, нам нужно рассматривать не только сами reads, но и их
reverse-compliment

## 9. Сборка генома на основе парных сочетаний

Один из способов увеличить длину чтения - читать 2 фрагмента генома, которые находятся друг от друга на известном
расстоянии

**(k, d)-mer** - substring [first k-mer][d nucleotides][second k-mer]

Хотим использовать (k, d) - reads чтобы восстановить геном.

Каждый (k, d)-mer должен быть путем длины k + d + 1 в оригинальном графе de-Bruijn

**Prefix/Suffix of (k,d)-mer** - The (k - 1, d + 1)-mer formed by taking prefix/siffix of each k-mer

Определив это можем построить граф де-брюйна, где на ребра (k, d) -mer

![img](http://bioinformaticsalgorithms.com/images/Assembly/composition_graph_paired_reads.png)

Путей может быть несколько, те из них, которые не склеиваются правильно - неправильные

## 10. Секвенирование антибиотиков. Алгоритмы секвенирования циклических пептидов.

_Would I Be Here Without Antibiotics?_

Антибиотики кодируются аминокислотами (как и протеины). Но не все антибиотики соответствуют догме (ДНК -> РНК ->
протеин)

NRP - non-ribosomal peptide, синтезируются не рибосомой, а синтетазой.

Масс спек - дорогие молекулярные весы. 1 Dalton ~ mass of proton / neutron

Будем работать с целыми массами. Чтобы получить массу циклического пептида, сложим массы аминокислот входящих в его
состав. Перешли от алфавита 20 аминокислот в алфавит 18 целых масс.

После масспека получаем информацию о массе каждого фрагмента молекулы (префикса/суффикса)

По теоретическому масспеку нужно построить пептид

Brute-Force: генерируем все пептиды нужной массы и сравниваем масспек.
(Число пептидов обладающее нужной массой растет очень быстро ~10^n )

Branch & Bound алгоритм:

- Посмотреть какие 1-mer встречаются в массах
- Удлинить каждый из 1-mer в 2-mer и отбросить лишний вариант
- И так далее

## 11. Особенности обработки настоящих масс-спеков. Конволюция спектров.

На экспериментальном спектре могут быть:

- не все измерения (суффиксы префкисы) (плохо разорвали связи)
- лишние измерения (что-то прилетело, частички пластика и так далее)
- шумные измерения
- слипщиеся пики

Давайте заведем Score для сравнения масс спектров (например число общих масс)

Тогда B & B стоит переделать в Beam Search (отсылка к гольфу?)

```
LeaderboardCyclopeptideSequencing(Spectrum, N)
        Leaderboard ← set containing only the empty peptide
        LeaderPeptide ← empty peptide
        while Leaderboard is non-empty
            Leaderboard ← Expand(Leaderboard)
            for each Peptide in Leaderboard
                if Mass(Peptide) = ParentMass(Spectrum)
                    if Score(Peptide, Spectrum) > Score(LeaderPeptide, Spectrum)
                        LeaderPeptide ← Peptide
                else if Mass(Peptide) > ParentMass(Spectrum)
                    remove Peptide from Leaderboard
            Leaderboard ← Trim(Leaderboard, Spectrum, N)
        output LeaderPeptide
```

Это не идеальный алгоритм (ну beam search), например m(AG) = m(Q).

На самом деле аминокислот для NRP может быть намного больше чем 20. (100+)
Будем считать, что любая масса от 57 до 200 возможна.

### Spectral Convolution

![img](http://bioinformaticsalgorithms.com/images/Antibiotics/spectral_convolution_nqel.png)

Создадим алфавит из масс которые либо присутствуют в масс спеке либо получаются попарной разностью. Из него возьмем
только значения которые встречаются чаще других.

![img](https://stepik.org/media/attachments/lessons/104/tyrocidine_convolution_2.png)

На самом деле масс спек это отношение массы к заряду, и вообще выглядит он странно.

![img](http://bioinformaticsalgorithms.com/images/Antibiotics/real_spectrum.png)

Пики состоят не только из точек, но из куполов, которые перекрываются и их как-то нужно разделять. Все очень сложно!

## 12. Выравнивание последовательностей. Задача о наибольшей общей подпоследовательности.

Хотим сравнить два сходных гена, понять величину сходства. Расстояние хэмминга плохо работает когда гены разной длины
или получаются циклическим сдвигом.

#### Longest Common Subsequence Problem:

Find a longest common subsequence of two strings.

- **Input:** Two strings.
- **Output:** A longest common subsequence of these strings.

Хотим пройти по сетке, увидев как можно больше достопримечательностей

![img](http://bioinformaticsalgorithms.com/images/Alignment/tourist_map.png)

Построим граф с весами на ребрах. Нужно найти путь оптимальной стоимости. Для DAG (directed acyclic graph) это простая
задача на динамическое программирование.

Также можно решать и исходную задачу:

![img](http://bioinformaticsalgorithms.com/images/Alignment/alignment_path-1.png)

Нужно не забывать, что нужна для этого топологическая сортировка (dfs)

Можно ввести умный Score: _mismatch_ penalty -mu, _indel_ penalty -sigma

![img](http://bioinformaticsalgorithms.com/images/Alignment/alignment_scoring_matrix.png)

На самом деле для аминокислот использую специальную матрицу для замен: Scoring matrix (PAM)

## 13. Глобальное выравнивание. Локальное выравнивание. Множественное выравнивание

#### Global Alignment Problem:

Find a highest-scoring alignment of two strings as defined by a scoring matrix.

- **Input:** Two strings and a scoring matrix Score.
- **Output:** An alignment of the strings whose alignment score (as defined by Score) is maximized over all alignments
  of the strings.

Такой подход не работает для длинных строк, в которых только некоторые участки могут быть консервативными и должны
выравниваться.

#### Local Alignment Problem:

Find the highest-scoring local alignment between two strings.

- **Input:** Strings v and w as well as a matrix score.
- **Output:** Substrings of v and w whose global alignment score (as defined by score) is maximized among all substrings
  of v and w.

Хотим искать участок параллельный главной диагонали, необязательно целый

![img](http://bioinformaticsalgorithms.com/images/Alignment/global_and_local_alignments.png)

Чтобы решать: Free taxi rides in the alignment graph

![img](http://bioinformaticsalgorithms.com/images/Alignment/free_taxi_rides.png)

Connecting the source (0, 0) to every other node by adding a zero-weight edge and connecting every node to the sink (n,
m) by a zero-weight edge will result in a DAG perfectly suited for solving the Local Alignment Problem, shown below.

#### Affine gap penalties

In our previously defined linear scoring model, if σ is the penalty for the insertion or deletion of a single symbol,
then σ · k is the penalty for the insertion or deletion of an interval of k symbols. This cost model unfortunately
results in inadequate scoring for biological sequences. Mutations are often caused by errors in DNA replication that
insert or delete an entire interval of k nucleotides as a single event instead of as k independent insertions or
deletions. Thus, penalizing such an indel by σ · k represents an excessive penalty.

#### Alignment with Affine Gap Penalties Problem:

Construct a highest-scoring global alignment between two strings (with affine gap penalties).

- **Input:** Two strings, a matrix score, and numbers σ and ε.
- **Output:** A highest scoring global alignment between these strings, as defined by the scoring matrix score and by
  the gap opening and extension penalties σ and ε.

Добавляем длинные ребра

![img](http://bioinformaticsalgorithms.com/images/Alignment/long_indel_edges-consolidated.png)

Но это слишком много ребер! Генерируем новый граф, из трех слоев.

Слой вставок, слой удалений, слой match-ей

![img](http://bioinformaticsalgorithms.com/images/Alignment/three_level_graph_path.png)

Чтобы решать исходную задачу за линейную память нужен meet in the middle, считаем матрицу значений в центре с двух
сторон и понимаем где разделить решетку на части (сложность именно в восстановлении пути)

![img](http://bioinformaticsalgorithms.com/images/Alignment/middle_node_3-4.png)

#### Multiple Alignment Problem:

Find the highest-scoring alignment between multiple strings under a given scoring matrix.

- **Input:** A collection of t strings and a t-dimensional matrix Score.
- **Output:** A multiple alignment of these strings whose score (as defined by the matrix Score) is maximized among all
  possible alignments of these strings.

Решается просто жирным многомерным графом.

## 14. Геномные перестройки. Сортировки обращениями. Точки разрыва.

What evolutionary forces have transformed the genome of the human-mouse ancestor into the present-day human and mouse
genomes?

To simplify genome comparison, we will first focus on the X chromosome, which is one of the two sex-determining
chromosomes in mammals and has retained nearly all its genes throughout mammalian evolution. We can therefore view the X
chromosome as a “mini-genome” when comparing mice to humans, since this chromosome’s genes have not jumped around onto
different chromosomes (and vice-versa)

It turns out not only that most human genes have mouse counterparts, but also that hundreds of similar genes often line
up one after another in the same order in the two species genomes. Each of the eleven colored segments in the figure
below represents such a procession of similar genes and is called a synteny block. Later, we will explain how to
construct synteny blocks and what the left and right directions of the blocks signify.

![img](http://bioinformaticsalgorithms.com/images/Rearrangements/mouse_and_human_synteny_blocks.png)

The figure below shows a series of seven reversals transforming the mouse X chromosome into the human X chromosome.

![img](http://bioinformaticsalgorithms.com/images/Rearrangements/transforming_mouse_into_human_7_reversals.png)

You can visualize a reversal as breaking the genome on both sides of a chromosomal interval, flipping the interval, and
then gluing the resulting segments in a new order. Keeping in mind that earthquakes occur more frequently along fault
lines, we wonder if a similar principle holds for reversals — are they occurring over and over again in the same genomic
regions? A fundamental question in chromosome evolution studies is whether the **breakage points** of reversals (i.e.,
the the ends of the inverted intervals) occur along “fault lines” called **rearrangement hotspots**.

Сначала люди думали, что точек разрыва не бывает и разрыввы везде равновероятны, но потом посмотрели на распределение
длин блоков синтении. Получилось, что распределение размеров блоков должно быть экспоненциальным. На деле оно таким и
оказалось.

Давайте научимся за минимальное число шагов превращать один геном в другой разворотами. Genome rearrangement studies
typically ignore the lengths of synteny blocks and represent chromosomes by signed permutations.

- Mouse: (+1 −7 +6 −10 +9 −8 +2 −11 −3 +5 +4)
- Human: (+1 +2 +3 +4 +5 +6 +7 +8 +9 +10 +11)

#### Sorting by Reversals Problem:

Compute the reversal distance between a permutation and the identity permutation.

- **Input:** A permutation P.
- **Output:** The reversal distance d_{rev}(P) (число перестановок которые надо совершить чтобы из перестановки получить
  тождественную).

Жадный алгоритм: давайте поставим на место сначала 1, потом 2, потом 3... Для этого если элемент k не на месте, находим
где он, флипаем подстроку от [k, position(k)] и получаем либо +k либо -k на k-ой позиции, возможно придется сделать еще
один flip и продолжить.

Это конечно не оптимальный алгоритм. We therefore say that consecutive elements (pi, pi+1) in permutation P = (p1 ...
pn) form an adjacency if p[i+1] − p[i]
is equal to 1. By definition, for any positive element k < n, both (k k + 1) and (−(k + 1) −k) are **adjacencies**. If
pi+1 − pi is not equal to 1, then we say that (pi pi+1) is a **breakpoint**.

Добавим к перестановке мысленно 0 в начало и n + 1 в конце, чтобы (-1, -2, -3) имело больше 0 breakpoints.

Because any pair of consecutive elements of a permutation form either a breakpoint or adjacency, we have the following
identity for any permutation P of length n:

- Adjacencies(P) + Breakpoints(P) = n + 1

Процесс сортировки на самом деле это процесс устранения breakpoints.

![img](http://bioinformaticsalgorithms.com/images/Rearrangements/sorting_by_reversals_example.png)

A reversal can eliminate at most two breakpoints, so two reversals can eliminate at most four breakpoints, three
reversals can eliminate at most six breakpoints, and so on. This reasoning establishes the following theorem.

**Breakpoint Theorem:** The reversal distance drev(P) is at least equal to Breakpoints(P)/2.

Чтобы пойти дальше давайте сначала обобщим задачу на несколько хромосом, так будет проще.

## 15. 2-разрывы и расстояние на их основе. Граф точек разрыва.

Рассматриваем теперь 2 хромосомы. Кроме reversals могут быть: fusion (склеить 2 хромосомы в 1), fission (разбить
хромосому на две), translocation (разбить 2 хромосомы на кусочки и переклеить)

Считаем что все хромосомы цикличные, это неправда, но так проще. 

We now have a multichromosomal genomic model, along with four types of rearrangements (reversals, translocations, fusions, and fissions) that can transform one genome into another. To model genomes with circular chromosomes, we will use a **genome graph**.
First represent each synteny block by a directed black edge indicating its direction, and then link black edges corresponding to adjacent synteny blocks with a colored undirected edge.

![img](http://bioinformaticsalgorithms.com/images/Rearrangements/genome_graph.png)

#### 2-breaks


We now focus on one of the chromosomes in a multi-chromosomal genome and consider a reversal transforming the circular chromosome P = (+a −b −c +d) into Q = (+a −b −d +c). 
As illustrated in the figure below, keeping the black edges fixed allows us to visualize the effect of the reversal. As you can see, the reversal deletes (“breaks”) two red edges in P (connecting b to c and d to a) and replaces them with two new red edges (connecting b to d and c to a).

![img](http://bioinformaticsalgorithms.com/images/Rearrangements/genome_reversal.png)

The figure below illustrates a fission of genome P = (+a −b −c +d) into Q = (+a −b)(−c +d); reversing this operation corresponds to a fusion of the two chromosomes of Q to yield P.

![img](http://bioinformaticsalgorithms.com/images/Rearrangements/fission_and_fusion.png)

A translocation involving two linear chromosomes can also be mimicked by circularizing these chromosomes and then replacing two red edges with two different red edges, as shown in the figure below.

![img](http://bioinformaticsalgorithms.com/images/Rearrangements/translocation.png)

We would like to find a shortest sequence of 2-breaks transforming genome P into genome Q, and we refer to the number of operations in a shortest sequence of 2-breaks transforming P into Q as the 2-break distance between P and Q, denoted d(P, Q).

#### 2-Break Distance Problem:
Find the 2-break distance between two genomes.

- **Input:** Two genomes with circular chromosomes on the same set of synteny blocks.
- **Output:** The 2-break distance between these genomes.

#### Breakpoint Graph

![img](https://i.ibb.co/68q4SHL/Screenshot-2022-01-11-at-12-36-56.png)

We denote the number of red-blue alternating cycles in BreakpointGraph(P, Q) as Cycles(P, Q).

By extension, we can view a series of 2-breaks transforming P into Q as a series of 2-breaks transforming BreakpointGraph(P, Q) into BreakpointGraph(Q, Q), the trivial breakpoint graph

![img](http://bioinformaticsalgorithms.com/images/Rearrangements/2-break_series.png)

The Breakpoint Theorem stated that a reversal applied to a chromosome P can reduce Breakpoints(P) by at most 2. We now prove that a 2-break applied to a multichromosomal genome P can increase Cycles(P, Q) by at most 1, i.e., for any 2-break transforming P into P′, and for any genome Q, Cycles(P′, Q) cannot exceed Cycles(P, Q) + 1.

#### Cycle Theorem:

Given genomes P and Q, any 2-break applied to P can increase Cycles(P, Q) by at most 1.

_Proof_: A 2-break adds two new red edges and thus forms at most 2 new cycles (containing two new red edges) in BreakpointGraph(P, Q). At the same time, it deletes two red edges and thus deletes at least 1 old cycle (containing two old edges) from BreakpointGraph(P, Q). Thus, the number of red-blue cycles in BreakpointGraph(P, Q) increases by at most 2 − 1 = 1, implying that Cycles(P, Q) increases by at most 1. ▢

![img](http://bioinformaticsalgorithms.com/images/Rearrangements/cycle_theorem_proof.png)

#### 2-Break Distance Theorem: 

The 2-break distance between genomes P and Q is equal to Blocks(P, Q)− Cycles(P, Q).

_Proof_: We call a transformation of P into Q by 2-breaks a sorting by 2-breaks﻿.  From before, we know that any sorting by 2-breaks must increase the number of alternating cycles by Cycles(Q, Q)− Cycles(P, Q), which is equal to Blocks(P, Q) − Cycles(P, Q) because Blocks(P, Q) = Cycles(Q, Q). The Cycle Theorem implies that each 2-break increases the number of cycles in the breakpoint graph by at most 1. This immediately implies in turn that d(P, Q) is at least Blocks(P, Q) − Cycles(P, Q). If P is not equal to Q, there must be a non-trivial cycle in BreakPointGraph(P, Q), i.e., a cycle with more than two edges. As shown in the figure from the cycle theorem (see middle figure below), any non-trivial cycle in the breakpoint graph can be split into two cycles by a 2-break, implying that we can always find a 2-break increasing the number of red-blue cycles by 1. Therefore, d(P, Q) is equal to Blocks(P, Q) − Cycles(P, Q). ▢

![img](http://bioinformaticsalgorithms.com/images/Rearrangements/cycle_theorem_proof.png)

Чтобы понять какие именно нужно делать 2-breaks. 

We are now ready to find a series of intermediate genomes in a shortest transformation of P into Q by 2-breaks. The idea of our algorithm, called ShortestRearrangementScenario, is to find a 2-break that will increase the number of red-blue cycles in the breakpoint graph by 1. To do so, as illustrated in the figure below, we select an arbitrary blue edge in a non-trivial alternating red-blue cycle and perform the 2-break on the two red edges flanking this blue edge in order to split the red-blue cycle into two cycles (at least one of which is trivial).

![img](http://bioinformaticsalgorithms.com/images/Rearrangements/shortest_rearrangement_scenario.png)

## 16. Выделение блоков синтении 
**Genomic dot-plots and shared k-mers**

Biologists sometimes visualize repeated k-mers within a string as a collection of points in the plane; a point with coordinates (x, y) represents identical k-mers occurring at positions x and y in the string. The two panels in the figure below present two of these genomic dot plots. (k=3 on the left and 2 on the right)

![img](http://bioinformaticsalgorithms.com/images/Rearrangements/genomic_dot_plots-1-2.png)

**Finding shared k-mers**

Recall that a synteny block is defined by many similar genes occurring in the same order in two genomes. Since similar genes often share the same k-mers (for an appropriately chosen value of k), let’s first find the positions of all k -mers that are shared by the human and mouse X chromosomes. If we choose k to be sufficiently large (e.g., k = 30), then it is rather unlikely that shared k-mers represent spurious similarities. A more likely explanation is that they come from related genes (or shared repeats) in the human and mouse genomes.
Formally, we say that a k-mer is shared by two genomes if either the k-mer or its reverse complement appears in each genome.

We can further generalize the genomic dot plot to analyze the shared k-mer content of two genomes. We color the point (x, y) red if the two genomes share a k-mer starting at respective positions x and y. We color (x, y) blue if the two genomes have reverse complementary k-mers at these starting positions. See the figure below.

![img](http://bioinformaticsalgorithms.com/images/Rearrangements/genomic_dot_plots-4.png)

**Constructing synteny blocks from shared k-mers**

![img](http://bioinformaticsalgorithms.com/images/Rearrangements/e-coli_dot-plot.png)

Один из вариантов найти нужные диагонали - объединять компоненты графов, проводя ребра если точки находятся ниже порога дистанции

 ![img](http://bioinformaticsalgorithms.com/images/Rearrangements/synteny_graphs.png)


## 17. Матрицы расстояний и эволюционные деревья

Вопрос: кто передал нам SARS? Чтобы ответить на это нам нужны эволюционные деревья.

Эволюционные деревья как графы:
![img](http://bioinformaticsalgorithms.com/images/Evolution/tree_of_life.png)

Матрица расстояний - матрица содержащая информацию о расстоянии между парами из n организмов:

- симметричная
- Неотрицательная d[i, j] >= 0
- Неравенство треугольника: d[i, j] + d[j, k] >= d[i, k]

![img](http://bioinformaticsalgorithms.com/images/Evolution/mammal_alignment_distance_matrix.png)

Дерево с n листьями реализует расстояние, если расстояние (путями) между листьями соответствует расстоянию указанному в
матрице.

Not every distance matrix has a tree fitting it. We therefore call a distance matrix additive if there exists a tree
that fits this matrix and non-additive otherwise (an example 4 x 4 non-additive matrix is shown below).

![img](http://bioinformaticsalgorithms.com/images/Evolution/nonadditive_distance_matrix.png)

#### Distance-based Phylogeny problem:

Construct an evolutionary tree from a distance matrix

- **Input:** A distance matrix
- **Output:** The simple tree fitting this distance matrix (if this matrix is additive)

(tree is simple, if there is no vertices with degree 2)

**Theorem:** Every simple tree with at least four nodes has a pair of neighboring leaves.
(идея доказательства - рассмотреть самый длинный путь в дереве)

Отсюда можно выразить d[k, m]:
![img](http://bioinformaticsalgorithms.com/images/Evolution/neighboring_leaves_equality.png)

This discussion implies a recursive algorithm for the Distance-Based Phylogeny Problem:

- find a pair of neighboring leaves i and j by selecting the minimum element Di,j in the distance matrix;
- replace i and j with their parent, and recompute the distances from this parent to all other leaves as described
  above;
- solve the Distance-Based Phylogeny problem for the smaller tree;
- add the previously removed leaves i and j back to the tree.

**Но этот подход не всегда работает.** Не верно, что ближайшие по расстоянию элементы являются соседями.

Новый подход.
**limb** - ребро к листу дерева (не внутреннее)

#### Limb Length Problem:

Compute the length of a limb in a tree defined by an additive distance matrix.

- **Input:** An additive distance matrix D and an integer j.
- **Output:** LimbLength(j), the length of the limb connecting leaf j to its parent in Tree(D).

![img](https://i.ibb.co/FxZzjxQ/Screenshot-2022-01-09-at-13-54-42.png)

![img](https://i.ibb.co/2Y2jLdP/Screenshot-2022-01-09-at-13-55-57.png)

![img](https://i.ibb.co/5FhTPZt/Screenshot-2022-01-09-at-13-58-16.png)

Since we now know how to find the length of any limb in Tree(D), we can construct Tree(D) recursively using the
algorithm illustrated in the figure below and described on the next few steps.

We can now recursively find Tree(D) in four steps:

- pick an arbitrary leaf j, compute LimbLength(j), and construct the distance matrix Dtrimmed;
- solve the Distance-Based Phylogeny Problem for Dtrimmed;
- identify the point in Tree(Dtrimmed) where leaf j should be attached in Tree(D);
- add a limb of length LimbLength(j) growing from this attachment point in Tree(Dtrimmed) to form Tree(D).

![img](http://bioinformaticsalgorithms.com/images/Evolution/additive_phylogeny.png)

Там на самом деле чуть сложнее из-за того что непонятно куда возвращать удаленную вершину. Нужно найти после исправления матрицы
две вершины путь между которыми проходит через родителя нашей удаленной вершины и посчитать, сколько расстояние. Расстояние может сказат
оставить родителя в уже свуществующей вршине или разбить ребро в каком-то месте. [Вот тут](https://github.com/pavlov200912/spbu-bioinf/blob/master/homework_2021.04.17/7C.py) у тебя хорошо написано на питоне



## 18. Ультраметрические эволюционные деревья. Метод объединения соседей.

Перейдем к корневым деревьям. Рассмотрим молекулярные часы. Присвоим значение возраста каждой вершине в дереве (у
листьев 0 лет).

**Ultrametric tree:** distance from root to any leaf is the same (age of root)

Такое дерево можно строить используя кластеризацию.

- Сначала каждый лист отдельный кластер.
- Объединим два кластера с наименьшим расстоянием
- Пересчитаем расстояние: D[c1, c2] = (sum_{i \in c1} sum{j \in c2} d[i, j]) / |c1| |c2|
- Чтобы вычислить возраст узла нужно вычислить расстояние между кластерами и поделить его на два
- Выкидываем ненужные строки и матрицы расстояний и добавляем новый столбец и строку для объединенных кластеров

![img](http://bioinformaticsalgorithms.com/images/Evolution/UPGMA.png)

! Алгоритм не реализовывает матрицу расстояний в дереве.

На самом деле можно использовать формулу пересчета для новых кластеров Ж

```
d[cnew, cm] = (d[ci, cm] * |ci| + d[cj, cm] * |cj|) / (|ci| + |cj|)
```

#### Проверка аддитивности матрицы (Four point condition)

![img](https://i.ibb.co/GWDV85F/Screenshot-2022-01-09-at-14-36-10.png)

![img](https://i.ibb.co/yWmScdV/Screenshot-2022-01-09-at-14-37-01.png)

#### Метод объединения соседей.

UPGMA (алгоритм выше) может не выдавать результат удовлетворяющий матрице расстояний даже если матрица аддитивная.
Алгоритм первый может работать неправильно на аддитивных матрицах, алгоритм с поиском limbs мы не рассматривали, но он
не работает для неаддитивных матриц. А есть алгоритм который работает идеально на аддитивных матрицах, но и на
неаддитивных работает?

The central idea of NeighborJoining is that although finding a minimum element in a distance matrix D is not guaranteed
to yield a pair of neighbors in Tree(D), we can convert D into a different matrix whose minimum element does yield a
pair of neighbors.

First, given an n × n distance matrix D, we define TotalDistanceD(i) as the sum ∑1≤k≤n Di,k of distances from leaf i to
all other leaves. The neighbor-joining matrix D* (see below) is defined such that for any i and j, D\*i,i = 0 and D\*i,j
= (n - 2) · Di,j - TotalDistanceD(i) - TotalDistanceD(j).

Такая матрица выбрасывает outliers, для которых расстояние до других листов существенно превышает среднее

![img](http://bioinformaticsalgorithms.com/images/Evolution/neighbor-joining_conversion.png)

**Neighbor-Joining Theorem**: Given an additive matrix D, the smallest element D\*i,j of its neighbor-joining matrix D*
reveals a pair of neighboring leaves i and j in Tree(D).

![img](https://i.ibb.co/JspktPg/Screenshot-2022-01-09-at-14-45-28.png)

```
NeighborJoining(D)
    n ← number of rows in D
    if n = 2
        T ← tree consisting of a single edge of length D1,2
        return T
    D* ← neighbor-joining matrix constructed from the distance matrix D
    find elements i and j such that D*i,j is a minimum non-diagonal element of D*
    Δ ← (TotalDistanceD(i) - TotalDistanceD(j)) /(n - 2)
    limbLengthi ← (1/2)(Di,j + Δ)
    limbLengthj ← (1/2)(Di,j - Δ)
    add a new row/column m to D so that Dk,m = Dm,k = (1/2)(Dk,i + Dk,j - Di,j) for any k
    D ← D with rows i and j removed
    D ← D with columns i and j removed
    T ← NeighborJoining(D)
    add two new limbs (connecting node m with leaves i and j) to the tree T
    assign length limbLengthi to Limb(i)
    assign length limbLengthj to Limb(j)
    return T
```

Если бы матрица была аддитивна мы бы могли использовать выражение как раньше чтобы вычислить LimbLength(i) = (D[i, k] +
D[i,j] - D[j, k]) / 2. Если матрица неаддитивна, это значение может зависеть от выбора k. Нужно просто усреднить эти
расстояния.

![img](https://i.ibb.co/J50d8bM/Screenshot-2022-01-09-at-14-51-02.png)

![img](https://i.ibb.co/gSpfqVk/Screenshot-2022-01-09-at-14-51-46.png)

**Доказательство теоремы Neighbour-Joining**

Пусть Mult_{i, j}[e] - сколько раз вес ребра e входит в значение D\*[i, j]

**Lemma** Для limbs Mult_{i, j}[e] = -2

Если ребро инцидентное i: оно встречается (n-2) раза в слагаемом (n-2)*D[i,j],
(n-1) раз в слагаемом TotalDistance(i) и 1 раз в TotalDistance(j), итого
(n - 2) - (n - 1) - 1 = -2

Если ребро не инцидентное i или j, оно войдет 1 раз в TotalDistance(i) и 1 раз в TotalDistance(j)
итого -2 **.**

**Lemma** Для внутренних ребер Mult_{i, j}[e] = -2 если путь из i в j проходит через e, -2 * Leaves(i, j) иначе, где
Leaves(i, j)[e] = {если e удалим из дерева получим поддерево которое одно содержит i, j друго не содержит ни одной из
этих вершин, вот Leaves(i, j) это размер поддерева не содержащего i, j}

Для первого случая: вклад (n - 2) для первого слагаемого. Удалим ребро e в поддереве где осталось i будет k вершнин, где
осталось j будет (n - k) вершин, тогда вклад от TotalDistance(i) будет (n - k), а вклад от TotalDistance(j) будет k,
итого (n - 2) - k - (n - k) = -2

Для второго случая: В d[i, j] вклада никакого, в TotalDistance(i), TotalDistance(j) вклад по Leaves(i, j), итого -2 *
Leaves(i, j)**.**

Док-во основного утверждения:

(д-ть:) Пусть D\* [i, j] наим элемент матрицы D\*, тогда i, j пара соседей в T(D):

Пусть i, j не являются парой соседей, тогда найдется пара соседей k, l такая что D\*[k, l] < D\*[i, j] (сейчас докажем
это)

Заметим, что если a, b являются соседями то для любой вершины c, которая не является соседом a, D\*[a, b] < D\*[a, c] (
потому что для соседей все ребра кроме инцидентных домножаются на Leaves(i, j), причем на максимально возможный
вариант (по очереди откусываем каждое ребро, потому что все ребра не лежат на пути))

Это значит, что i, j не могут иметь соседей (иначе было бы значение в D\* поменьше)

Тогда у Parent(j) есть поддерево T2, у Parent(i) поддерево T1, т.к дерево простое и степень либо 1 либо 3

![img](http://bioinformaticsalgorithms.com/images/Evolution/neighbor-joining_proof.png)

Допустим размер T1 <= размера T2, т.к i и j не в T1 или T2, размер T1 < n / 2. Так как у i нет соседей, T1 состоит хотя
бы и 2 вершин, тогда T1 содержит пару соседей - назовем из k, l

Рассмотри внутреннее ребро e, докажем что Multi{k, l}[e] <= Mult_{i, j}[e] (тем самым докажем D\*[k, l] <= D\*[i, j]).

- Если e лежит на Path(i, j) тогда Multi{i, j}[e] = -2 => очевидно
- Если e лежит на Path(i, k), удалим ребро e, листьев в поддереве содержащем l, k меньше, чем листьев в поддереве
  содержащем листья i, j => Mult_{k, l}[e] < Mult{i, j}[e]
- Если е не лежит ни на пути из (i, j) ни на пути из (i, k), тогда кратность будет одинаковая

Так как хотя бы одно ребро второго случая найдется (У нас нет ребер нулевой длины, мы так попросили), D*[k, l] будет
меньше => **чтд**

## 19. Построение эволюционных деревьев на основе признаков. Парсимония.

Есть крылья? Нет крыльев? Анализ признаков (characters)

Dollo's principle of irreversibility: эволюция не восстанавливает, что она отбросила
(филогения свидетельствует об обратном)

парсимония - экономия эволюционных действий (природа не тратит время зря)

Сейчас вместо матриц признаков используют последовательности нуклеотидов

![img](http://bioinformaticsalgorithms.com/images/Evolution/four_species_small_parsimony_1.png)

Given a tree T with every node labeled by a string of length m, we will therefore set the length of edge (v, w) equal to
the number of substitutions (Hamming distance) between the strings labeling v and w. The parsimony score of T is the sum
of the lengths of its edges (see the figure below).

#### Small Parsimony Problem.

Find the most parsimonious labeling of the internal nodes of a rooted tree.

- **Input:** A rooted binary tree with each leaf labeled by a string of length m.
- **Output:** A labeling of all other nodes of the tree by strings of length m that minimizes the parsimony score of the
  tree.

Упрощение: сейчас в каждой вершине написана строка из признаков, давайте разделим на несколько независимых деревьев, по
букве в каждом дереве (разделили независимые признаки)

Решается динамикой, для вершины v и поддерева T_{v}, будем считать s[k][v] - минимальную стоимость поддерева T_{v}, при
условии что сама вершина v помечена символом k.

![img](https://i.ibb.co/wN3BF3W/Screenshot-2022-01-09-at-17-48-09.png)

![img](http://bioinformaticsalgorithms.com/images/Evolution/Sankoff_step_2.png)

When the position of the root in the tree is unknown, we can simply assign the root to any edge that we like, apply
SmallParsimony to the resulting rooted tree, and then remove the root. It can be shown that this method provides a
solution to the following problem.

#### Small Parsimony Problem in an Unrooted Tree Problem:

Find the most parsimonious labeling of the internal nodes in an unrooted tree.

- **Input:** An unrooted binary tree with each leaf labeled by a string of length m.
- **Output:** A labeling of all other nodes of the tree by strings of length m that minimizes the parsimony score of the
  tree.

Нам не всегда известен корень и дерево

#### Large Parsimony Problem:

Given a set of strings, find a tree (with leaves labeled by all these strings) having minimum parsimony score.

- **Input:** A collection of strings of equal length.
- **Output:** An unrooted binary tree T that minimizes the parsimony score among all possible unrooted binary trees with
  leaves labeled by these strings.

Unfortunately, it turns out that the Large Parsimony problem is NP-Complete, in part because the number of different
trees grows very quickly with respect to the number of leaves.

First, note that in any unrooted binary tree, the removal of an internal edge, along with the two internal nodes that
this edge connects, results in four subtrees, which we will call W, X, Y, and Z (see figure on the previous step). These
four subtrees can be combined into a tree in three different ways, which we denote WX|YZ, WY|XZ, and WZ|XY. These three
trees are called nearest neighbors; a nearest neighbor interchange operation replaces a tree with one of its nearest
neighbors.

![img](http://bioinformaticsalgorithms.com/images/Evolution/nearest_neighbor_interchange.png)

Повращаем всеми способами пока улучшается

```
NearestNeighborInterchange(Strings)
     score ← ∞
     generate an arbitrary unrooted binary tree Tree with |Strings| leaves
     label the leaves of Tree by arbitrary strings from Strings
     solve  the  Small Parsimony in an Unrooted Tree Problem for Tree
     label the internal nodes of Tree according to a most parsimonious labeling
     newScore ← the parsimony score of Tree
     newTree ← Tree
     while newScore < score
          score ← newScore
          Tree ← newTree
          for each internal edge e in Tree
               for each nearest neighbor NeighborTree of Tree with respect to the edge e
                    solve the Small Parsimony in an Unrooted Tree Problem for NeighborTree
                    neighborScore ← the minimum parsimony score of NeighborTree
                    if neighborScore < newScore
                         newScore ← neighborScore
                         newTree ← NeighborTree
     return newTree
```

#### Least-Squares Phylogeny

Можем построить дерево с матрицей максимально похожей на нашу матрицу

(для сравнения матриц посчитаем сумму квадратов элементов)

It turns out that for a specific tree T, it is easy to find edge lengths in T minimizing Discrepancy(T, D). Yet our
ability to minimize the sum of squared errors for a specific tree does not imply that we can efficiently solve the Least
Squares Distance-Based Phylogeny Problem, since the number of different trees grows very quickly as the number of leaves
in the tree increases. In fact, the Least Squares Distance-Based Phylogeny winds up being NP-Complete, and so we must
abandon the hope of designing a fast algorithm to find a tree that best fits a non-additive matrix. In the next two
sections, we will explore heuristics for constructing trees from non-additive matrices that solve this problem
approximately.

## 20. Кластеризация: алгоритм Лойда

Дрожжи использую глюкозу, чтобы превращать ее в этанол. Когда она заканчивается, они могут изменить механизм метаболизма
и начать питаться этанолом. Это называется Diauxic shift, это возможно только при наличии кислорода. Какие гены отвечают
за это?

Давайте выделим гены которые нам интересны, и замерим уровень их экспрессии в интересные нам моменты времени.

![img](http://bioinformaticsalgorithms.com/images/Clustering/ten_yeast_gene_matrix_highlighted.png)

#### Lloyd Algorithm

It first chooses k arbitrary distinct points Centers from Data as centers and then iteratively performs the following
two steps (see figure below):

- **Centers to Clusters:** After centers have been selected, assign each data point to the cluster corresponding to its
  nearest center; ties are broken arbitrarily.
- **Clusters to Centers:** After data points have been assigned to clusters, assign each cluster’s center of gravity to
  be the cluster’s new center.

![img](http://bioinformaticsalgorithms.com/images/Clustering/Lloyd.png)

If the Lloyd algorithm has not converged, the squared error distortion must decrease in any step, according to the
following reasoning:

- In a “Centers to Clusters” step, if a data point is assigned to a new center, then this point must be closer to the
  new center than its previous center. Thus, the squared error distortion must decrease.
- In a “Clusters to Center” step, if a center is updated as a cluster’s center of gravity, then by the Center of Gravity
  Theorem, the new center is the only point minimizing the squared error distortion for the points in its cluster. Thus,
  the squared error distortion must decrease.

Prove that the number of iterations of the Lloyd algorithm does not exceed the number of partitions of the data set into
k clusters.

```
k-Means++Initializer(Data, k)
    Centers ← the set consisting of a single randomly chosen point from Data
    while |Centers| < k 
        randomly select DataPoint from Data with probability proportional to d(DataPoint, Centers)^2 
        add DataPoint to Centers
    return Centers
```

Limitations:

![img](http://bioinformaticsalgorithms.com/images/Clustering/difficult_clustering_problems_correct_clustering.png)

One weakness with our formulation of the k-means Clustering Problem is that it forces us to make a “hard” assignment of
each point to only one cluster. This strategy makes little sense for midpoints, or points that are approximately
equidistant from two centers (see figure below).

## 21. Кластеризация EM алгоритм

Тактика:

(Data, ?, Parameters) → (Data, HiddenVector, Parameters)
→ (Data, HiddenVector, ?)
→ (Data, HiddenVector, Parameters')
→ (Data, ?, Parameters')
→ (Data, HiddenVector', Parameters')
→ ...

Скрытые переменные - кластеры которые соответствуют каждой точке. Параметры - средние значения распределений.

На шаге E - строим скрытые значения, на шаге M - строим параметры

Скрытая матрица в данном случае - для каждой точки набор вероятностей что она лежит в одном из кластеров

![img](https://i.ibb.co/vZdr8XT/Screenshot-2022-01-10-at-12-57-32.png)


![img](https://i.ibb.co/S7d8LVG/Screenshot-2022-01-12-at-14-11-46.png)
## 22. Иерархическая кластеризация

Иногда полезно разбивать объекты в иерархическую структуру

![img](http://bioinformaticsalgorithms.com/images/Clustering/dendogram_slices.png)

**UPGMA**

```
HierarchicalClustering(D, n)
    Clusters ← n single-element clusters labeled 1, ... , n 
     construct a graph T with n isolated nodes labeled by single elements 1, ... , n 
    while there is more than one cluster 
        find the two closest clusters Ci and Cj
        merge Ci and Cj into a new cluster Cnew with |Ci| + |Cj| elements
        add a new node labeled by cluster Cnew to T
        connect node Cnew to Ci and Cj by directed edges
        remove the rows and columns of D corresponding to Ci and Cj
        remove Ci and Cj from Clusters
        add a row/column to D for Cnew by computing D(Cnew, C) for each C in Clusters 
        add Cnew to Clusters 
    assign root in T as a node with no incoming edges
    return T
```

В этом варианте расстояние между кластерами может считаться как:

- Среднее расстояний всех пар точек в кластерах
- минимальное расстояние между двумя точками из кластеров
- много других вариантов

## 23. Префиксные и суффиксные деревья

How do we locate disease-causing mutations?

Последнее время создавалось много баз данных геномов человека. Вопрос: можем ли мы использовать эти references для
сборки новых геномов? Почему не использовать Assembly? (это долго, в клинике не сделаешь)

#### Multiple Pattern Matching Problem:

Find all occurrences of a collection of patterns in a text.

- **Input:** A string Text and a collection Patterns containing (shorter) strings.
- **Output:** All starting positions in Text where a string from Patterns appears as a substring.

O(|sum of length of paterns| * |text|) это слишком долго.

To this end, we will consolidate Patterns into a directed tree called a trie (pronounced “try”), which is written Trie(
Patterns) and has the following properties (see the figure on the next step).

- The trie has a single root node with indegree 0, denoted root; all other nodes have indegree 1.
- Each edge of Trie(Patterns) is labeled with a letter of the alphabet.
- Edges leading out of a given node have distinct labels.
- Every string in Patterns is spelled out by concatenating the letters along some path from the root downward.
- Every path from the root to a leaf, or node with outdegree 0, spells a string from Patterns.

![img](http://bioinformaticsalgorithms.com/images/BWT/trie.png)

```
TrieMatching(Text, Trie)
    while Text is nonempty
        PrefixTrieMatching(Text, Trie)
        remove first symbol from Text
```

We need |Patterns| steps to construct Trie(Patterns), which contains at most |Patterns|+1 nodes. Each iteration of
PrefixTrieMatching takes at most |LongestPattern| steps, where LongestPattern is the longest string in Patterns.
TrieMatching makes |Text| total calls to PrefixTrieMatching, making the total number of steps equal to |Patterns| +
|Text| · |LongestPattern|.

Такое дерево занимает много места (памяти), хотя может работать быстро (например если использовать Ахо-Корасик, с
суффиксными ссылками)

Давайте строить дерево не из patterns, а из всех суффиксов genome

![img](http://bioinformaticsalgorithms.com/images/BWT/suffix_trie_threading.png)

Accordingly, the runtime and memory required to construct SuffixTrie(Text) are both equal to the combined length of all
suffixes in Text. There are |Text| suffixes of Text, ranging in length from 1 to |Text| and having total length |Text|·(
|Text|+1)/2, which is O(|Text|2). Thus, we need to reduce both the construction time and memory requirements of suffix
tries to make them practical.

Let’s not give up hope on suffix tries. We can reduce the number of edges in SuffixTrie(Text) by combining the edges on
any non-branching path into a single edge. We then label this edge with the concatenation of symbols on the consolidated
edges, as shown in the figure below. The resulting data structure is called a suffix tree, written SuffixTree(Text).
Число вершин в таком дереве < 2 * |Genome| (потому что число внутренних вершин дерева с n листьями < n, доказывается по
индукции)

![img](http://bioinformaticsalgorithms.com/images/BWT/suffix_tree.png)

## 24. Преобразование Барроуза-Уилера

На практике константа у построения суффиксного дерева за линию большая. (у памяти тоже)

Что сделать чтобы представить геном компактно?

To answer this question, we will digress to consider the seemingly unrelated topic of text compression. In one simple
compression technique called run-length encoding, we replace a run of k consecutive occurrences of symbol s with only
two symbols: k, followed by s. For example, run-length encoding would compress the string TTTTTGGGAAAACCCCCCA into
5T3G4A6C1A.

Run-length encoding works well for strings having lots of long runs, but genomes do not have many runs. What they do
have is repeats, as we learned when assembling genomes. It would therefore be nice if we could first manipulate the
genome to convert repeats into runs and then apply run-length encoding to the resulting string.

**Constructing the Burrows-Wheeler transform**

Similarly to suffix arrays — order all the cyclic rotations of Text lexicographically to form a |Text| × |Text| matrix
of symbols that we call the Burrows-Wheeler matrix and denote by M(Text). The figure below illustrates the construction
of M(Text).

Последняя строка матрицы будет результатом нашего преобразования.

![img](http://bioinformaticsalgorithms.com/images/BWT/Burrows-Wheeler.png)

Идея в том что много одинаковых слов встречаются в тексте и например перед nd почти всегда идет а (and)

![img](http://bioinformaticsalgorithms.com/images/BWT/BWT_Watson_Crick.png)

Как разжать преобразование? Во-первых мы знаем первую строку матрицы - это посорченные буквы. Во-вторых у нас есть
следующее свойство:

**The First-Last Property**
![img](http://bioinformaticsalgorithms.com/images/BWT/first-last-3.png)

Буквы в первой строке идут в таком же порядке, как буквы в последней строке.

Это поможет нам восстановить всю последовательность

![img](http://bioinformaticsalgorithms.com/images/BWT/bwt-inversion-3.png)

![img](http://bioinformaticsalgorithms.com/images/BWT/inverting_burrows_wheeler_1.png)

![img](http://bioinformaticsalgorithms.com/images/BWT/inverting_burrows_wheeler_2.png)

**Pattern matching**

Ищем подстроку ana, причем начинаем с конца - со второй a

![img](http://bioinformaticsalgorithms.com/images/BWT/top_bottom_pointers.png)

In order to update the top and bottom pointers, we need to determine where "n1" and "n3" occur in FirstColumn. The
Last-to-First array, denoted LastToFirst(i), answers the following question: given a symbol at position i in LastColumn,
what is its position in FirstColumn?

![img](http://bioinformaticsalgorithms.com/images/BWT/bwt_pattern_matching-6.png)

```
 BWMatching(LastColumn, Pattern, LastToFirst)
        top ← 0
        bottom ← |LastColumn| − 1
        while top ≤ bottom
            if Pattern is nonempty
                symbol ← last letter in Pattern
                remove last letter from Pattern
                if positions from top to bottom in LastColumn contain an occurrence of symbol
                    topIndex ← first position of symbol among positions from top to bottom in LastColumn
                    bottomIndex ← last position of symbol among positions from top to bottom in LastColumn
                    top ← LastToFirst(topIndex)
                    bottom ← LastToFirst(bottomIndex)
                else
                    return 0
            else
                return bottom − top + 1
```

Это можно ускорить если использовать всякие хеш-мапы для подсчета числа символов в последней колонке, а потом перейти к
подсчету символов в первой колонке:

```
top ← FirstOccurrence(symbol) + Countsymbol(top, LastColumn)
bottom ← FirstOccurrence(symbol) + Countsymbol(bottom + 1, LastColumn) − 1
```

Потом сказал что для поиска самих матчей нужно просто хранить суффиксный массив

![img](http://bioinformaticsalgorithms.com/images/BWT/bwt_array_suffix_array.png)

Тут можно экономить память, хранить только разреженный суффиксный массив (только элементы кратные K), но тогда поиск
конкретных значений будет требовать хождения от найденной позиции до ближайщей кратной K

![img](http://bioinformaticsalgorithms.com/images/BWT/partial_suffix_array.png)

Можно еще хранить не все, а только некоторые строки массива Count:

- What about runtime? Using checkpoint arrays, we can compute the top and bottom pointers in a constant number of
  steps (i.e., fewer than C). Since each Pattern requires at most |Pattern| pointer updates, the modified
  BetterBWMatching algorithm now requires O(|Patterns|) runtime, which is the same as using a trie or suffix array.

- Furthermore, we now only have to store the following data in memory: BWT(Text), FirstOccurrence, the partial suffix
  array, and the checkpoint arrays. Storing this data requires memory approximately equal to 1.5 · |Text|. Thus, we have
  finally knocked down the memory required for solving the Multiple Pattern Matching Problem for millions of sequencing
  reads into a workable range.

Если наш шаблон встречается в тексте с d опечатками, тогда существует разбиение шаблона на d + 1 "равных" частей такое
что найдется хотя бы один точный match. Давайте как-то разобьем на d + 1 частей, будем искать точные вхождения и если
найдем приложим к этому месту

Можно использовать BWT опять же. Давайте в процессе поиска вхождений разрешать брать неправильные буквы в последнем
столбце, при этом одновременно считать сколько mismatches уже совершили.

![img](http://bioinformaticsalgorithms.com/images/BWT/backtracking_bowtie.png)

In practice, this heuristic faces complications. We do not want to start allowing mismatched strings at the early stages
of BetterBWMatching, or else we will have to consider too many frivolous candidate strings. We may therefore require
that a suffix of Pattern of some threshold length matches Text exactly. Moreover, the method becomes time-intensive when
using large values of d, as we must explore many inexact matches. Practical applications often limit the value of d to
at most 3.

## 25. Выравнивание последовательностей. Алгоритм Витерби

Classical vaccines against viruses are often made from the surface proteins of a virus. These vaccines stimulate the
human immune system to recognize viral envelope proteins as foreign, destroy them, and keep a record of it, so that the
immune system can identify and eradicate the virus in a later encounter.

However, HIV viral envelope proteins are extremely variable because the virus must mutate rapidly in order to survive (
see DETOUR: The Red Queen Effect). The HIV population in a single infected individual rapidly evolves to evade the human
immune system (see Figure below), not to mention that HIV strains taken from different patients represent multiple
highly diverged subtypes. Therefore, a successful HIV vaccine must be broad enough to account for this variability.

![img](http://bioinformaticsalgorithms.com/images/HMM/gp120_alignment.png)

The figure below shows a motif logo from the V3 loop of gp120 and illustrates that some positions in gp120 are
relatively conserved, whereas others are extremely variable. Furthermore, this motif logo does not account for
insertions and deletions, which are prevalent in other regions of gp120 that are less conserved than the V3 loop. These
insertions and deletions make analyzing gp120 even more complex.

Because the columns of this multiple alignment have varying levels of conservation, we question the wisdom of using the
same amino acid scoring matrix (as well as indel penalties) across different columns of an alignment. A better approach
would use a different scoring approach across different columns. For example, an amino acid differing from R in position
3 of the alignment below should incur a larger penalty than an amino acid deferring from S in position 11.

![img](http://bioinformaticsalgorithms.com/images/HMM/V3_alignment_logo.png)

In other words, the problem formulation of multiple sequence alignment introduced in a previous chapter does not offer
an adequate translation of the biological problem of HIV classification into an algorithmic problem.

Допустим у нас есть новый белок и мы хотим сопоставить его с семейством выравниваний. Эту задачу мы научимся решать.

Тут идет длинная подводка с Якудзой и честной / нечестной монеткой. Есть монетки с разными параметрами, хотим определять
на каждом шаге какая была монетка.

GC-content: combined frequency of G and C in the genome. В человеческом геноме это 42%. Тогда казалось бы CC, CG, GC, GG
мы должны встречать с частотой (0.21)^2 ~ 4%, но на самом деле CG встречается только 2%

Но! Так происходит не везде, от methylation мутаций защищены CG-islands (CG appears frequently)

A naive approach to search for CG-islands in a genome would slide a window down the genome, declaring windows with
higher frequencies of CG as potential CG-islands. The disadvantages of this method are analogous to those of using a
sliding window to determine which coin the crooked dealer most likely used at any given point in time. We do not know
how long the window should be, and overlapping windows may simultaneously classify the same genomic position as
belonging to a CG-island and as not belonging to a CG-island.

Превратим дилера в машину. After each step, the machine makes two decisions:

- Which hidden state will I move to next?
- Which symbol will I emit in that state?

Получили HMM.

In general, an HMM (∑, States, Transition, Emission) is defined by a set of four objects:

- an alphabet Σ of emitted symbols;
- a set States of hidden states;
- a |States| × |States| matrix Transition = (transitionl, k) of transition probabilities, where transitionl, k
  represents the probability of moving from state l to state k;
- a |States|×|Σ| matrix Emission = (emissionk(b)) of emission probabilities, where emissionk(b) represents the
  probability of emitting symbol b from alphabet Σ when the HMM is in state k.

![img](http://bioinformaticsalgorithms.com/images/HMM/HMM_diagram_complete.png)

A hidden path π = π1 ...πn in an HMM is the sequence of states that the HMM passes through; such a path corresponds to a
path of solid edges in the HMM diagram. We can now rephrase the improperly formulated Casino Problem as finding the most
likely hidden path π for a string x of symbols emitted by an HMM.

![img](http://bioinformaticsalgorithms.com/images/HMM/sequence_hidden_paths.png)

#### Decoding Problem:

Find an optimal hidden path in an HMM given a string of its emitted symbols.

- **Input:** A string x = x1 . . . xn emitted by an HMM (Σ, States, Transition, Emission).
- **Output:** A path π that maximizes the probability Pr(x, π) over all possible paths through this HMM.

In 1967, Andrew Viterbi used an HMM-inspired analog of a Manhattan-like grid to solve the Decoding Problem. For an HMM
emitting a string of n symbols x = x1 . . . xn, the nodes in the HMM’s Viterbi graph are divided into |States| rows and
n columns (see figure below).

Weight_{i}(l, k) = transition[π[i - 1], π[i]] * emission[π[i], x[i]]

![img](http://bioinformaticsalgorithms.com/images/HMM/Viterbi_graph_three_states_source_sink.png)

#### The Viterbi algorithm

We will apply a dynamic programming algorithm to solve the Decoding Problem. First, define s[k,i] as the product weight
of an optimal path (i.e., a path with maximum product weight) from source to the node (k, i). The Viterbi algorithm
relies on the fact that the first i − 1 edges of an optimal path from source to (k, i) must form an optimal path from
source to (l, i − 1) for some (unknown) state l. This observation yields the following recurrence:

![img](https://i.ibb.co/NrcNHpL/Screenshot-2022-01-10-at-21-40-49.png)

## 26. Выравнивание последовательностей. Профиль HMM

Given a family of related proteins, we can check whether a newly sequenced protein belongs to this family by aligning a
new protein to all members of the family at once. (В общем, у нас есть много семейств протеинов которые уже содержат
построенные выравнивания внутри семейства, нам дан новый протеин мы хотим его куда-то присобачить, т.е выравнять к
какому-то из семейств)

Давайте посмотрим на выравнивание наших последовательностей (для какого-то семейства) и выкинем из него ненужные
столбцы (в которых пропусков например больше чем константа)

Построим матрицу профиля для получившегося выравнивания

![img](http://bioinformaticsalgorithms.com/images/HMM/multiple_alignment_2.png)

Заведем по состоянию M[i] для каждого столбца, тогда столбец матрицы профиля это вероятность, что это состояние выдаст
определенный символ (emit matrix)

Не забудем о возможности удалений или вставку в потенциальную строку которую мы хотим выравнять

!вся сложность билета это нарисовать эту модельку, выучи ее (или пойми)

![img](http://bioinformaticsalgorithms.com/images/HMM/alignment_HMM_paths.png)

Как посчитать вероятность перехода и выдачи символов? Transition оцениваются с помощью строк которые нам уже даны (
строки выравниваний). Посчитаем для каждого перехода его вероятность в нашем графе (ну просто счетчики по всем путям из
примеров). С Emission точно также, для каждой вершины поймем сколько раз какую штуку мы выдавали.

(ну понятно что совсем нули нельзя ставить в не посещенные вершины, нужно сгладить, но очевидно только для существующих
ребер, из D1 в D5 перейти нельзя) (причем pseudocounts добавляют следующим образом:
в нули записывают sigma = 0.01 например и потом нормализируют матрицу (кек))

Осталась последняя проблема, у нас на самом деле состояния D_i они silent. По опредлению #columns = # emitted symbols,
но мы можем взять другой путь и для него построить другой граф (другой длины в ширину)

![img](http://bioinformaticsalgorithms.com/images/HMM/AEFDFDC_path_1.png)

Для того чтобы избавиться, будем делать удаления ребрами вниз

![img](http://bioinformaticsalgorithms.com/images/HMM/profile_HMM_vertical_deletion_edges.png)

There is still a minor flaw with the graph from the previous step. If the HMM moves from the initial state into
Deletion(1), then the HMM will move through the first column without emitting a symbol. We will therefore transform the
initial state into a column of silent states containing the initial state and all deletion states (figure below). This
way, if the HMM enters Deletion(1) from the initial state, it can move downward through deletion states before
transitioning to a match or insertion state in the first column.

![img](http://bioinformaticsalgorithms.com/images/HMM/profile_HMM_Viterbi_complete.png)

## 27. Обучение Витерби. Алгоритм Баума-Уэлча

Thus far, our analysis has assumed that we know the parameters of an HMM, i.e., its transition and emission
probabilities. We have described a naive — and not necessarily optimal — heuristic for choosing these parameters for a
profile HMM, but it is not clear how to select parameters for an arbitrary HMM.

We will collectively refer to the matrices Transition and Emission as Parameters. Our goal is to find Parameters and π
when we are only given the emitted string x.

Если мы знаем x и π, мы можем просто посчитать частоту каждого ребра и каждого символа. Таким образом появляется простой
алгоритм обучения
**Viterbi learning**, (x, ?, Parameters) → (x, π, Parameters)  → (x, π, ?)
→ (x, π, Parameters′)  → (x, ?, Parameters′)
→ (x, π′, Parameters′)  → (x, π′, ?)
→ (x, π′, Parameters′′) → . . .

Такое обучение не мягкое, мы генерируем один оптимальный путь и потом с помощью него жестко обновляем параметры.

In the case of an arbitrary HMM, we would like to compute the conditional probability Pr(πi = k|x) that the HMM was in
state k at time i given that it emitted string x.

![img](https://i.ibb.co/9b93sVs/Screenshot-2022-01-11-at-10-31-11.png)

Чтобы посчитать backward, считаем такую же динамику как для forward, но из стока по обратным ребрам.

Prove that Pr(πi = l, πi+1 = k|x) is equal to forwardl, i · Weighti(l, k) · backwardk, i+1/forward(sink).

![img](http://bioinformaticsalgorithms.com/images/HMM/forward_edge.png)

The probabilities Pr(πi = k|x) can be put into a |States| × n responsibility matrix Π∗, where Π∗k, i corresponds to a
node in the Viterbi graph and is equal to Pr(πi = k|x). The figure below (top) shows the “responsibility” matrix Π∗ for
the crooked casino.

The probabilities Pr(πi = l, πi+1 = k|x) can be put into another |States| × |States| × (n − 1) responsibility matrix
Π∗∗, where Π∗∗l, k, i corresponds to an edge in the Viterbi graph and is equal to Pr(πi = l, πi+1 = k|x) (figure below (
bottom)). For brevity, we use Π to collectively refer to the matrices Π∗ and Π∗∗.

The expectation maximization algorithm for parameter estimation, called **Baum-Welch learning**, alternates between two
steps. In the E-step, it estimates the responsibility profile Π given the current parameters:

(x, ?, Parameters) → Π

Then, in the M-step, it re-estimates the parameters from the responsibility profile:

(x, Π, ?) → Parameters

![img](https://i.ibb.co/hsVxBSH/Screenshot-2022-01-11-at-10-38-34.png)

![img](https://i.ibb.co/xGBv0Wb/Screenshot-2022-01-11-at-10-39-15.png)

## 28. Секвенирование и идентификация линейных пептидов

For example, suppose we are studying the chicken ribosome, a complex molecular machine consisting of many proteins. Knowing the chicken proteome does not tell us which specific proteins compose the ribosome complex. Instead, we can isolate the ribosome, break it apart, and identify which proteins it contains. In practice, merely confirming that a 10 amino acid-long peptide from a known chicken protein is present in a sample is usually sufficient to confirm this protein’s presence in the sample. The process of confirming that a peptide from a known proteome is present in a sample is called peptide identification. But how could we form a T. rex proteome?

Although peptide identification dominates modern proteomics studies, the proteomes of many species, including extinct species like T. rex, remain unknown. In this case, biologists rely on de novo peptide sequencing, or inferring the amino acid sequence of a peptide without relying on a proteome, and so this is where we will begin.

Мы уже изучали задачу секвенирования циклических пептидов. Теперь хотим посмотреть на линейные, отличие в том, что теперь нам непонятно где префикс, а где суффикс пептида. 

Построим граф с массами которые мы получили и между вершинами проведем ребра аминокислоты

![img](http://bioinformaticsalgorithms.com/images/Proteomics/graph_ideal_spectrum.png)

Важно что не каждый путь в таком графе описывает нужный пептид. (После того как нашли путь давайте проверим спектр еще раз)
Все пути искать в DAG сложно (хотя вроде дфсом можно). 

На практике много лишних пропущенных шумных масс. А еще после разбиения пептиды могут терять например молекулы воды. 

На практике у нас пики с разной интенсивностью и мы хотим выбрать как можно более подходящее объяснение. Дорлжны ли мы использовать метрику
shared peak count (ignores intensities) или sum of intensities of explained peaks (large peaks will dominate score)

Although the shared peak count works better than the intensity count in practice, it is still far from ideal; a better approach would account for intensities of peaks without letting the tallest peaks dominate the score. To achieve this goal, we will convert peptides and spectra into vectors, then define a scoring function that is the dot product of these vectors.

First, given an amino acid string Peptide = a1 . . . an of length n, we will represent its prefix masses using a binary peptide vector Peptide' with Mass(Peptide) coordinates. This vector contains a 1 at each of the n prefix coordinates

- Mass(a1), Mass(a1a2), . . . , Mass(a1a2 . . . an) ,
- and it contains a 0 in each of the remaining noise coordinates. 

В свою очередь спектр массы m превратим в **spectral vector**: 
- s1, s2, .., sm
- the values si (amplitude) approximates the likelihood that mass i is the prefix mass of an peptide that generated the spectrum

Почему префикс? Indeed, any peak of mass s in a spectrum may be interpreted as either a prefix mass or suffix mass of an unknown peptide Peptide that generated the spectrum. Moreover, its twin peak, with mass equal to Mass(Peptide) − s, may be interpreted as either a mass of a suffix or a mass of a prefix of the same peptide.
To deal with this uncertainty, mass spectrometrists convert a spectrum Spectrum into a spectral vector Spectrum' that consolidates information about the intensities of each peak and its twin into a single value, called an amplitude, at the coordinate representing the mass of the hypothetical prefix peptide for these twins. Why? Because algorithms interpreting a consolidated spectrum become easier than algorithms attempting to account for both twins (see "DETOUR: The Anti-Symmetric Path Problem"). To make matters more complicated, amplitudes in the spectral vectors also account for intensities of ion fragments with various charges and intensities of ion fragments with water and ammonia molecule loss.

After a peptide Peptide has been transformed into a peptide vector Peptide' = (p1, . . . , pm) and a spectrum Spectrum has been transformed into a spectral vector Spectrum' = (s1, . . . , sm) of the same length, we define Score(Peptide, Spectrum) = Score(Peptide', Spectrum') as the dot product of Peptide' and Spectrum',

- Score(Peptide, Spectrum) = p1 · s1 + . . . + pm · sm .

In the remainder of this chapter, we will work with spectral vectors instead of spectra. Given a spectral vector Spectrum', our goal is to find a peptide Peptide maximizing Score(Peptide', Spectrum'). 

Given a spectral vector Spectrum' = (s1, . . . , sm), we will construct a DAG on m + 1 nodes, indexed with the integers from 0 (source) to m (sink), and then connect node i to node j by a directed edge if j − i is equal to the mass of an amino acid (see figure below). 

![img](http://bioinformaticsalgorithms.com/images/Proteomics/spectral_DAG.png)

Despite many attempts, researchers have still not devised a scoring function that reliably assigns the highest score to the biologically correct peptide, i.e., the peptide that generated the spectrum. Fortunately, although the correct peptide often does not achieve the highest score among all peptides, it typically does score highest among all peptides limited to the species’s proteome. As a result, we can transition from peptide sequencing to peptide identification by limiting our search to peptides present in the proteome, which we concatenate into a single amino acid string Proteome.

Попробуем оценить значимость идентификации пептидов.  

If Score(Peptide', Spectrum') is greater than or equal to threshold, then we conclude that Peptide is present in the sample and call the pair (Peptide, Spectrum') a peptide-spectrum match (PSM). The resulting collection of PSMs for SpectralVectors is denoted PSMthreshold(Proteome, SpectralVectors).

#### PSM Search Problem:
Identify all peptide-spectrum matches scoring above a threshold for a set of spectra and a proteome.

Input: A set of spectral vectors SpectralVectors, an amino acid string Proteome, and an integer threshold.
Output: The set PSMthreshold(Proteome, SpectralVectors).

## 29. Спектральные словари

Чтобы оценить значимость результатом, можно решить задачу PSM Search против случайного протеома.
В таком DecoyProteome вообще не должно быть PSM, поэтом процент найденных в нем это процент ошибок нашего метода.

Хотим посчитать False Discovery Rate: |PSM_threshold(DecoyProteome, SpectralVectors)| / |PSM_threshold(Proteome, SpectralVectors)| 

Как оценить sifnificance of i ndividual PSM? Хотим оценить ожидаемое число строк из нашего словаря, которое встретится в сгенерированном тексте.
(Оцениваем)

Now imagine that instead of a monkey typing words, we have an algorithm generating the set of all peptides scoring at least threshold against a spectral vector Spectrum'. We will henceforth call this set of high-scoring peptides a spectral dictionary, denoted

- Dictionary_{threshold}(Spectrum') .

For a PSM (Peptide, Spectrum'), we will use the term PSM dictionary, denoted Dictionary(Peptide, Spectrum'), to refer to the spectral dictionary

- Dictionary_{Score(Peptide, Spectrum')} (Spectrum') .

#### Expected Number of High-Scoring Peptides Problem:
Find the expected number of high-scoring peptides against a given spectrum in a decoy proteome.

- **Input:** A spectral vector Spectrum and integers threshold and n (length of a decoy proteome).
- **Output:** E(Dictionary_threshold(Spectrum', n)).

Вероятность вхождения фиксированного пептида в протеом 1 / 20 ^ |peptide|, тогда E(Dictionary) = n * sum(for each peptide in dictionary, P(peptide))

Такую штуку sum(for each peptide in dictionary, P(peptide)) назовем вероятностью словаря P(Dictionary)

#### Probability of Spectral Dictionary Problem:
Find the probability of a spectral dictionary for a given spectrum and score threshold.

- **Input:** A spectral vector Spectrum and an integer threshold.
- **Output:** The probability of Dictionarythreshold(Spectrum') .

We will first compute the number of peptides in a spectral dictionary, since this simpler problem will provide insights on how to compute the probability of a spectral dictionary.

#### Size of Spectral Dictionary Problem:
Find the size of the spectral dictionary for a given spectral vector and score threshold.

- **Input:** A spectral vector Spectrum' and an integer threshold.
- **Output:** The number of peptides in Dictionarythreshold(Spectrum').

We will use dynamic programming to solve the Size of Spectral Dictionary Problem. Given a spectral vector Spectrum' = (s1 , . . . , sm), we define its i-prefix (for i between 1 and m) as Spectrumi = (s1 , . . . , si ) and introduce a variable Size(i, t) as the number of peptides Peptide of mass i such that Score(Peptide, Spectrum'i ) is equal to t.

Size(i,t)= ∑ all amino acids a  Size(i−∣a∣,t−s[i])

Аналогичено для вероятнотсей. 

![img](https://i.ibb.co/ZcLgMmL/Screenshot-2022-01-11-at-23-03-33.png)

## 30. Модифицированные пептиды. Спектральное выравнивание. 

The PSMSearch algorithm can only identify a peptide if it occurs in a proteome without mutations. Yet some of the peptides in the dinosaur experiment, reproduced below,  are mutated.

In addition to searching for mutated peptides, we will also need to search for post-translational modifications, which alter amino acids after a protein has been translated from RNA. In fact, most proteins are modified after translation, and hundreds of types of modifications have been discovered.

A modification of mass d applied to an amino acid results in adding d to the mass of this amino acid.

If d is positive, then the resulting modified peptide has a peptide vector that differs from the original peptide vector Peptide' by inserting a block of d zeroes before the i-th occurrence of 1 in Peptide'. In the more rare case that d is negative, the modified peptide corresponds to deleting a block of |d| zeroes from Peptide' (figure below).

![img](http://bioinformaticsalgorithms.com/images/Proteomics/block_indels.png)

#### Spectral Alignment Problem: 

Given a peptide and a spectral vector, find a modified variant of this peptide that maximizes the peptide-spectrum score, among all variants of the peptide with up to k modifications.

- **Input:** An amino acid string Peptide, a spectral vector Spectrum', and an integer k.
- **Output:** A peptide of maximum score against Spectrum' among all peptides in Variantsk(Peptide).

Решается динамикой. 

Сделаем решетку Peptide = a1..an of mass m and Peptide[mode] = a1 ... an' of mass m + D

- Каждая вершина соответствует какой-то массе. 
- Диагональные ребра под 45 градусов - нормальные аминокислоты, которые были в оригинальном пептиде
- Пунктирные ребра - аминокислоты поменяли вес 


![img](http://bioinformaticsalgorithms.com/images/Proteomics/peptide_variants_graph.png)

На самом деле ненужные ряды могут быть удалены

![img](http://bioinformaticsalgorithms.com/images/Proteomics/PSM_graph.png)

Нам нужно следить, чтобы было не более чем k пунктирных ребер. Давайте разобъем граф на k + 1 слоев, теперь пунктирные ребра опускают нас на слой ниже

![img](http://bioinformaticsalgorithms.com/images/Proteomics/spectral_alignment_graph.png)

The maximum score of all peptides with at most k modifications is therefore the maximum value of Score(m, m + D, t) as t ranges from 0 to k.
(В итоге нам нужно найти путь максимального веса, причем по вершинам - они у нас взвешенные, который завершается в углу одного из уровней)

![img](http://bioinformaticsalgorithms.com/images/Proteomics/spectral_alignment_graph.png)

Вот такая динамика получилась, не сдохнуть бы ее писать