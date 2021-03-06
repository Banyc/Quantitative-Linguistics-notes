# Word Vectors

## Human Language & Words

-   Human vs orangutan

    -   Human language is the networking language in human computer network.
    -   Human team up
    -   writing -> make knowledge last long (accumulative?)

-   Human Language is slow

    -   -> compression information in communication by assuming subjects have mutual knowledge (common sense?)

-   Words
    -   := represent things
    -   How computer understands words? (relationships between words, meanings of words)
        -   use `WordNet`
            -   Synonym
        -   use `nltk`

## Words representations

### One-hot vectors

```python
motel = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0]
hotel = [0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0]
```

... those indexes represent respectively `house`, `cat`, `dog`, `some chairs`, `agreeable`, ...

Cons:

-   too many words -> too many indexes.
-   not showing the relationship between `motel` and `hotel`
    -   if showing the relationship by matrix -> matrix still too big

### By context

Distributional semantics: word's meaning <- mostly appear close-by

<pre>
...government debt problems turing into <b>banking</b> crises as happened in 2009...
    ...saying that Europe needs unified <b>banking</b> regulation to replace the hodgepodge...
            ...India has just given its <b>banking</b> system a shot in the arm...
</pre>

... the words around the `banking` is the meaning of `banking`.

### Word vectors

```python
banking = [0.286, 0.792, -0.177, -0.107, 0.109, -0.542, 0.349, 0.271]
```

-   dense vector
    -   not one-hot alike
-   distributed representation?
-   limited dimensions
    -   practically >= 50, namely 300, 1,000, 2,000, 4,000...
-   each word is a vector
-   all the words in a vector space
    -   ![](img/2020-12-12-15-27-52.png)
    -   different part (cluster) of the vector space has a pattern
        -   ![](img/2020-12-12-15-28-03.png)
        -   ![](img/2020-12-12-15-28-21.png)
        -   pattern: similarity
-   value of each dimension <- learned by algorithm. (black box)

## Word2vec

### Overview

To put all word vectors to the vector space in the proper location. ![](img/2020-12-12-16-12-16.png)

1.  Neural network: having `into` --{predict}--> `problems`, `turning`, `banking`, `crises`
1.  Change word vectors
1.  go to the next word ![](img/2020-12-12-16-15-27.png)
1.  loop

### Objective function

![](img/2020-12-12-16-24-21.png)

-   Likelihood

    -   Time complexity = `O((T) * (2 * m))`
    -   Intuitive:

        ```csharp
        int m;  // given window size
        int T;  // given size of corpus
        double theta;  // given all hyper variables θ to be optimized
        int t;  // index of center word
        double likelihood = 1;

        // foreach (var word in the_corpus)
        for (t = 0; t < T; t++)
        {
            var center_word = corpus[t];
            int j;
            // for each context word within the window
            for (j = -m; j < m; j++)
            {
                // ignore center word
                if (j == 0)
                {
                    continue;
                }

                var context_word = corpus[t + j];
                likelihood *= P(context_word, center_word, theta)
            }
        }
        ```

    -   `m` is usually from 5 to 10

-   objective function
    -   := a loss function
        -   aim to be minimized
    -   `-1` := the better prediction, the smaller loss
    -   $\cfrac{1}{T}$ := decouple with size of corpus
    -   `log()` always put on a product
    -   Con: need to sum up the entire vocabulary.
        -   solution: the skip-gram model with negative sampling:
            -   ![](img/2020-12-13-11-54-44.png)
                -   $\sigma$ := sigmoid function
                    -   ![](img/2020-12-13-11-59-45.png)
                    -   $\sigma (x) = \cfrac {1} {1 + e ^{-x}}$
            -   intuitive goal:
                1.  take all the words that have shown up and give them big probabilities (scores);
                1.  and take negative sampling some random words that have not shown up and give them as small probabilities (scores) as possible.
            -   `-1` in $-u_k^T v_c$: make random words with small probabilities (scores).
                -   it flips the result over axis y in sigmoid.
                -   Meaning `1 - probability`.
            -   `K`: could be 10, 15 negative samples.
            -   How to sample?
                -   ![](img/2020-12-13-12-10-17.png)
                -   `w` := every word in the entire vocabulary.
                -   `P(w)` := the probability of being picked as a negative sample word?
                    -   a probability distribution.
                -   `U(w)` sums up the count of the word for every word in the vocabulary.
                -   `Z`: the size of the entire vocabulary.
                    -   for normalization.
                    -   capital Z often means for normalization.

### Prediction function

![](img/2020-12-12-17-27-40.png)

![](img/2020-12-12-17-22-03.png)

-   $v_{w}$ := `w` is a **center word**
-   $u_{w}$ := `w` is a **context word**
-   `o` := a context word
-   `c` := a center word
-   `V` := entire vocabulary
    -   a symbol table
    -   a list having all words in the corpus
    -   has removed the replicas
-   `w` := every word in the entire vocabulary
-   The softmax function is a function that turns a vector of K real values into a vector of K real values that sum to 1

### Training model

![](img/2020-12-12-21-27-27.png)

-   shape of `θ` := `[2V, d]`
    -   in `θ` there are `2V` vectors
    -   each vector has `d` dimensions
