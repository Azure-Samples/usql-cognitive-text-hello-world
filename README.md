---
services: data-lake-analytics
platforms: dotnet
author: saveenr
---

# USQL/Cognitive Text Hello World


### Input data

```
REFERENCE ASSEMBLY [TextCommon];
REFERENCE ASSEMBLY [TextSentiment];
REFERENCE ASSEMBLY [TextKeyPhrase];

@WarAndPeace =
    EXTRACT No int,
            Year string,
            Book string,
            Chapter string,
            Text string
    FROM @"/usqlext/samples/cognition/war_and_peace.csv"
    USING Extractors.Csv();
```

### Extract key phrases for each paragraph

```
@keyphrase =
    PROCESS @WarAndPeace
    PRODUCE No,
            Year,
            Book,
            Chapter,
            Text,
            KeyPhrase string
    READONLY No,
            Year,
            Book,
            Chapter,
            Text
    USING new Cognition.Text.KeyPhraseExtractor();

// Tokenize the key phrases.
@kpsplits =
    SELECT No,
        Year,
        Book,
        Chapter,
        Text,
        T.KeyPhrase
    FROM @keyphrase
        CROSS APPLY
            new Cognition.Text.Splitter("KeyPhrase") AS T(KeyPhrase);
```


### Perform sentiment analysis on each paragraph

```
@sentiment =
    PROCESS @WarAndPeace
    PRODUCE No,
            Year,
            Book,
            Chapter,
            Text,
            Sentiment string,
            Conf double
    READONLY No,
            Year,
            Book,
            Chapter,
            Text
    USING new Cognition.Text.SentimentAnalyzer(true);
```
