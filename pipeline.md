# Pipeline

| Component | Choice |
| :--- | :--- |
| Hosting | [AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/) |
| Server Environment | [Node.js](https://nodejs.org/en/) |
| Server Framework | [Express.js](https://expressjs.com/) |
| Database | [MongoDB](https://www.mongodb.com/) |
| Front End Javascript | [Angular](https://angular.io/) |
| News Document Source | [News API](https://newsapi.org/) |
| Document Processing and Similarity Measurement | [spaCy](https://spacy.io/) |



# Putting It Together

We'll host on an AWS Elastic Beanstalk instance. We can start on the free tier and see how it goes, but will probably have to pay a little bit of money if we want this to be reasonable quick.

Server will be Express running on Node.js with mongoose to integrate with MongoDB.

We will have to establish a limited set of news sources which we will draw results from, as well as some kind of bias table so that we can classify these sources according to their bias.

Every hour, we query News API via the python requests package to receive the current top articles from our set of sources. We extract the meaningful text from each article using python-goose, then analyze its sentiment index using spaCy. We store this data in our database with a schema like:

```
{title: String, source: String, url: String; imageUrl: String, sentiment: Number}
```

When a user submits an article, we first check to see if it is in our results database. If not, we will extract and analyze the sentiment of its main text as discussed above. If the article is from one of our core sources, then we check its text for similarity with all the current top articles from sources with sufficiently different biases and return appropriately similar articles from each. If the article is not from one of our main sources, then we check for similarity with all current articles from all sources with sufficiently different sentiment. From these, we select and return the few which are most similar to the input article using spaCy. We then store the input article and its information in our database with the same schema as before, but this time adding an array of our previously created schema, which will serve as the results for this input.



