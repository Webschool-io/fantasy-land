# Fantasy Land Especificação

[![Se junte ao chat em https://gitter.im/fantasyland/fantasy-land](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/fantasyland/fantasy-land?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

(aka "Algebraic JavaScript Specification")

![](logo.png)

Este projeto especifica interoperabilidade das estruturas algébricas comum:

* [Setoid](#setoid)
* [Semigroup](#semigroup)
* [Monoid](#monoid)
* [Functor](#functor)
* [Apply](#apply)
* [Applicative](#applicative)
* [Foldable](#foldable)
* [Traversable](#traversable)
* [Chain](#chain)
* [Monad](#monad)
* [Extend](#extend)
* [Comonad](#comonad)

![](figures/dependencies.png)

## Geral

Uma álgebra é um conjunto de valores, um conjunto de operadores que está fechado
e sob algumas leis que devem obedecer.

Cada álgebra Fantasy Land é uma especificação separada. Uma álgebra pode
ter dependências de outras álgebras que devem ser executadas. Uma
álgebra também pode indicar outros métodos de álgebra que não precisam de ser
implementados e como eles podem ser derivados a partir de novos métodos.

## Terminologia

1. "value" é qualquer valor JavaScript, incluindo a que tem as estruturas definidas abaixo.
2. "equivalent" é uma definição adequada de equivalência para o valor dado.
     A definição deve garantir que os dois valores podem ser trocados de forma segura em um programa que respeite abstrações. Por exemplo:
    - Duas listas são equivalentes se elas são equivalentes em todos os índices.
    - Dois objetos JavaScript antigos simples, interpretadas como dicionários, são equivalentes quando eles são equivalentes para todas as chaves.
    - Duas promessas são equivalentes quando deu valores equivalentes.
    - Duas funções são equivalentes se conduzirem a resultados equivalentes para as entradas equivalentes.

## Algebras

### Setoid

1. `a.equals(a) === true` (reflexividade)
2. `a.equals(b) === b.equals(a)` (simetria)
3. If `a.equals(b)` and `b.equals(c)`, then `a.equals(c)` (transitividade)

#### método `equals` 

Um valor que tem uma Setoid deve fornecer um método `equals`. O
`método equals` recebe um argumento:

    a.equals(b)

1. `b` deve ser um valor da mesma Setoid

    1. Se `B` não é o mesmo Setoid, comportamento de `equals` é não especificado (recomendado voltar `false`).

2. `equals` deve retornar um boolean (`ou` true` false`).

### Semigroup

1. `a.concat(b).concat(c)` é equivalente a `a.concat(b.concat(c))` (associatividade)

#### método `concat`

Um valor que tem uma Semigroup deve fornecer um método `concat`. O
`método concat` recebe um argumento:

    s.concat(b)

1. `b` deve ser um valor da mesma Semigroup

    1. Se `B` não é o mesmo semigroup, comportamento de 'é concat` não especificado.

2. `concat` deve retornar um valor do mesmo Semigroup.

### Monoid

Um valor que implementa a especificação Monoid também deve implementar
a especificação Semigroup.

1. `m.concat(m.empty())` is equivalent to `m` (right identity)
2. `m.empty().concat(m)` is equivalent to `m` (left identity)

#### método `empty`

Um valor que tem uma Monoid deve fornecer um método `empty` sobre si mesma ou seu objeto `constructor`. O método `empty` não tem argumentos:

    m.empty()
    m.constructor.empty()

1. `empty` deve retornar um valor do mesmo Monoid

### Functor

1. `u.map(function(a) { return a; })` é equivalente a `u` (identidade)
2. `u.map(function(x) { return f(g(x)); })` é equivalente a `u.map(g).map(f)` (composição)

#### método `map`

Um valor que tem uma Functor deve fornecer um método `map`. O `map`
método tem um argumento:

    u.map(f)

1. `f` deve ser uma função,

    1.Se `f` não é uma função, o comportamento do  `map` é não especificado.
    2. `f` pode retornar qualquer valor.

2. `map` deve retornar um valor do mesmo Functor

### Apply

Um valor que implementa a especificação Apply deve também implementar a especificação Functor.

1. `a.map(function(f) { return function(g) { return function(x) { return f(g(x))}; }; }).ap(u).ap(v)` is equivalent to `a.ap(u.ap(v))` (composition)

#### método `ap`

Um valor que tem uma Apply deve fornecer um método `ap`. O `ap`
método recebe um argumento:

    a.ap(b)

1. `a` deve ser uma função de um Aplicar,

    1. Se `a` não representa uma função, o comportamento do` ap` não é especificado.

2. `b` deve ser um Apply de qualquer valor

3. `ap` deve aplicar-se a função em Apply `a` ao valor em Aplicar` B`

### Aplicação

Um valor que implementa a especificação Applicative deve também
implementar a especificação Apply.

A value which satisfies the specification of an Applicative does not
need to implement:

* Functor's `map`; derivable as `function(f) { return this.of(f).ap(this); }`

1. `a.of(function(x) { return x; }).ap(v)` is equivalent to `v` (identity)
2. `a.of(f).ap(a.of(x))` is equivalent to `a.of(f(x))` (homomorphism)
3. `u.ap(a.of(y))` is equivalent to `a.of(function(f) { return f(y); }).ap(u)` (interchange)

#### método `of`

A value which has an Applicative must provide an `of` method on itself
or its `constructor` object. The `of` method takes one argument:

    a.of(b)
    a.constructor.of(b)

1. `of` must provide a value of the same Applicative

    1. No parts of `b` should be checked

### Foldable

1. `u.reduce` is equivalent to `u.toArray().reduce`

* `toArray`; derivable as `function() { return this.reduce(function(acc, x) { return acc.concat(x); }, []); }`

#### método `reduce`

A value which has a Foldable must provide a `reduce` method. The `reduce`
method takes two arguments:

    u.reduce(f, x)

1. `f` must be a binary function

    1. if `f` is not a function, the behaviour of `reduce` is unspecified.
    2. The first argument to `f` must be the same type as `x`.
    3. `f` must return a value of the same type as `x`

1. `x` is the initial accumulator value for the reduction

### Traversable

A value that implements the Traversable specification must also
implement the Functor specification.

1. `t(u.sequence(f.of))` is equivalent to `u.map(t).sequence(g.of)`
where `t` is a natural transformation from `f` to `g` (naturality)

2. `u.map(function(x){ return Id(x); }).sequence(Id.of)` is equivalent to `Id.of` (identity)

3. `u.map(Compose).sequence(Compose.of)` is equivalent to
   `Compose(u.sequence(f.of).map(function(x) { return x.sequence(g.of); }))` (composition)

* `traverse`; derivable as `function(f, of) { return this.map(f).sequence(of); }`

#### método `sequence`

A value which has a Traversable must provide a `sequence` method. The `sequence`
method takes one argument:

    u.sequence(of) 

1. `of` must return the Applicative that `u` contains.

### Chain

A value that implements the Chain specification must also
implement the Apply specification.

A value which satisfies the specification of a Chain does not
need to implement:

* Apply's `ap`; derivable as `function ap(m) { return this.chain(function(f) { return m.map(f); }); }`

1. `m.chain(f).chain(g)` is equivalent to `m.chain(function(x) { return f(x).chain(g); })` (associativity)

#### método `chain`

A value which has a Chain must provide a `chain` method. The `chain`
method takes one argument:

    m.chain(f)

1. `f` must be a function which returns a value

    1. If `f` is not a function, the behaviour of `chain` is
       unspecified.
    2. `f` must return a value of the same Chain

2. `chain` must return a value of the same Chain

### Monad

A value that implements the Monad specification must also implement
the Applicative and Chain specifications.

A value which satisfies the specification of a Monad does not need to
implement:

* Apply's `ap`; derivable as `function(m) { return this.chain(function(f) { return m.map(f); }); }`
* Functor's `map`; derivable as `function(f) { var m = this; return m.chain(function(a) { return m.of(f(a)); })}`

1. `m.of(a).chain(f)` is equivalent to `f(a)` (left identity)
2. `m.chain(m.of)` is equivalent to `m` (right identity)

### Extend

1. `w.extend(g).extend(f)`
   is equivalent to 
   `w.extend( function(_w){ return f( _w.extend(g) ); } )`

#### método `extend`

An Extend must provide an `extend` method. The `extend`
method takes one argument:
     
     w.extend(f)

1. `f` must be a function which returns a value

    1. If `f` is not a function, the behaviour of `extend` is
       unspecified.
    2. `f` must return a value of type `v`, for some variable `v` contained in `w`.

2. `extend` must return a value of the same Extend.

### Comonad

A value that implements the Comonad specification must also implement the Functor and Extend specifications.

1. `w.extend(function(_w){ return _w.extract(); })` is equivalent to `w`
2. `w.extend(f).extract()` is equivalent to `f(w)`
3. `w.extend(f)` is equivalent to `w.extend(function(x) { return x; }).map(f)`

#### método`extract`

A value which has a Comonad must provide an `extract` method on itself. 
The `extract` method takes no arguments:
    
    c.extract()

1. `extract` must return a value of type `v`, for some variable `v` contained in `w`.
    1. `v` must have the same type that `f` returns in `extend`.








## Notes

1. If there's more than a single way to implement the methods and
   laws, the implementation should choose one and provide wrappers for
   other uses.
2. It's discouraged to overload the specified methods. It can easily
   result in broken and buggy behaviour.
3. It is recommended to throw an exception on unspecified behaviour.
4. An `Id` container which implements all methods is provided in
   `id.js`.
