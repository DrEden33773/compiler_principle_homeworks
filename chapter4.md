# Chapter 4

## 讨论

### 0.1

> 为什么 `左递归` 消除后, 文法的 `右递归` 对 `自上而下` 的语法分析无害?

`自上而下语法分析器` 的另一个名称是 `LL语法分析器`, 具体的含义为:

- `L`: 从左到右扫描输入串
- `L`: 遵循最左推导

`LL` 分析中 `左递归` 致命性的灾难在于, 试图匹配一个具有 `左递归` 模式的 `产生式`, 会导致这个产生式不停的向左侧展开 (且没有终止条件), 使得用于匹配 `token` 的指针停滞不前 (匹配过程中, 匹配指针的移动方向是唯一固定的). 相反的, 只要成功匹配到了一个 `不具有左递归` 模式的 `产生式`, 分析器就会在记录后, 更新匹配指针.

`右递归` 的存在, 会在向右展开 `产生式` 的同时, 向右移动匹配指针, 而待匹配的 `字符串` 显然是有穷的. 那么 `右递归` 的存在, 还是可以保证匹配过程的 `有穷性`.

**换言之**, `右递归` 情况下, 每次展开都会使得右递归的非终结符出现在产生式的末尾, 使产生式的长度不断减小, 最终终止分析.

### 0.2

> 现在已知:

$$
A \to \alpha_1 | \alpha_2 | \alpha_3 | \dots | \alpha_n
$$

> 如果:

$$
First(\alpha_i) \cap First(\alpha_j) = \set{\epsilon}
$$

> 是否会对自上而下的语法分析造成影响?

并不会造成影响.

一般情况下, 确保候选式对应的 `First` 集合 `两两不交` 的目的在于, 让任意一个 `可能存在后继` 的 `输入符号`, 能够仅仅根据 `首字符` 等特征确定一个 `唯一候选` 作为匹配.

`空串` 意味着, 不存在任何有效的 `输入符号`, 既没有前驱也没有后继. 在这种情况下, 随意指派任意一个 `候选` 后, `LL 分析` 都会终止, 不会对结果的正确性产生影响.

## 作业

### 1.1

> 已知文法:

$$
\begin{align}
G(E): E &\to [F]E|[F] \\
F &\to Fi|i
\end{align}
$$

> 1. 消除该文法的左递归, 提取左公因子
> 2. 给出改造后文法的 `每一个非终结符` 的 `First` 和 `Follow` 集合
> 3. 给出改造后文法的 `预测分析表`

#### Case1: $[, ]$ 都是 `终结符`

$ T_1: $

$$
\begin{align}
G(E): E &\to [F]E' \\
E' &\to E| \epsilon \\
F &\to iF' \\
F' &\to iF'| \epsilon
\end{align}
$$

$ T_2: $

| $v$  |     $First(v)$     |  $Follow(v)$   |
| :--: | :----------------: | :------------: |
| $E$  |     $\set{[}$      | $\set{\sharp}$ |
| $E'$ | $\set{[,\epsilon}$ | $\set{\sharp}$ |
| $F$  |     $\set{i}$      |   $\set{]}$    |
| $F'$ | $\set{i,\epsilon}$ |   $\set{]}$    |

$ T_3: $

|      |      $[$      |     $i$      |        $]$        |     $\sharp$      |
| :--: | :-----------: | :----------: | :---------------: | :---------------: |
| $E$  | $E \to [F]E'$ |      ~       |         ~         |         ~         |
| $E'$ |  $E' \to E$   |      ~       |         ~         | $E' \to \epsilon$ |
| $F$  |       ~       | $F \to iF'$  |         ~         |         ~         |
| $F'$ |       ~       | $F' \to iF'$ | $F' \to \epsilon$ |         ~         |

#### Case2: $[, ]$ 是 `扩充的巴克斯范式` 中的 `特定符号`

**注意:** 经求证, 一般题目不做特殊说明, 所有产生式都不会启用 `扩充巴科斯范式` 的规则

$T_1$

此时, 先将题目改写为:

$$
\begin{align}
G(E): E &\to FE|E|F|\epsilon \\
F &\to Fi|i
\end{align}
$$

此时, 很不幸的是, $ E \to E | \epsilon $ 这个产生式, 结合上下文, 可以使用 `等价牺牲`, 完成 `消除左递归`:

$$
\begin{align}
G(E): E &\to FE | F | \epsilon \\
F &\to Fi|i
\end{align}
$$

然后就可以, 进一步 `消除左递归`, 提取左公因子了:

$$
\begin{align}
G(E): E &\to FE' | \epsilon \\
E' &\to E | \epsilon \\
F &\to iF' \\
F' &\to Fi | \epsilon
\end{align}
$$

$T_2$

