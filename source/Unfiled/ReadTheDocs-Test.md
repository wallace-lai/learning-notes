# ReadTheDocs功能测试

作者：wallace-lai </br>
发布：2023-11-22 </br>
更新：2023-11-30 </br>

## 1. sphinxcontrib-mermaid

### 1.1 sequence diagram
```{mermaid}
sequenceDiagram
  participant Alice
  participant Bob
  Alice->John: Hello John, how are you?
  loop Healthcheck
      John->John: Fight against hypochondria
  end
  Note right of John: Rational thoughts <br/>prevail...
  John-->Alice: Great!
  John->Bob: How about you?
  Bob-->John: Jolly good!
```
## 2. mathjax

### 2.1 One Line
$a^2 + b^2 = c^2$

### 2.2 MultiLine

Maxwell's Equations :

$$
\begin{aligned}
\nabla \cdot D &= \beta \\
\nabla \cdot B &= 0 \\
\nabla \times E &= - \frac{\partial B}{\partial t} \\
\nabla \times H &= J + \frac{\partial D}{\partial t}
\end{aligned}
$$

## 3. myst-parser

### 3.1 tasklist

- [x] Step 1
- [ ] Step 2
- [ ] Step 3

