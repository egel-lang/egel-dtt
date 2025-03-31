# Dependent Type Theory in Egel

A one-file literate script implementing Martin-Löf dependent type theory
in Egel.

# An implementation of Martin-Löf dependent type theory in Egel

In the following script you'll find an interactive prover for a toy 
implementation of Martin-Löf dependent type theory and interactive
prover. It closely follows an 
[OCaml tutorial](https://math.andrej.com/2012/11/08/how-to-implement-dependent-type-theory-i/)
by Andrej Bauer 

## A minimal type theory

We are going to implement:

- a hierarchy of universes `Type_0`, `Type_1`, ..., 
- dependent products `Π x:A . B`,
- functions `λ x: A . e`, and
- application `e_0 e_1`.

## Egel preamble

```
    import "prelude.eg"
    using System, List
```

## Syntax

Starting off, a number of data constructors for the abstract syntax 
tree.

```
    data t_var, t_univ, t_pi, t_lambda, t_app
```

For the concrete syntax we choose:

- universes are written `Type 0`, `Type 1`, `Type 2`, ...
- the dependent product is written `forall x : A . B`,
- a function is written `fun x : A => B`,
- application is juxtaposition `e0 e1`.

If `x` does not appear freely in `B`, then we write `A -> B` instead 
of `forall x : A . B`.

## Lexer

We make use of the internal lexer of Egel but discard all whitespace 
and token information.

```
    def lexer = 
        do System::tokenize ""
        |> filter [(_,"whitespace",_) -> false | _ -> true]
        |> map [((_,R,C),_,T) -> ((R,C),T)]
```

## Tests
```
    def main = OS::read_line OS::stdin |> lexer
```

