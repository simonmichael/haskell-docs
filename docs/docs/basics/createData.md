---
comments: true
---

## Products

You can make your own types like this:

```haskell
data Square = Sq Int Int -- (1)!
```

1. The choice of the names `Square` and `Sq` are both arbitrary. Both must be capitalized.

This creates a new type `Square`, where values of type `Square` look like `Sq i j`, where `i` and `j` are `Int`s:

```haskell title="repl example"
> :t Sq 5 4
Sq 5 4 :: Square
> :t Sq 5 True
"Couldn't match expected type ‘Int’ with actual type ‘Bool’"
> :t Sq 
Sq :: Int -> (Int -> Square) -- (1)!
> :t Sq 3
Sq 3 :: Int -> Square -- (2)!
```

1. Actually, the repl will drop the brackets, and show: `#!haskell Int -> Int -> Square`.

2. If this is unclear, see [here](/basics/functions/#currying) for more info.

??? Tip
    The type `Square` contains the same information as the product type `(Int, Int)`. These types are different, in the sense that code which expects one will not work with the other, but it is easy to write loss-less functions between them:

    ```haskell
    fromSq :: Square -> (Int, Int)
    fromSq (Sq i j) = (i, j)

    toSq :: (Int, Int) -> Square
    toSq (i, j) = Sq i j
    ```

!!! Note 
    The number of types following `Sq` can be 0 or more. For example:
    ```haskell title="repl example"
    > data SquareAlt = SqAlt Int
    > data SquareAlt2 = SqAlt2
    > :t SqAlt 
    ```

    If the number of types is `1`, you will see a suggestion to replace `data` with `newtype`. See more about `newtype` [here](https://kowainik.github.io/posts/haskell-mini-patterns)

### Records

You can also name entries:

```hs 
data Entity = Sq {row :: Int, col :: Int}
```

`row` and `col` are now accessing functions:

```hs title="repl example"
> entity = Sq 4 5
> :t entity
entity :: Entity
> row entity
4
> col entity
5
> :t row
row :: Entity -> Int
> :t col
col :: Entity -> Int
```

## Sums


```haskell
data Entity = Sq Int Int | Player Bool -- (1)!
```

1. `Entity` is a type*, but `Sq` and `Player` are values belonging to that type

The vertical bar `|` indicates that an `Entity` is *either* made with `Sq` *or* with `Piece`:

```haskell title="repl example"
> Sq 4 6
Sq 4 6 :: Entity
> :t Player False
Player False :: Entity
> :t Player 
Player :: Bool -> Entity
```

??? Note
    The type `Entity` contains the same information as the type `#!haskell Either (Int, Int) Bool`, and one could write functions between them:

    ```haskell
    fromEntity :: Entity -> Either (Int, Int) Bool
    fromEntity (Sq i j) = Left (i, j)
    fromEntity (Player bool) = Right bool

    toEntity :: Either (Int, Int) Bool -> Entity
    toEntity (Left (i ,j)) = Sq i j
    toEntity (Right bool) = Player bool
    ```




!!! Tip 

    You can combine products and sums, using your own types:

    ```haskell
    data ChessPiece = Piece PieceType Color | SquareType Square -- (1)!
    data Color = Black | White
    data PieceType = Bishop | Rook
    data Square = Sq Int Int
    ```

    1. Note that Haskell is unconcerned by the order of definitions: the definition of `Color` comes after its use in the definition of `ChessPiece`. 





## Parameterized types

One can also create types which take another type as a parameter:

```haskell
data Piece c = Bishop c | Knight c | King c
```

This creates `Piece Bool`, `Piece Int`, and so on:

```hs title="repl example"
> data Piece c = Bishop c | Knight c | King c
> :t Knight True
Knight True :: Piece Bool
let four = 4 :: Int
> :t King four
King four :: Piece Int
> :t King (King True)
King (King True) :: Piece (Piece Bool)
```

??? Note
    One can think of `Piece` as a *function on types*, which gives a type `c`, produces a type `Piece c`. 

    To make this idea explicit, one can say that `Piece` has *kind* `Type -> Type`, where *kind* is a name for the types that types themselves have. 

## Recursive types

```haskell
data BinTree = Leaf Int | Branch BinTree BinTree
```

Here, `BinTree` is being used *recursively* in its own definition. Values of `BinTree` include:

```haskell title="repl example"

> data BinTree = Leaf Int | Branch BinTree BinTree
> :t Leaf 4
Leaf 4 :: BinTree
> :t Leaf True
> :t Branch (Leaf 4) (Leaf 5)
Branch (Leaf 4) (Leaf 5) :: BinTree
> :t Branch (Leaf 3) ((Branch (Leaf 6) (Leaf 8)))
Branch (Leaf 3) ((Branch (Leaf 6) (Leaf 8))) :: BinTree
```

Recursive types can also be parametrized:

```hs
data BinTree a = Leaf a | Branch (BinTree a) (BinTree a)
```

```hs title="repl example"
data BinTree a = Leaf a | Branch (BinTree a) (BinTree a)
> :t Leaf True
Leaf True :: BinTree Bool
> :t Leaf ()
Leaf () :: BinTree ()
> :t Branch (Leaf True) (Leaf False)
Branch (Leaf True) (Leaf False) :: BinTree Bool
> :t Branch (Leaf True) (Leaf ())
"Couldn't match expected type ‘Bool’ with actual type ‘()’" -- (1)!
```

1. The definition of `Branch` requires that the left and right branch be trees *of the same type*, which is why this doesn't work.

Here is a more complex recursive type and a program of that type:

```hs
machine :: Machine Int Int
machine = machine1 where 
    
    machine1 :: Machine Int Int
    machine1 = M (\i -> (i, if i > 10 then machine2 else machine1))

    machine2 :: Machine Int Int
    machine2 = M (\i -> (0, machine2))
```

!!! Note
    The list type can be defined recursively in this way:

    ```hs
    data List a = EmptyList | HeadThenList a (List a)
    ```

    In fact, the `[a]` type in Haskell is defined in this way, with the `[1,2,3]` being extra syntax for convenience.


## Synonyms

One can also give new names to existing types:

```hs
type Number = Double
```

This can be useful for readability, particularly for quite complex types:

```hs
type Failure = Text
data Result = ...
type Program = Either Failure Result
```