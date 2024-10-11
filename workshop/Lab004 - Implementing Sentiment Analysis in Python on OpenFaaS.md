---
date: "Friday, October 11th 2024, 1:58:33 pm"
modified: "Friday, October 11th 2024, 3:55:37 pm"
---

## Implementing a Function in Python

This is the final part of the workshop. We want to go through the steps needed to make something reasonable in Python. Still a demo, but with all the aspects that you might need in the module.

Perhaps you want to make a new folder now and switch to that. This will keep your code organized.

``` sh
$ mkdir ../lab004 && cd ../lab004
```

### Scaffold the empty function

We have seen this already. Create the empty function with the `faas-cli`. Remember the correct prefix.

``` sh
faas-cli new --lang python3 sentiment --prefix=registry.kokishin.de/shoehn
```

As a developer, I just can't help but deploy the thing for testing 😇

> Pro-tip: if you rename your YAML file to `stack.yml` then you need not pass the `-f` flag to any of the commands.
> Let's follow that convention from now on.

``` sh
$ faas-cli up 

$ echo -n "Fribourg is great, it's always sunny there." | faas-cli invoke sentiment
Fribourg is great, it's always sunny there.
```

As you see, I already added a valid input for the final function.

What will we need for the final implementation of the function?

- We will need a library called TextBlob
- We will need a function to deploy everything

The template provides us with the structure to use an external library in python. We just need to add it to the `requirements.txt` file.

    textblob

Then we can use that library in the `handler.py` file. Let's try it with a minimal example.

``` python
from textblob import TextBlob

def handle(req):
    """handle a request to the function
    Args:
        req (str): request body
    """

    text = """
The titular threat of The Blob has always struck me as the ultimate movie
monster.
"""
    blob = TextBlob(text)
    return req
```

Let's deploy and test it out.

``` sh
$ faas-cli up 
Deploying: sentiment.

Unexpected status: 500, message: error deleting container sentiment, sentiment, cannot delete running task sentiment: failed precondition

Function 'sentiment' failed to deploy with status code: 500
```

Oops, you need to delete the old one first, then re-deploy.

``` sh
$ faas-cli remove 
Deleting: sentiment.
Removing old function.
```

``` sh
$ echo -n "Fribourg is great, it's always sunny there." | faas-cli invoke sentiment
Fribourg is great, it's always sunny there.
```

Still boring, but we could load and use an external library for our function. Then we will finish that handler now.

``` python
import json
from textblob import TextBlob
import nltk

MIN_CORPORA = [
    "brown",  # Required for FastNPExtractor
    "punkt",  # Required for WordTokenizer
    "punkt_tab",
    "wordnet",  # Required for lemmatization
    "averaged_perceptron_tagger",  # Required for NLTKTagger
]

for each in MIN_CORPORA:
    nltk.download(each, quiet=True)
        
def handle(req):
    """handle a request to the function
    Args:
        req (str): request body
    """

    ## Add the request body into a TextBlob
    blob = TextBlob(req)

    res = {
        "polarity": 0,
        "subjectivity": 0
    }

    for sentence in blob.sentences:
        res["subjectivity"] = res["subjectivity"] + sentence.sentiment.subjectivity
        res["polarity"] = res["polarity"] + sentence.sentiment.polarity

    total = len(blob.sentences)
    res["sentence_count"] = total
    res["polarity"] = res["polarity"] / total
    res["subjectivity"] = res["subjectivity"] / total

    return json.dumps(res)
```

Now we can re-deploy and have a look at our final function for today.
