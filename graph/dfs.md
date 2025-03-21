# Поиск в глубину

Поиск в глубину (DFS) — один из базовых алгоритмов работы с графами.

Позволяет находить пути и обходить граф рекурсивно или с использованием стека.
В процессе работы может находить лексикографически первый путь, если вершины обходятся в порядке их номеров.
Работает за `O(n+m)`, где `n` — количество вершин, `m` — количество рёбер.

## Реализация

Реализуем вышеописанный алгоритм на языке JavaScript.

### Инициализация

Представим граф как двухмерных массив.
Каждое ребро будет массивом из двух элементов (вершин):
ребро из вершины `i` в вершины `j` запишем как `[i, j]`.
Сам граф будет массивом рёбер, например
```js
[
  [0, 1],
  [1, 2],
  [1, 3],
]
```

### Обход

```js
const g = ...; // граф
const n = ...; // число вершин

const color = []; // цвет вершины (0, 1, или 2)
const timeIn = [];
const timeOut = []; // "времена" захода и выхода из вершины

let dfsTimer = 0; // "таймер" для определения времён

function dfs(v) {
  timeIn[v] = dfsTimer++;
  color[v] = 1; // вершина в процессе обработки
    
  for (let i = 0; i < g[v].length; i++) {
    let to = g[v][i];
    
    if (color[to] === 0) {
      dfs(to);
    }
  }
    
  color[v] = 2; // вершина обработана
  timeOut[v] = dfsTimer++;
}
```

Это наиболее общий код.

Во многих случаях времена захода и выхода из вершины не важны,
так же как и не важны цвета вершин
(но тогда надо будет ввести аналогичный по смыслу булевский массив `visited`).

Вот наиболее простая реализация:

```js
const g = []; // граф (массив массивов)
const n; // число вершин

const visited = []; // массив посещённых вершин

function dfs(v) {
  visited[v] = true;
    
  for (let i = 0; i < g[v].length; i++) {
    const to = g[v][i];
        
    if (!visited[to]) {
      dfs(to);
    }
  }
}
```

## Применение

### Поиск любого пути в графе

### Поиск лексикографически первого пути в графе

### Проверка, является ли одна вершина дерева предком другой

В начале и конце итерации поиска в глубину будет запоминать "время" захода и выхода в каждой вершине.
Теперь за `O(1)` можно найти ответ: вершина `i` является предком вершины `j` тогда и только тогда,
когда `start[i] < start[j]` и `end[i] > end[j]`.

### Задача LCA (наименьший общий предок)

### Топологическая сортировка

Запускаем серию поисков в глубину, чтобы обойти все вершины графа.
Отсортируем вершины по времени выхода по убыванию — это и будет ответом.

### Проверка графа на ацикличность и нахождение цикла

### Поиск компонент сильной связности

Сначала делаем топологическую сортировку, потом транспонируем граф и
проводим снова серию поисков в глубину в порядке,
определяемом топологической сортировкой.

Каждое дерево поиска — сильносвязная компонента.

### Поиск мостов

Сначала превращаем граф в ориентированный, делая серию поисков в глубину,
и ориентируя каждое ребро так, как мы пытались по нему пройти.
Затем находим сильносвязные компоненты.

Мостами являются те рёбра,
концы которых принадлежат разным сильносвязным компонентам.