-   `u` and `v` vectors are initialized randomly

Learn more from the backpropagation...

Gradient:

$$
\cfrac {\partial log(P(o | c))} {\partial v_c}
$$

$$
= u_o - \cfrac {\sum ^V _{x = 1} (e ^ {u ^T _o * v_c } * u_x)} {\sum ^V _{w = 1} e ^{u^T_w * v_c}}
$$

$$
= u_o - \sum ^V _{x = 1} (\cfrac {e ^ {u ^T _o * v_c }} {\sum ^V _{w = 1} e ^{u^T_w * v_c}} * u_x)
$$

$$
= u_o - \sum ^V _{x = 1} (P(x | c) * u_x)
$$

### Optimization

Stochastic Gradient Descent

-   Intuitive:

    ![](img/2020-12-13-01-27-19.png)

-   mini-batches
-   batch := a random window sample
-   Cons:
    -   some 32 batches have only hundreds distinct words but the entire vocabulary is way larger than that.
    -   ![](img/2020-12-13-01-35-02.png)
    -   solution
        -   assign selected rows to the slices of `theta_grad`? or `theta`?

### Details

The analogy could also be found in the word vectors

-   $v_{queen} \approx v_{king} - v_{man} + v_{woman}$.

Why two `u`, `v` vectors for the same word in $\theta$?

-   easy to calculate $\cfrac {\partial log(P(o | c))} {\partial v_c}$
-   since the dot product is symmetric, the two vectors are close to each other. later take the average of the two vectors

Two model variants (training logic):

-   Skip-grams (SG)
    -   predict context words (position independent) given center word.
-   Continuous Bag of Words (CBOW)
    -   predict center word from the bag of context words

... Skip-gram model is selected.

### Window-based co-occurrence matrix

-   suppose window size is 1 (normally 5-10)
-   ![](img/2020-12-13-13-49-07.png)
    -   symmetric
    -   raw count
-   Cons:
    -   matrix is too big. `shape = V * V`.
        -   Solution
            -   dense word vector
                -   shrink to 25-1000 dimensions like word2vec
                -   ![](img/2020-12-13-13-59-26.png)
                -   ![](img/2020-12-13-13-59-55.png)
                -   ![](img/2020-12-13-14-00-09.png)
                -   but never works very well...
            -   deal with high-frequency words when counting
                -   ways to modify the count
                    -   log scale the count
                        -   `log(X)`?
                    -   ceiling function to the count
                        -   `min(X, t)`, `t` ~= `100`
                    -   Use Pearson correlations instead of counts, then set negative values to 0
-   Pros:
    -   ![](img/2020-12-13-14-16-24.png)

### Combining count matrix and neural methods

![](img/2020-12-13-14-17-35.png)

Aim: meaning components <--{encode}-- ratios of co-occurrence probabilities.

![](img/2020-12-13-14-28-57.png)

-   $P(x | thing)$ := co-occurrence probability.
-   Count matrix embodies in the columns?
-   Neural methods embody in the function $P(x | thing)$?

![](img/2020-12-13-14-40-31.png)

![](img/2020-12-13-14-41-20.png)

-   $w_i^T w^{'}_j - logX_{i j}$ := difference.
-   $b_i$ and $b^{'}_j$ := biases.

### Evaluate word vectors

![](img/2020-12-13-14-48-29.png)

-   methods:
    -   intrinsic
        -   if guessing the right **part** of a speech?
        -   if putting synonyms close together?
    -   extrinsic
        -   evaluate in specific real applications.
        -   web search.
        -   question answering.
        -   phone dialog system.

![](img/2020-12-13-14-52-31.png)

![](img/2020-12-13-14-54-03.png)

![](img/2020-12-13-14-54-53.png)

![](img/2020-12-13-14-55-19.png)

![](img/2020-12-13-14-57-23.png)

![](img/2020-12-13-14-57-49.png)

![](img/2020-12-13-14-59-59.png)

![](img/2020-12-13-15-01-57.png)

-   wiki explains the connections and relations of concepts.

![](img/2020-12-13-15-04-28.png)

-   human column := human judge the similarity of the two words by scales from 0 to 10.
-   then check the distance between the word vectors in the space.

![](img/2020-12-13-15-44-26.png)

### Ambiguity of word senses

![](img/2020-12-13-15-16-39.png)

-   ![](img/2020-12-13-15-17-48.png)

![](img/2020-12-13-15-18-40.png)

-   For each common word, cluster all the contexts in which it occurs. To see if there are multiple clear clusters. If so, split the word into pseudo-words. Then change the words in the corpus.
-   separate senses -> pseudo-words
    -   $bank_1$
    -   $bank_2$
-   but different senses are also relative to each other. Different senses are derivative from the previous existing senses.

![](img/2020-12-13-15-21-20.png)

-   superposition := weighted average / weighted sum.
-   $v_{pike}$ := superposition.
-   $v_{pike_n}$ := components.
-   $a_n$ := frequency of $v_{pike_n}$.
-   $a_n v_{pike_n}$ := weighted component.
-   what benefit? Given superposition, find different component vectors.
    -   too many dimensions -> components/senses are sparse throughout the space -> easy to separate components out from a superposition.
    -   `tie` has five underlying meaning components. (the five columns)
