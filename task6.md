F. Бинарная биометрия
Данная задача состоит в том, чтобы обучить бинарную нейронную сеть предсказывать пол говорящего человека по записи.

В терминах данной задачи бинарной нейронной сетью называется сеть, веса которой могут быть равны только 1 и -1, функцией активации является 
s
i
g
n
−
1
sign 
−1
​
 , а к выходу применяется функция 
s
i
g
n
0
sign 
0
​
 .

s
i
g
n
−
1
(
x
)
=
{
x
≤
0
,
−
1
x
>
0
,
1
sign 
−1
​
 (x)={ 
x≤0,−1
x>0,1
​
 

s
i
g
n
0
(
x
)
=
{
x
≤
0
,
0
x
>
0
,
1
sign 
0
​
 (x)={ 
x≤0,0
x>0,1
​
 

Архитектура данной бинарной сети задана заранее:

conv1d слой с n_in = 64, n_out=128, kernel=9, padding=0, принимает на вход матрицу размера 
[
t
,
64
]
[t,64] и отдаёт на выход матрицу размера 
[
t
−
8
,
128
]
[t−8,128]
maxpool1d c kernel = 2, принимает на вход матрицу размера 
[
t
−
8
,
128
]
[t−8,128] и отдаёт на выход матрицу размера 
[
(
t
+
1
)
/
/
2
−
4
,
128
]
[(t+1)//2−4,128]
s
i
g
n
−
1
sign 
−1
​
 , не изменяет размерности входной матрицу
conv1d слой c n_in=128, n_out = 128, kernel=9, padding=0, принимает на вход матрицу размера 
[
(
t
+
1
)
/
/
2
−
4
,
128
]
[(t+1)//2−4,128] и отдаёт на выход матрицу размера 
[
(
t
+
1
)
/
/
2
−
12
,
128
]
[(t+1)//2−12,128]
maxpool1d c kernel = 2, принимает на вход матрицу размера 
[
(
t
+
1
)
/
/
2
−
4
,
128
]
[(t+1)//2−4,128] и отдаёт на выход матрицу размера 
[
(
t
+
3
)
/
/
4
−
6
,
128
]
[(t+3)//4−6,128]
s
i
g
n
−
1
sign 
−1
​
 , не изменяет размерности входной матрицу
conv1d слой c n_in=128, n_out = 128, kernel=9, padding=0, принимает на вход матрицу размера 
[
(
t
+
3
)
/
/
4
−
6
,
128
]
[(t+3)//4−6,128] и отдаёт на выход матрицу размера 
[
(
t
+
3
)
/
/
4
−
14
,
128
]
[(t+3)//4−14,128]
s
i
g
n
−
1
sign 
−1
​
 , не изменяет размерности входной матрицу
maxpool1d полностью сжимающий размерность времени, принимает на вход матрицу размера 
[
(
t
+
3
)
/
/
4
−
14
,
128
]
[(t+3)//4−14,128] и отдаёт вектор размера 128
линейный слой с n_in=128, n_out=1, принимает на вход вектор размера 128 и возвращает одно число
s
i
g
n
0
sign 
0
​
 , приводит полученное число к классу 1 или 0
Функции sign используются только во время инференса квантизованной модели, во время обучения же можно использовать любую другую функцию, которую сочтёте нужной.

Тренировочный данные:

можно скачать по ссылке https://disk.yandex.ru/d/0duIDWFoTNOvoA
состоят из tsv файла, где в каждой строке содержится имя файла и соответствующая ему метка 0 или 1
файлов с признками в npy формате
Проверяющая система ожидает получить файл следующего вида:

состоит из -1 и 1
последовательно выводятся элементы для каждого слоя
внутри одного слоя сразу выводятся веса, а потом вектор bias.
веса свёрточного слоя задаются тензорами размера 
[
n
o
u
t
,
n
i
n
,
k
e
r
n
e
l
]
[n 
out
​
 ,n 
in
​
 ,kernel], веса линейного слоя тензорами размера 
[
n
o
u
t
,
n
i
n
]
[n 
out
​
 ,n 
in
​
 ]
в первую очередь происходит итерация по первой размерности, в последнюю очередь по последней, так 
i
i-ым выведенным элементом тензора весов свёрточного слоя является элемент с индексом 
[
i
/
(
n
i
n
∗
k
e
r
n
e
l
)
,
(
i
/
k
e
r
n
e
l
)
[i/(n 
in
​
 ∗kernel),(i/kernel)
Задача считается пройденной, если на тестовом сете получен accuracy > 0.96

Пример квантизации и сохранения модели для pytorch:

with open('result', 'w') as fout:

    for weights in model.parameters():

        for param in torch.where(torch.flatten(weights) > 0, 1, -1):

            fout.write(str(param.item()) + '\n')