| $v$  |     $First(v)$      |  $Follow(v)$   |
| :--: | :-----------------: | :------------: |
| $E$  | $\set{i, \epsilon}$ | $\set{\sharp}$ |
| $E'$ | $\set{i, \epsilon}$ | $\set{\sharp}$ |
| $F$  |      $\set{i}$      |   $\set{i}$    |
| $F'$ | $\set{i, \epsilon}$ |   $\set{i}$    |

$T_3$

|      |     $i$     |     $\sharp$      |
| :--: | :---------: | :---------------: |
| $E$  | $E \to FE'$ | $E \to \epsilon$  |
| $E'$ | $E' \to E$  | $E' \to \epsilon$ |
| $F$  | $F \to iF'$ |                   |
| $F'$ | $F' \to Fi$ | $F' \to \epsilon$ |

### 1.2

> 已知文法:

$$
\begin{align}
G(S): S &\to (A)|bS|b \\
A &\to A;S|S
\end{align}
$$

> 1. 消除该文法的左递归, 提取左公因子
> 2. 给出改造后文法的 `每一个非终结符` 的 `First` 和 `Follow` 集合
> 3. 给出改造后文法的 `预测分析表`

T1:

$$
\begin{align}
G(S): S &\to (A) | bS' \\
S' &\to S | \epsilon \\
A &\to SA' \\
A' &\to ;SA' | \epsilon
\end{align}
$$

T2:

| $v$  |       $First(v)$       |     $Follow(v)$      |
| :--: | :--------------------: | :------------------: |
| $S$  |      $\set{(, b}$      | $\set{\sharp, ;, )}$ |
| $S'$ | $\set{(, b, \epsilon}$ | $\set{\sharp, ;, )}$ |
| $A$  |      $\set{(, b}$      |      $\set{)}$       |
| $A'$ |  $\set{;, \epsilon}$   |      $\set{)}$       |

T3:

|      |     $($     |     $b$     |        $)$        |        $;$        |     $\sharp$      |
| :--: | :---------: | :---------: | :---------------: | :---------------: | :---------------: |
| $S$  | $S \to (A)$ | $S \to bS'$ |                   |                   |                   |
| $S'$ | $S' \to S$  | $S' \to S$  | $S' \to \epsilon$ | $S' \to \epsilon$ | $S' \to \epsilon$ |
| $A$  | $A \to SA'$ | $A \to SA'$ |                   |                   |                   |
| $A'$ |             |             | $A' \to \epsilon$ |   $A' \to ;SA'$   |                   |

### 1.3 (P81-T2)

#### 写在前面

这个题目出的有问题, 原来的题目中 $T \to +FT'$ 这个地方很明显是印错了, 出题人想表达的意思应该是 $T \to FT'$

因为这个东西本质上是只包含 $+, *, \wedge$ 操作的算术表达式

如果按照原题, 这个文法是有二义性的, 细节为:

