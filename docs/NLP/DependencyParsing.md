# Dependency Parsing

## Linguistic structure

-   Phrase structure grammar
    -   := context-free grammars
    -   unit := vertex of the tree
    -   ```
        NP -> Det N
            | Det Adj N
        ```
-   Dependency structure

    -   ![](img/2020-12-14-08-40-01.png)
        -   `[ ]` := sub unit
            -   either `{[ ]}` or `[] {}`, not allow `[ { ] }`?
        -   if a node has at least a child, the node itself could not be separated alone as a unit.
        -   valid units:
            -   `in kitchen`
                -   position
            -   `the kitchen`
                -   which kitchen
            -   `in the kitchen`
                -   position + which
            -   `in crate`
                -   position
            -   `the crate`
                -   which one
            -   `large crate`
                -   size
            -   `crate in the kitchen`
            -   `crate by the door`

-   Human communicate complex ideas by including words in structure.
-   Aim: `unit --{what connnects}-- other unit`.

## Ambiguity

### Prepositional phrase attachment Ambiguity

Prepositional phrase could be attached to different tokens.

![](img/2020-12-14-09-17-25.png)

```
                        kill
                 (subj)/    \(obj)   \ (nmod)
                      /      \         \
                   cops       man      knife
            /     /                  /
          /      /                  /
       San    Jose                 with
```

... cops using a knife to kill a man.

vs.

```
                        kill
                       /    \
                      /      \
                   cops       man
            /     /                 \ (nmod)
          /      /                    \
       San    Jose                    knife
                                     /
                                    /
                                   with
```

... cops kill the man who was holding a knife.

![](img/2020-12-14-10-09-30.png)

-   exponential choices -> a lot of ambiguities

### Coordination scope ambiguity

Tokens of both sides around `and`, `or` or `,` want to govern the whole subtree.

![](img/2020-12-14-10-14-38.png)

```
Shuttle veteran and longtime NASA executive Fred Gregory appointed to board
```

```
                                                         appointed
                                                   /                  \
                                            Fred Gregory              board
             /                                                       /
        veteran                                                    to
       /                         \
Shuttle                           executive
                   /              /
                and          NASA
                           /
                    longtime
```

```
                                                         appointed
              /                                                       \
        veteran                                                       board
       /                                     \                       /
Shuttle                                     Fred Gregory           to
                   /                        /
                and               executive
                                 /
                             NASA
                            /
                    longtime
```

![](img/2020-12-14-12-08-52.png)

```
No heart, cognitive issues
```

```
                    issues
       /
   heart
  /      \
No        cognitive
         /
        ,
```

-   `,` := or.

```
   heart
  /                 \
No                  issues
         /         /
        , cognitive
```

-   `,` := and.

### Adjectival modifier ambiguity

Adjective attached to different tokens.

![](img/2020-12-14-12-36-11.png)

```
Students get first hand job experience
```

```
         get
        /                  \
Students                    experience
                       /   /
                   hand job
           (amod) /
             first
```

-   `amod` := adjective modifier

```
         get
        /                  \
Students                    experience
                  /        /
             first      job
                       /
                   hand
```

### Verb Phrase (VP) attachment ambiguity

![](img/2020-12-14-13-03-07.png)

```
Mutilated body washes up on Rio beach
```

-   `... to VP`.

## Dependency Grammar

![](img/2020-12-14-13-11-42.png)

-   Linear: `The results demonstrated that KaiC interacts rhythmically with SasA KaiA and KaiB`.
-   information units:
    -   `KaiC interacts SasA`
    -   `KaiC interacts SasA KaiA`
    -   `KaiC interacts SasA KaiB`
-   `nsubj` := nominal subject
-   `det` := determiner
-   `nmod` := nominal modifier
-   `ccomp` := clausal complement
-   `cc` := Coordinating Conjunction
-   `nn` := Common noun (singular noun)
-   `nns` := plural noun
-   `nnp` := Proper Noun
-   `jj` := adjective
-   `case` := "Case" is a linguistics term regarding a manner of categorizing nouns, pronouns, adjectives, participles, and numerals according to their traditionally corresponding grammatical functions within a given phrase, clause, or sentence.

![](img/2020-12-14-13-45-56.png)

-   `aux` := Auxiliary, a helping element, typically a verb, that adds meaning to the basic meaning of the main verb in a clause.

![](img/2020-12-14-13-58-37.png)

![](img/2020-12-14-14-01-15.png)

-   Universal Dependencies
    -   corpus
    -   <https://universaldependencies.org/>
    -   unify all grammar structures -> become standard.
    -   authorize on ambiguity

![](img/2020-12-14-14-11-49.png)

-   if non-projection is not allowed, modify:
    -   `I'll give a talk on bootstrapping tomorrow.`

## Parsing

### Transition based parsing

Arc-standard transition-based parser:

![](img/2020-12-14-14-14-49.png)

-   `A` := dependencies.
-   $\sigma$ := stack.
-   $\beta$ := buffer.

Parsing:

1. ![](img/2020-12-14-14-19-36.png)
    - dark frame := stack.
    - yellow frame := buffer.
    - only `[root]` is on the stack -> shift.
1. ![](img/2020-12-14-14-23-00.png)
    - check if `I` depends on `[root]` -> false -> shift.
1. ![](img/2020-12-14-14-29-40.png)
    - find dependency in stack that `stack[1] <- stack[2]` -> reduction -> Left-Arc.
1. ![](img/2020-12-14-14-24-02.png)
    - remove dependent from stack and add dependency to `A`.
    - could have had a reduction again saying `[root] -> ate`, but there is `fish` on the buffer that `ate -> fish` -> shift.
1. ![](img/2020-12-14-14-36-02.png)
    - find dependency in stack that `stack[1] -> stack[2]` -> reduction -> Right-Arc.
1. ![](img/2020-12-14-14-24-43.png)
    - remove dependent from stack and add dependency to `A`.
    - find dependency in stack that `stack[0] -> stack[1]` -> reduction -> Right-Arc.
1. ![](img/2020-12-14-14-25-02.png)
    - remove dependent from stack and add dependency to `A`.
    - only `[root]` on stack and empty buffer -> finish.

Con:

-   travel exponential size tree of possibilities. 
-   even dynamic programming reduce to $O(n^2)$.

### Machine learning based parser

![](img/2020-12-14-14-43-46.png)

-   machine learning classifier
    -   empirical.
    -   predicts the next step:
        -   shift
        -   left-arc
        -   right-arc
        -   ...
    -   high accuracy.
    -   no traversal at all.
    -   time complexity: $O(n)$.


