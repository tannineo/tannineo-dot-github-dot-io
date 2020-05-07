---
title: Hello World
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

<!-- more -->

## Code

```js
function quicksort(X, i, j) {
  if (i < j) {
    p = partition(X, i, j)
    quicksort(X, i, p - 1)
    quicksort(X, p + 1, j)
  }
}

function partition(X, i, j) {
  var pivot = X[j]
  var m = i
  for (var n = i; n < j; n++) {
    if (X[n] <= pivot) {
      swap(X, m, n)
      m += 1
    }
  }
  swap(X, m, j)
  return m
}

function swap(X, a, b) {
  var z = X[a]
  X[a] = X[b]
  X[b] = z
}

var X = []

for (var i = 0; i < 20; i++) {
  X.push(Math.floor(Math.random() * 100))
}

quicksort(X, 0, X.length - 1)
console.info(X)

```

## Math

The dollar sign `$` does not work with `hexo-math` using `katex`.

### Tags

```tex
This inline equation {% math %}\cos 2\theta = \cos^2 \theta - \sin^2 \theta{% endmath %}.
```

This inline equation {% math %}\cos 2\theta = \cos^2 \theta - \sin^2 \theta{% endmath %}.

```tex
{% math %}
  \begin{aligned}
    \dot{x} & = \sigma(y-x) \\
    \dot{y} & = \rho x - y - xz \\
    \dot{z} & = -\beta z + xy
  \end{aligned}
{% endmath %}
```

{% math %}
  \begin{aligned}
    \dot{x} & = \sigma(y-x) \\
    \dot{y} & = \rho x - y - xz \\
    \dot{z} & = -\beta z + xy
  \end{aligned}
{% endmath %}

## Others
