# 十分钟魔法练习：λ演算

### By 「玩火」

> 前置技能：Java基础，ADT

## Intro

程序员们总是为哪种语言更好而争论不休，而强悍的大佬也为自己造出语言而感到高兴。造语言也被称为程序员的三大浪漫之一。这样一项看上去高难度的活动总是让萌新望而生畏，接下来我要介绍一种世界上最简单的**图灵完备**语言并给出60行Java代码的解释器实现。让萌新也能体验造语言的乐趣。

## λ演算

1936年，丘奇(Alonzo Church)提出了一种非常简单的计算模型，叫λ演算(Lambda Calculus)。

> 一些不严谨的通俗理解：
>
> λ表达式中的函数定义 `(λ x. E)` 就是定义了数学上的函数 `f(x)=E` ，只不过没有名字， `λ` 代表一个函数定义的开始，而 `.` 左边的是函数的自变量，可以是任意符号，这里用了 `x` ， `.` 的右边是函数的内容 `E` ，可以是任意 λ 表达式。
>
> 而函数应用 `F X` 就是对于一个数学上的函数 `F` 求值 `F(X)` ， `F` 就是函数， `X` 就是参数。比如 `(λ x. x)` 就是 `f(x)=x` ，比如 `(λ x. (x x))` 可以表示为 `f(x)=x(x)` ，其中 `x` 应当是个函数，不过这在数学里面是不允许的，而 `((λ x. (x x)) y)` 就可以表示为数学上的 `f(x)=x(x), f(y)` 也就是 `y(y)` 。
>
> 和传统数学函数最不一样的是λ演算里面的函数可以在任何位置被定义并且没有名字，并且可以被当作变量传递也可以作为函数的计算结果。

一个λ表达式有三种组成可能：变量 `x` 、函数定义 `(λ x. E)` 、函数应用 `(F X)` 。其中 `x` 是一个抽象的符号， `E, F, X` 是 λ 表达式。注意这是递归的定义，我们可以通过组合三种形式来构造复杂的 λ 表达式。比如 `((λ x. (x x)) y)` 整体是一个函数应用，其 `F` 是函数定义 `(λ x. (x x))` ， `X` 是 `y` ，而 `(λ x. (x x))` 函数定义的 `x` 是变量 `x` ， `E` 是 `(x x)` 。

λ表达式的计算也称为归约 (reduce) ，只需要将函数应用整体变换，变换结果为其作为函数定义的第一项 `F` (也就是 `(λ x. E)` ) 中 `E` 里出现的所有**自由**的 `x` 替换为其第二项 `X` ，也就是说 `((λ x. E) X)` 会被归约为 `E(x -> X)` ，。听上去挺复杂，举个最简单的例子 `((λ x. (x x) y)` 可以归约为 `(y y)` 。我这里提到了自由的 `x` ，意思是说它不是任何λ函数定义的自变量，比如 `(λ x. (x t))` 中的 `x` 就是不自由的， `t` 就是自由的。

函数定义有比函数应用更低的优先级，也就是说是 `(λ x. (x x))` 可以写成 `(λ x. x x)` 。函数应用是左结合的，所以 `((x x) x)` 可以写成 `(x x x)` 。

## 解释器

首先，我们要用 ADT 定义出 λ 表达式的数据结构：

```java
interface Expr {};
// Value 变量
class Val implements Expr {
    String x;
    Val(String s) {
        x = s;
    }
    public String toString() {
        return x;
    }
}
// Function 函数定义
class Fun implements Expr {
    String x;
    Expr e;
    Fun(String s, Expr a) {
        x = s;
        e = a;
    }
    public String toString() {
        return "(λ " + x + ". " 
            + e.toString() + ")";
    }
}
// Apply 函数应用
class App implements Expr {
    Expr f, x;
    App(Expr e1, Expr e2) {
        f = e1;
        x = e2;
    }
    public String toString() {
        return "(" + f.toString() 
            + " " + x.toString() + ")";
    }
}
```

然后就可以构造 λ 表达式了，比如 `(λ x. x (λ x. x)) y` 就可以这样构造：

```java
Expr expr = new App(
    new Fun("x", 
        new App(new Val("x"), 
            new Fun("x", new Val("x")))), 
    new Val("y"))
```

然后就可以定义归约函数 reduce 和应用自由变量函数 apply ：

```java
interface Expr {
    Expr reduce();
    Expr apply(String s, Expr ex);
}

class Val implements Expr {
    // ...
    public Expr reduce() { return this; }
    public Expr apply(String s, Expr ex) {
        if (s.equals(x)) return ex;
        return this;
    }
}

class Fun implements Expr {
    // ...
    public Expr reduce() { return this; }
    public Expr apply(String s, Expr ex) {
        if (s.equals(x)) return this;
        return new Fun(x, e.apply(s, ex));
    }
}

class App implements Expr {
    // ...
    public Expr reduce() {
        Expr fr = f.reduce();
        if (fr instanceof Fun) {
            Fun fun = (Fun) fr;
            return fun.e.apply(
                fun.x, x).reduce();
        }
        return new App(fr, x);
    }
    public Expr apply(String s, Expr ex) {
        return new App(f.apply(s, ex),
                       x.apply(s, ex));
    }
}

```

以上就是 60 行 Java 写成的解释器啦！

