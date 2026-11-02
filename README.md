# Comparing Bigram, LSTM, and Transformer Language Models

## Task

I solved the task of predicting the next character using three different models: bigram, LSTM, and Transformer.

## Dataset

Public-domain book available at gutenberg.org/ebooks/11 (Plain Text UTF-8, saved as `alice.txt`). `alice.txt` should be saved in the same folder as the notebook. Dataset was ≈ 145,000 characters, 75 unique characters, 90/10 train/validation split.

Note: removed the Project Gutenberg license text at the start and end of the text file and the [Illustration] placeholders.

## Models

### Bigram

The Bigram model uses only the character that immediately precedes the character that is predicted. The context window is brief: it ignores all the earlier characters and only predicts based on previously learned frequencies. Because of the limited window, this method does not build many whole words as evidenced by the output “I f thul... ta... warme e... be her”.

What it can do well is learn common letter pairs and some short words (be, her) and parts of words (warme).

### LSTM

The LSTM model uses a block of 16 characters as a context window. This allows the model to see more, 16 times as many previous characters as the bigram model window, so it can retain a memory of up to several previous whole words. Predictably, the output from this model contains a higher number of whole words and parts of words (I, ding, a, could, go, was, she, last, round, not, Alice). LSTM struggles with longer dependencies because it passes its memory along one character at a time, so the character at position one tends to have less influence on the next predicted character than the character at position 16, indicating a recency bias in favor of the character nearest the prediction.

### Transformer

The transformer used token embeddings, which added the information for how a character tends to be used (vowels tend to be used in a similar way, so their vectors tend to be similar).
And it also used positional embeddings, which added where the character sat within the 16-character window.

The self-attention in the transformer model meant that the earlier characters contributed directly to the next-character prediction while the subsequent characters were masked. The LSTM’s earlier characters tended to be diluted in their contribution to the predicted character, while the transformer tended to preserve their contribution.

The feedforward layer of the transformer processed each character's vector on its own, which contained the context obtained by attention from any prior characters. It found patterns in each vector and then the output layer created a raw score for each of the 75 possible next-character predictions.

## Results

| Model | Train Loss | Validation Loss | Training Time (s) | Generated Text Quality | Notes |
|---|---|---|---|---|---|
| Bigram | 2.4392 | 2.4460 | 1.46 | Low | Fastest; mostly nonsense |
| LSTM | 1.8824 | 1.9340 | 2.78 | Medium | More real words, including "Alice" |
| Transformer | 1.9198 | 1.9309 | 3.52 | Medium | Slowest; closest to the book's dialogue style |

Text quality ratings: Low = mostly nonsense; Medium = many real words and some dialogue structure; High = mostly real words and readable phrases.

All three of the models used the same settings: 1,000 steps, batch size 32, 16-character context window, and learning rate 0.01 with losses averaged over 50 batches. The bigram model resulted in some short words, although it was the fastest. The LSTM model resulted in longer words and fewer nonsense character strings. The Transformer model resulted in the longest string of recognizable words. All the models began to pick up the dialogue structure of the book to differing degrees.

## Generated Text Samples

### Bigram

```
“I f thul s. ?” Hasicehes by she dily d s Do s sero fitho med I’Ohet isht ithierid y he swang id ise as, Dundf he
e be waronane. tory _,
ta ganghangey t Ofabheretiers warme e
be her atherdourend y Saupeasks, f atulishbe
o tGjd All shea tthecisw r

mpls at
An t.

athe we ighe bo M(Whe w gad in: am w
```

### LSTM

```
“Ye. I
thinp chored ding the Mock it seemak haden a could knith Doleged she ouble go was I erauster ous she last as Forgerpeen the
beglat she
simomione Soo she turtto
round the deatse, of not I howhouurton in joss Alice,” seatterple his a lang she ittlicters habice wercark?_ Ceverplowers, wrean eef
```

### Transformer

```
* _fidelo, “Whe twast ture.

“Ohuted
obe be, nepeall,” the I thate nothe like beeaging lipse hersased hat cring as so ggan baute use of the ly dionging ”replied that off theret is aniwe the cle, “But it que fout,” Alice and tlitle there alayenn that it and try hinglespitiong, and it ons in
```

## Analysis

The Transformer model performed best. Despite the negligible difference in validation loss between LSTM and Transformer, the quality of the text output was somewhat better than LSTM's output. The cost of that improved output sample was the longer training time.

The results table showed that the bigram model’s validation loss was 2.446, which decreased to approximately 1.934 and 1.931 for the LSTM and Transformer models. The bigram model was the fastest to train at 1.46 seconds, LSTM was 2.78 seconds, and the Transformer was 3.52 seconds. The Transformer was the slowest; it took 27% longer than LSTM, but did not show a measurable difference in validation loss, 1.934 compared with 1.931. In this case, the corpus was a short book, so the absolute difference in time was not significant, but the additional time needed to train the Transformer could not justify the negligible improvement in loss (≈0.16 %).

In these samples, the bigram model has a few short words and parts of words, he, be, all, and warme, but most of the output is nonsense (“I f, tGjd, tthecisw, atherdoured, waronane., tory). The LSTM model has more short words, some longer words, and word parts (“Ye., ding, the, Mock, could, she, go, was, last, round the, of, not, Alice). This model even returned the main character’s name, Alice. The Transformer model returned more short words, longer words, and some common word parts (the, thate, nothe, like, lipse, hat, as, so, use of the ly, “replied that off theret, is, “But it, Alice and, there, that it and try, and it ons in). There are longer strings with discernible short words. Although the difference in loss was negligible, the generated output was noticeably improved in these samples. All three models began to pick up the characteristic dialogue style from the small corpus, although the Transformer model followed it most closely, producing the dialogue tag `”replied`.

## What Attention Improved

The Transformer model improved the number and length of recognizable words in the 300-character sample. The sample also had more runs of consecutive words. This is because attention looks at all earlier characters directly and weights them. Unlike the LSTM, where earlier characters’ information can fade, attention keeps each earlier character’s contribution available, which likely led to longer strings of discernible words. Training for these models occurred on a small corpus, so the negligible difference in validation loss between the LSTM and the Transformer models may not hold with a larger corpus or model.

## Reflection

First, I was surprised to see how much bigram was able to do with such a simple model, a small corpus, in such a short amount of time. While it was not the best performing model, that is still impressive. It began to pick up the dialogue structure in the short book, and it did that in under 1.5 seconds.

Next, the LSTM model was able to resolve even more whole words than bigram. Although the words were short, it returned more words and fewer nonsense strings than bigram.

Although there was only one sample taken for comparison from each model, the difference in the sample from LSTM to transformer was noticeable. There were 15 correctly spaced characters in a row that also returned whole words (that it and try). Even though the transformer model was the slowest, it returned more words with limited training. We should consider that loss barely changed from LSTM to transformer, but the text returned was noticeably improved after attention was added.

I think that attention gave the transformer model direct access to all 16 earlier characters; that powered the improvement in the generated text from LSTM, which is why it returned a higher number of longer words.

## AI Use

See [`ai_use_summary.md`](ai_use_summary.md) for details on how I used Claude.
