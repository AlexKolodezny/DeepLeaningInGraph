# DeepLeaningInGraph
## Текст
### 1 Слайд
Другим подходом к построения нейронных сетей на графах являются графовые свёрточные сети, в них как и в рекурентных нейронных сетях на каждом шаге вектор ембеддинга для вершины пересчитывается как функция он ембеддингов соседних вершин, однако на каждом слое обучаются свои параметры. В результате формула для одного слоя свёрточной сети выглядит так (формула), где Psi перестановочно-инвариантная функция принимающая эмбединги от соседей, а phi и psi маленькие это некоторые параметризованные функции, которые мы обучаем на данном слое.

### 2 Слайд
Примером конкретной такой функции, использованной в Graph Convolution Network, представлена такая ,которую можно получить используя спектральную теорию графов, здесь sigma --- сигмоида, W --- обучаемая матрица, L --- нормированный лапласиан. В качестве перестановочно-инвариантной функции используется сумма.

### 3 Слайд (Sampling)
Одной из проблем обучения нейронных сетей на графах является вычислительная сложность, особенно в тех случаях, когда граф является близким к плотному, в этом случае количество рёбер пропорционально квадрату количества вершин.
Одним из методов борьбы являются методы sampling-а. Например можно для каждой вершины выбирать на каждом слое случайное подмножество соседей, которые будут давать вклад в новый вектор для вершины, такой способ был предложен в архитектуре GraphSAGE. Он позволяет уменьшить вычислительную стоимость в случае плотных графов, но при этом всё равно приходится пересчитывать градиенты для всех вершин.
Другой способ, предложенный в архитектуре FastGCN, это выбрать просто подмножество вершин графа и оптимизировать ембединги только для полученного подграфа. Этот способ особенно эффективный, если вершин очень много, так как нам не надо пересчитывать градиенты для всех вершин.

### 4 Слайд (Pooling)
Другой способ ускорения обучения это добавление слоёв pooling-а, когда кластера вершин по определённой схеме объединяются в одну вершину, в результате чего уменьшаются размеры графа, что в принципе ускоряет обучение.
Pooling так же сам по себе решает задачу кластеризации графа.

### 5 Слайд (Differentiable Pooling)
Одним из вариантов pooling-а является differentiable pooling, который является адаптивным методом, то есть его можно обучить.
В нём для каждой вершины вычисляется матрица soft-membership вероятность попадания в кластер, получается она обучением отдельного свёрточного слоя или нескольких свёрточных слоёв и применением функции softmax.
Далее с помощью полученной матрицы надо пересчитать новые эмбединги для вершин (что можно записать через перемножение матриц) и новую матрицу смежности.
Можно заметить, что проблема этого алгоритма это то, что получается полная матрица смежности.

### 6 Слайд (Top-k Pooling)
Такой проблемы нет в другом алгоритме pooling-а - top-k pooling. В нём мы хотим выбрать множество вершин, которые останутся в нашем графе, для этого мы считаем величину attention для каждой вершины как проекция его embedding-а на обучаемый вектор p, что то же самое, что скалярное произведение с нормированным вектором p. То есть мы получаем вектор s с attention-ами. Далее мы выбираем top-k вершин с наибольшим attention-ом, и оставляем в графе только эти вершины и их рёбра, то есть делаем slice матриц.
Очевидным улучшением является метод self-attention pooling, когда мы имеем отдельную свёрточную сеть для подсчёта attention.

### 7 Слайд (Другие методы pooling-а)
Другой подход есть в edge pooling-е, где мы итерационно выбираем ребра, которые сжимаем и объединяем инцидентные вершины.
Другой подход к pooling-у это топологический pooling, который не является адаптивным, то есть он не обучается в процессе, это различные методы основанные на спектральном анализе матрицы смежности, такие как GRACLUS или на факторизации матрицы смежности.

### 8 Слайд (Graph embedding)
Иногда в задачах надо рассматривать граф целиком, а не вершины по отдельности, например, когда мы хотим делать классификацию графов (такая задача возникает например в химии, где молекулы представляют в виде графов, и мы определить их свойства).
Мы будем делать embedding графов агрегируя embedding-и нод, например можно делать pooling графа, пока не останется одна вершина.

### 9 Слайд (Graph pooling)
Но чаще агрегацию делают как применение перестановочно инвариантной функции от эмбедингов нод, как представлено на слайде, тогда это просто дополнительный слой в нейронной сети.
Самый простой пример такой функции можно получить если вязть Psi как сумму или максимум, а f просто тождественная. 

### 10 Слайд (Общая структура свёрточной сети на графах)
В результате если собрать все идеи, то мы можем получить примерный способ построения нейронной сети для графов. Схема изображена на рисунке. Нейронная сеть состоит из свёрточных слоёв, между которыми можно разместить слои pooling-а. Если мы делаем классификацию вершин, то нам достаточно сделать слой классификатора над embedding-ами, тот же softmax, а если мы делаем некоторую классификацию графа, то нам надо сделать агрегацию embedding-ов вершин и у уже на полученном ембединге сделать классификацию.