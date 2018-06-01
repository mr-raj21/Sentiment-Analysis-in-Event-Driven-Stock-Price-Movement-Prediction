# Sentiment Analysis for Event-Driven Stock Prediction
Use natural-language processing (NLP) to predict stock price movement based on Reuters News

1. Data Collection and Preprocessing

    1.1 get the whole ticker list to obtain the details of public companies

    1.2 crawl news from Reuters using BeautifulSoup
    
    1.3 crawl prices using urllib

2. Feature Engineering (Tokenization)
  
    2.1 Unify word format: unify tense, singular & plural, remove punctuations & stop words
  
    2.2 Implement one-hot encoding
  
    2.3 Pad word senquence (essentially a matrix) to keep the same dimension
  
3. Train a set of Bayesian Convolutional Neural Networks using Stochastic Gradient Langevin Dynamics to obtain more robustness
4. Use thinning models to predict future news

## Requirement
* Python 3
* [PyTorch > 0.4](https://pytorch.org/)
* numpy
* [NLTK](https://www.nltk.org/install.html)


## Usage

### 1. Data collection


#### 1.1 Download the ticker list from [NASDAQ](http://www.nasdaq.com/screening/companies-by-industry.aspx)

```bash
$ ./crawler/all_tickers.py 20  # keep the top e.g. 20% marketcap companies
```

#### 1.2 Use BeautifulSoup to crawl news headlines from [Reuters](http://www.reuters.com/finance/stocks/overview?symbol=FB.O)

*Note: you may need over one month to fetch the news you want.*

Suppose we find a piece of news about COO Qi Lu Resignation on May.18, 2018 at reuters.com

![](./imgs/baidu.PNG)

We can use the following script to crawl it and format it to our local file

```bash
$ ./crawler/reuters.py # we can relate the news with company and date, this is more precise than Bloomberg News
```

![](./imgs/111.png)

By brute-force iterating company tickers and dates, we can get the dataset with roughly 400,000 news in the end. Since a company may have multiple news in a single day, the current version will only use topStory news to train our models and ignore the others.

#### 1.3 Use urllib to crawl historical stock prices
 
Improvement here, use normalized return [5] over S&P 500 instead of return.

```bash
$ ./crawler/yahoo_finance.py # generate raw data: stockPrices_raw.json, containing open, close, ..., adjClose
$ ./create_label.py # use raw price data to generate stockReturns.json
```

### 2. Feature engineering (Tokenization)

Unify the word format, project word to a word vector, so every sentence results in a matrix.

Detail about unifying word format are: lower case, remove punctuation, get rid of stop words, unify tense and singular & plural.

```bash
$ ./tokenize_news.py
```

### 3. Train a Bayesian ConvNet to predict the stock price movement. 

Type the following to train a set of robust Bayesian models.
```bash
$ ./main.py -epochs 500 -static False
```

### 4. Prediction and analysis

```bash
$ ./main.py -predict "Top executive behind Baidu's artificial intelligence drive steps aside"
>>> Sell
```

Test the performance on the most recent news in two weeks.
```bash
$ ./main.py -eval True
>>> Testing    - loss: 67.6102  acc: 58.07%(41.8/72) 83.50%(3/3) 100.00%(0/0) 0.00%(0/0) 0.00%(0/0)
```
Note: the predictions are averaged (therefore some numbers, like 3/3, may be rounded to integers). From left to right, the predictions are more and more confident.


### 5. Future work

From the [work](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=1331573) by Tim Loughran and Bill McDonald, some words have strong indication of positive and negative effects in finance, we may need to dig into these words to find more information. A very simple but interest example can be found in [Financial Sentiment Analysis part1](http://francescopochetti.com/scrapying-around-web/), [part2](http://francescopochetti.com/financial-blogs-sentiment-analysis-part-crawling-web/)

As suggested by H Lee, we may consider to include features of earnings surprise due to its great value.

You are welcome to build a better stopword list and share it.


## Issues
1. remove_punctuation() handles middle name (e.g., P.F -> pf)

## References:

1. Yoon Kim, [Convolutional Neural Networks for Sentence Classification](http://www.aclweb.org/anthology/D14-1181), EMNLP, 2014
2. J Pennington, R Socher, CD Manning, [GloVe: Global Vectors for Word Representation](http://www-nlp.stanford.edu/pubs/glove.pdf), EMNLP, 2014
3. Max Welling, Yee Whye Teh, [Bayesian Learning via Stochastic Gradient Langevin Dynamics](https://pdfs.semanticscholar.org/aeed/631d6a84100b5e9a021ec1914095c66de415.pdf), ICML, 2011
4. Tim Loughran and Bill McDonald, 2011, “When is a Liability not a Liability?  Textual Analysis, Dictionaries, and 10-Ks,” Journal of Finance, 66:1, 35-65.
5. H Lee, etc, [On the Importance of Text Analysis for Stock Price Prediction](http://nlp.stanford.edu/pubs/lrec2014-stock.pdf), LREC, 2014
6. Xiao Ding, [Deep Learning for Event-Driven Stock Prediction](http://ijcai.org/Proceedings/15/Papers/329.pdf), IJCAI2015
7. [IMPLEMENTING A CNN FOR TEXT CLASSIFICATION IN TENSORFLOW](http://www.wildml.com/2015/12/implementing-a-cnn-for-text-classification-in-tensorflow/)
8. [Keras predict sentiment-movie-reviews using deep learning](http://machinelearningmastery.com/predict-sentiment-movie-reviews-using-deep-learning/)
9. [Keras sequence-classification-lstm-recurrent-neural-networks](http://machinelearningmastery.com/sequence-classification-lstm-recurrent-neural-networks-python-keras/)
10. [tf-idf + t-sne](https://github.com/lazyprogrammer/machine_learning_examples/blob/master/nlp_class2/tfidf_tsne.py)
11. [Implementation of CNN in sequence classification](https://github.com/dennybritz/cnn-text-classification-tf)
12. [Getting Started with Word2Vec and GloVe in Python](http://textminingonline.com/getting-started-with-word2vec-and-glove-in-python)
