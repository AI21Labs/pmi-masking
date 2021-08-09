# PMI-Masking
This repository includes the list of masked spans (the masking vocabulary) that is used in the [ICLR 2021 spotlight](https://iclr.cc/virtual/2021/spotlight/3496) PMI-Masking [paper](https://openreview.net/forum?id=3Aoft6NWFej), overviewed in this [blogpost](https://www.ai21.com/blog/pmi-masking). Below,

* Section 1 provides the list construction details. 
* Section 2 details the PMI-Masking method for bidirectional masked language models, which essentially treats all of the spans in the provided lists as units for masking. 
* Section 3 provides some scores of recently trained models on the [SQuAD2.0](https://rajpurkar.github.io/SQuAD-explorer/) and [RACE](https://www.cs.cmu.edu/~glai1/data/race/) benchmarks.

## Section 1: How we constructed the masking vocabulary

Given a pretraining corpus, we use its n-gram co-occurrence statistics to compile our masking vocabulary. We consider word n-grams of lengths 2–5 having over 10 occurrences in the corpus, and include the highest ranking collocations in the corpus, as measured via our proposed PMI_n measure (see equation below). Noticing that the PMI_n measure is sensitive to the length of the n-gram, we assemble per-length rankings for each n ∈ {2, 3, 4, 5}, and integrate these rankings to compose the masking vocabulary according to the following policy: 50% of the assembled list  are the top ranking bigrams, 25%  are the top ranking trigrams, and 12.5% are the top ranking 4-grams and 5-grams. After conducting a preliminary evaluation of how an n-gram’s quality as a collocation degrades with its PMI_n rank (detailed in the appendix of the paper), we chose the masking vocabulary size to be 800K, for which approximately half of pretraining corpus tokens were identified as part of some correlated n-gram

The PMI-Masking lists included in this repo are attained by using the following ranking score:

<img src="https://render.githubusercontent.com/render/math?math=\textrm{PMI}_n(w_1\ldots w_n)=\min_{\sigma\in\textrm{seg}(w_1\ldots w_n)}\log\frac{p^2(w_1\ldots w_n)}{\prod_{s\in\sigma}p(s)}">



This is in fact a modification of the original PMI_n score in equation 3 of the [paper](https://openreview.net/forum?id=3Aoft6NWFej) (that led to slight improvements) -- it multiplies in the frequency of the n-gram: p(w_1\ldots w_n). Intuitively, our method makes even more impact when more frequent n-grams are favored, since the signal introduced by the method is more prevalent.  

## Section 2: How to implement PMI masking given a masking vocabulary

The composed masking vocabulary is added to this repository. Given such a vocabulary, we treat its entries as units to be masked together. All input tokens not identified with entries from the masking vocabulary are treated independently as units for masking according to the Whole-Word Masking scheme of [Devlin et al](https://github.com/google-research/bert). If one masking vocabulary entry contains another entry in a given input, we treat the larger one as the unit for masking, e.g., if the masking vocabulary contains the n-grams “the united states”, “air force”, and “the united states airforce”, the latter will be one unit for masking when it appears. In the case of overlapping entries, we choose one at random as a unit for masking and treat the remaining tokens as independent units, e.g., if the input text contains “by the way out” and the masking vocabulary contains the n-grams“by the way” and “the way out”, we can choose either “by the way” and “out” or “by” and “the way out” as units for masking. After we segment the sequence of input tokens into units for masking, we then choose tokens for masking by sampling units uniformly at random until 15% of the tokens (the standard tokens of the 30K-token vocabulary) in the input are selected. As in the prior methods, replacement with [MASK](80%), random (10%), or original (10%) tokens is done at the unit level.

## Section 3: Results for the included lists

The PMI-Masking lists included in this repo are named after the corpus over which they were constructed:
* "wiki-bc" = Wikipedia + BookCorpus
* "owt-wiki-bc" = OpenWebText + Wikipedia + BookCorpus.  

The results of a BERT-Base sized model, attained after training for 1M steps on the respective training corpora and evaluating as detailed in the [paper](https://openreview.net/forum?id=3Aoft6NWFej) are:
|              | SQuAD2.0 (F1) | RACE |
| :---         |     :---:      |          :---: |
| wiki-bc      |   81.7	| 69.3   |
| owt-wiki-bc  |      82.5	 |  71.2    |
