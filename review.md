# Review

## 1. 记得不太清楚的知识点

1. 短语: 遍历所有子根, 每次推导到底即可得到一个短语
2. 直接短语: 子根的所有子节点, 必须均为叶子节点, 这样每次推导到底即可得到一个直接短语
3. 句柄: 最左直接短语
4. 素短语: 某个短语, 自身至少含有一个非终结符, 且该短语不可被拆分为更短的 `至少含有一个非终结符` 的短语, 则为素短语

## 2. 各种控制流语句的翻译手法

### $ S \to if~E~then~S_1 $

- 两遍

$$
  \begin{align*}
      E.true &:= \text{newlabel} \\
      E.false &:= S.next \\
      S_1.next &:= S.next \\
      S.code &:= \begin{align*}
        E.code ~||~ gen(E.true ~ :) ~||~ S_1.code
      \end{align*}
  \end{align*}
$$

- 一遍

$$
  \begin{align*}
  S &\to if~E~then~M~S_1 \\
  &\{ \\
  &backpatch(E.truelist, M.quad); \\
  &S.nextlist = merge(E.falselist, S_1.nextlist) \\
  &\} \\
  M &\to \epsilon ~ \{ M.quad = nextquad \}
  \end{align*}
$$

### $ S \to if~E~then~S_1~else~S_2 $

- 两遍

$$
  \begin{align*}
      E.true &:= \text{newlabel} \\
      E.false &:= \text{newlabel} \\
      S_1.next &:= S.next \\
      S_2.next &:= S.next \\
      S.code &:= E.code ~||~ \\
      &gen(E.true~:) ~||~ S_1.code ~||~ gen(goto~S_1.next) ~||~ \\
      &gen(E.false~:) ~||~ S_2.code ~||~ gen(goto~S_2.next) \\
  \end{align*}
$$

- 一遍

$$
  \begin{align*}
  S &\to if~E~then~M_1~S_1~N~else~M_2~S_2 \\
  &\{ \\
    &backpatch(E.truelist, M_1.quad); \\
    &backpatch(E.falselist, M_2.quad); \\
    &S.nextlist = merge(S_1.nextlist, N.nextlist, S_2.nextlist) \\
  &\} \\
  M &\to \epsilon ~ \{ M.quad = nextquad \} \\
  N &\to \epsilon \\
  &\{ \\
    &N.quad = nextquad \\
    &emit(j, -, -, -) \\
  &\} \\
  \end{align*}
$$

### $ S \to while~E~do~S_1 $

- 两遍

$$
  \begin{align*}
      S.begin &:= \text{newlabel} \\
      E.true &:= \text{newlabel} \\
      E.false &:= S.next \\
      S_1.next &:= S.begin \\
      S.code &:= gen(S.begin~:) ~||~ E.code ~||~ \\
        &gen(E.true~:) ~||~ S_1.code ~||~ gen(goto~S.begin)
  \end{align*}
$$

- 一遍

$$
  \begin{align*}
  S &\to while~M_1~E~do~M_2~S_1 \\
  &\{ \\
    &backpatch(E.truelist, M_2.quad); \\
    &backpatch(S_1.nextlist, M_1.quad); \\
    &S.nextlist = E.falselist; \\
    &emit(j, -, -, M_1.quad) \\
  &\} \\
  M &\to \epsilon ~ \{ M.quad = nextquad \} \\
  \end{align*}
$$

### $ S \to do~S_1~while~E $

- 两遍

$$
  \begin{align*}
      S.begin &= newlabel \\
      E.begin &= newlabel \\
      E.true &= newlabel \\
      S_1.next &= E.begin \\
      S.code &= gen(S.begin~:) ~||~ S_1.code ~||~ gen(goto~E.begin) ~||~ \\
        &gen(E.begin~:) ~||~ E.code ~||~ \\
        &gen(E.true~:) ~||~ gen(goto~S.begin)
  \end{align*}
