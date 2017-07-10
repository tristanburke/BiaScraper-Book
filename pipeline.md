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

We'll host on an AWS Elastic Beanstalk instance. We can start on the free tier and see how it goes, but will probably have to pay a little bit of money if we want this to be reasonably quick.

Server will be Express running on Node.js with mongoose to integrate with MongoDB.

We will have to establish a limited set of news sources which we will draw results from, as well as some kind of bias table so that we can classify these sources according to their bias.

Every hour, we query News API via the python requests package to receive the current top articles from our set of sources. We extract the meaningful text from each article using python-goose, then analyze its sentiment index using spaCy. We store this data in our database with a schema like:

```
{title: String, source: String, url: String; imageUrl: String, sentiment: Number}
```

When a user submits an article, we first check to see if it is in our results database. If not, we will extract and analyze the sentiment of its main text as discussed above. If the article is from one of our core sources, then we check its text for similarity with all the current top articles from sources with sufficiently different biases and return appropriately similar articles from each. If the article is not from one of our main sources, then we check for similarity with all current articles from all sources with sufficiently different sentiment. From these, we select and return the few which are most similar to the input article using spaCy. We then store the input article and its information in our database with the same schema as before, but this time adding an array of our previously created schema, which will serve as the results for this input.



We begin with a selection of target news sources, and keep a database collection of them and each of their corresponding bias values. Each day, we update an article database with all of the articles from target companies received by queries to newspaper. User enters url to article. Form sends post request to "/".  We take article url, then pass it into python script \(check\_results.py\) checks to see if it is in a database of previously input urls. If there is an entry for this database, we extract the result urls and image urls and send that back to the server to display to the user. If there is no entry for this database, we use newspaper to find the company which published the article, along with the article's title and text. Then we pass this info into another python script \(running on another virtual environment enterpreter which supports spaCy\). In this script, we check the publishing company for the input url given by the previous script. If this company is in our target company database, we select a list of other target companies with differing biases. If this company is not in our target company database, we just match it against all target sources. Then, we iterate over each article in our article database and use spaCy to measure its semantic similarity with the input article. We return the top match for each target company, then create a results database entry

```
resultSchema = {"title": String, "brand": String, "url": String, "text": String, "results": [articleSchema]}
```

Then, we pass the input url into another python script \(find\_matches.py\). This script will use use newspaper to get the text of the input article