$$
\begin{align}
T' &\to T | \epsilon \\
First(T') &= \set{+, \epsilon} \\
Follow(T') &= \set{+, \sharp, )} \\
First(T') \cap Follow(T') &= \set{+} \ne \Phi
\end{align}
$$

> 针对下面给定的文法 $G$:

$$
\begin{align}
G(E): E &\to TE' \\
E' &\to +E | \epsilon \\
T &\to FT' \\
T' &\to T | \epsilon \\
F &\to PF' \\
F' &\to *F' | \epsilon \\
P &\to (E) | a | b | \wedge
\end{align}
$$

> 请作答:
>
> 1. 给出文法中, 每个非终结符的 $First$ 和 $Follow$ 集合
> 2. 证明这个文法, 是 $LL(1)$ 的
> 3. 给出该文法的预测分析表

$T_1$

首先, 题目给出的文法, 不存在左递归结构, 也不存在左公共因子

所以, 直接下手:

| $v$  |            $First(v)$             |      $Follow(v)$       |
| :--: | :-------------------------------: | :--------------------: |
| $E$  |      $\set{(, a, b, \wedge}$      |   $\set{\sharp, )}$    |
| $E'$ |        $\set{+, \epsilon}$        |   $\set{\sharp, )}$    |
| $T$  |      $\set{(, a, b, \wedge}$      |  $\set{+, \sharp, )}$  |
| $T'$ | $\set{\epsilon, (, a, b, \wedge}$ |  $\set{+, \sharp, )}$  |
| $F$  |      $\set{(, a, b, \wedge}$      |  $\set{+, \sharp, )}$  |
| $F'$ |        $\set{*, \epsilon}$        |  $\set{+, \sharp, )}$  |
| $P$  |      $\set{(, a, b, \wedge}$      | $\set{*, +, \sharp,)}$ |

$T_2$

直接卡定义, 一个一个检验:

#### Condition1: 文法不含左递归

这个是显而易见的, 题目给出的文法, 不存在左递归结构

#### Condition2: $ \forall A \in V_N (A \to \alpha_1 | \alpha_2 | \dots | \alpha_n) \to First(\alpha_i) \cap First(\alpha_j) = \Phi ~ (i \ne j) $

$$
\begin{align}
(First(+E) := \set{+}) \cap (First(\epsilon) := \set{\epsilon}) &= \Phi \\
(First(T) := \set{(, a, b, \wedge}) \cap(First(\epsilon) := \set{\epsilon}) &= \Phi \\
(First(*F') := \set{*}) \cap(First(\epsilon) := \set{\epsilon}) &= \Phi
\end{align}
$$

满足该条件

#### Condition3: $ \forall A \in V_N (\epsilon \in First(A)) \to First(A) \cap Follow(A) = \Phi $

分析前置条件, 可以得知, 只需要检查 $E', T', F'$ 三个非终结符是否匹配该规则即可

经检查, 完全符合

$T_3$

|      |    $($     |       $)$       |    $a$     |    $b$     |   $\wedge$   |       $+$       |    $*$     |    $\sharp$     |
| :--: | :--------: | :-------------: | :--------: | :--------: | :----------: | :-------------: | :--------: | :-------------: |
| $E$  | $E\to TE'$ |                 | $E\to TE'$ | $E\to TE'$ |  $E\to TE'$  |                 |            |                 |
| $E'$ |            | $E'\to\epsilon$ |            |            |              |   $E'\to +E$    |            | $E'\to\epsilon$ |
| $T$  | $T\to FT'$ |                 | $T\to FT'$ | $T\to FT'$ |  $T\to FT'$  |                 |            |                 |
| $T'$ | $T'\to T$  | $T'\to\epsilon$ | $T'\to T$  | $T'\to T$  |  $T'\to T$   | $T'\to\epsilon$ |            | $T'\to\epsilon$ |
| $F$  | $F\to PF'$ |                 | $F\to PF'$ | $F\to PF'$ |  $F\to PF'$  |                 |            |                 |
| $F'$ |            | $F'\to\epsilon$ |            |            |              | $F'\to\epsilon$ | $F'\to*F'$ | $F'\to\epsilon$ |
| $P$  | $P\to(E)$  |                 |  $P\to a$  |  $P\to b$  | $P\to\wedge$ |                 |            |                 |

### 1.4 (P81-T4)

> 对下面的文法:

$$
\begin{align}
G(\text{Expr}): \text{Expr} &\to -\text{Expr} \\
\text{Expr} &\to (\text{Expr}) | \text{Var} ~ \text{ExprTail} \\
\text{ExprTail} &\to -\text{Expr} | \epsilon \\
\text{Var} &\to id ~ \text{VarTail} \\
\text{VarTail} &\to (Expr) | \epsilon
\end{align}
$$

> 请作答:
>
> 1. 给出文法中, 每个非终结符的 $First$ 和 $Follow$ 集合
> 2. 给出该文法的预测分析表

$T_1$

先简单改写一下文法形式:

$$
\begin{align}
G(\text{Expr}):
\text{Expr} &\to -\text{Expr} | (\text{Expr}) | \text{Var} ~ \text{ExprTail} \\
\text{ExprTail} &\to -\text{Expr} | \epsilon \\
\text{Var} &\to id ~ \text{VarTail} \\
\text{VarTail} &\to (\text{Expr}) | \epsilon
\end{align}
$$

|        $v$        |     $First(v)$      |     $Follow(v)$      |
| :---------------: | :-----------------: | :------------------: |
|   $\text{Expr}$   |  $\set{-, (, id}$   |  $\set{\sharp, )}$   |
| $\text{ExprTail}$ | $\set{-, \epsilon}$ |  $\set{\sharp, )}$   |
|   $\text{Var}$    |     $\set{id}$      | $\set{-, \sharp, )}$ |
| $\text{VarTail}$  | $\set{(, \epsilon}$ | $\set{-, \sharp, )}$ |

$T_2$

水到渠成:

|                   |                $-$                 |                $($                 |              $)$               |                      $id$                      |            $\sharp$            |
| :---------------: | :--------------------------------: | :--------------------------------: | :----------------------------: | :--------------------------------------------: | :----------------------------: |
|   $\text{Expr}$   |   $\text{Expr} \to -\text{Expr}$   |  $\text{Expr} \to (\text{Expr})$   |                                | $\text{Expr} \to \text{Var} ~ \text{ExprTail}$ |                                |
| $\text{ExprTail}$ | $\text{ExprTail} \to -\text{Expr}$ |                                    | $\text{ExprTail} \to \epsilon$ |                                                | $\text{ExprTail} \to \epsilon$ |
|   $\text{Var}$    |                                    |                                    |                                |      $\text{Var} \to id ~ \text{VarTail}$      |                                |
| $\text{VarTail}$  |   $\text{VarTail} \to \epsilon$    | $\text{VarTail} \to (\text{Expr})$ | $\text{VarTail} \to \epsilon$  |                                                | $\text{VarTail} \to \epsilon$  |
