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

- a hierarchy of universes written `Type 0`, `Type 1`, `Type 2`, ...
- the dependent product `forall x : A . B`,
- a function `fun x : A => B`,
- application is juxtaposition `e0 e1`.

If `x` does not appear freely in `B`, then we write `A -> B` instead 
of `forall x : A . B`.

## Egel preamble

```
    import "prelude.eg"
    using System, List
```

## Syntax

Starting off, a number of data constructors for the abstract syntax 
tree.

```
    data t_univ, t_pi, t_lambda, t_app
```

## Lexer

We make use of the internal lexer of Egel but discard all whitespace 
and token information.

```
    def lexer = 
        do System::tokenize ""
        |> filter [(_,"whitespace",_) -> false | _ -> true]
        |> map [((_,R,C),_,T) -> ((R,C),T)]
```

## Parser

```
    import "search.eg"
    using Search

    def look = [{} -> fail {} | {X|XX} -> success X XX]
    def token = [S -> look <*> \T -> let T = snd T in if T == S then success T else fail]
    def match = [S -> look <*> \T -> let T = snd T in if Regex::match (Regex::compile S) T 
                    then success T else fail]

    def parse_var = match "[a-zA-Z]+[0-9]*"

    def parse_universe =
        token "Type" <*> \_ -> match "[0-9+]" <*> \N -> success (t_univ (to_int N))

    def parse_forall =
        token "forall"
            <*> \_ -> parse_var <*> \X ->  token ":"
            <*> \_ -> parse_term <*> \A -> token "." <*> \_ -> parse_term
            <*> \B -> success (t_pi X A B)

    def parse_fun =
        token "fun"
            <*> \_ -> parse_var <*> \X ->  token ":"
            <*> \_ -> parse_term <*> \A -> token "=>" <*> \_ -> parse_term
            <*> \B -> success (t_lambda X A B)

    def parse_primary =
        parse_universe <+> parse_forall <+> parse_fun <+> parse_var
        <+> (token "(" <*> \_ -> parse_primary <*> \E -> token ")" <*> \_ -> success E)

    def parse_primary_opt =
        [E0 -> parse_primary </> \E1 -> parse_primary_opt (t_app E0 E1)]

    def parse_term =
        parse_primary </> \E0 -> parse_primary_opt E0

    def parse =
        search parse_term [X _ -> X] [X _ -> throw X] [X _ -> throw X]
        
```

## Tests
```
    def main = OS::read_line OS::stdin |> lexer |> [T -> print T "\n"; T] |> parse
```