$$

- 一遍

$$
  \begin{align*}
  S &\to do~M_1~S_1~while~M_2~E \\
  &\{ \\
    &backpatch(S_1.nextlist, M_2.quad); \\
    &backpatch(E.truelist, M_1.quad); \\
    &S.nextlist = E.falselist; \\
    &emit(j, -, -, M_1.quad) \\
  &\} \\
  M &\to \epsilon ~ \{ M.quad = nextquad \} \\
  \end{align*}
$$

### $ S \to repeat~until~E~do~S_1 $

- 两遍

$$
  \begin{align*}
      S.begin &:= \text{newlabel} \\
      E.false &:= \text{newlabel} \\
      E.true &:= S.next \\
      S_1.next &:= S.begin \\
      S.code &:= gen(S.begin~:) ~||~ E.code ~||~ \\
        &gen(E.false~:) ~||~ S_1.code ~||~ gen(goto~S.begin)
  \end{align*}
$$

- 一遍

$$
  \begin{align*}
  S &\to repeat~until~M_1~E~do~M_2~S_1 \\
  &\{ \\
    &backpatch(E.falselist, M_2.quad); \\
    &backpatch(S_1.nextlist, M_1.quad); \\
    &S.nextlist = E.truelist; \\
    &emit(j, -, -, M_1.quad) \\
  &\} \\
  M &\to \epsilon ~ \{ M.quad = nextquad \} \\
  \end{align*}
$$

### $ S \to repeat~S_1~until~E $

- 两遍

$$
  \begin{align*}
      S.begin &= newlabel \\
      E.begin &= newlabel \\
      E.false &= newlabel \\
      S_1.next &= E.begin \\
      S.code &= gen(S.begin~:) ~||~ S_1.code ~||~ gen(goto~E.begin) ~||~ \\
        &gen(E.begin~:) ~||~ E.code ~||~ \\
        &gen(E.false~:) ~||~ gen(goto~S.begin)
  \end{align*}
$$

- 一遍

$$
  \begin{align*}
  S &\to repeat~M_1~S_1~while~M_2~E \\
  &\{ \\
    &backpatch(S_1.nextlist, M_2.quad); \\
    &backpatch(E.falselist, M_1.quad); \\
    &S.nextlist = E.truelist; \\
    &emit(j, -, -, M_1.quad) \\
  &\} \\
  M &\to \epsilon ~ \{ M.quad = nextquad \} \\
  \end{align*}
$$

### $ S \to for(E_1;E_2;E_3)~S_1 $

- 两遍

$$
\begin{align*}
S_1.begin &= newlabel \\
E_1.begin,~E_2.begin,~E_3.begin &= newlabel,~... \\
E_2.true &= newlabel \\
E_2.false &= S.next \\
S_1.next &= E_3.begin \\
S.code &= gen(E_1.begin~:) ~||~ E_1.code ~||~ \\
  &gen(E_2.begin~:) ~||~ E_2.code ~||~ \\
  &gen(E_2.true~:) ~||~ gen(goto~S_1.begin) ~||~ \\
  &gen(E_3.begin~:) ~||~ E_3.code ~||~ gen(goto~E_2.begin) ~||~ \\
  &gen(S_1.begin~:) ~||~ S_1.code ~||~ gen(goto~E_3.begin)
\end{align*}
$$

- 一遍

$$
\begin{align*}
S &\to for(M_1~E_1;M_2~E_2;M_3~E_3~N)~M_4~S_1 \\
&\{ \\
  &backpatch(E_2.truelist, M_4.quad); \\
  &backpatch(S_1.nextlist, M_3.quad); \\
  &S.nextlist = E_2.falselist; \\
  &emit(j, -, -, M_3.quad) \\
&\} \\
M &\to \epsilon ~ \{ M.quad = nextquad \} \\
N &\to \epsilon ~ \{ emit(j, -, -, M_2.quad) \} \\
\end{align*}
$$
