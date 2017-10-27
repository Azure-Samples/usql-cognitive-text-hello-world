---
services: data-lake-analytics
platforms: dotnet
author: saveenr
---

# U-SQL/Cognitive Text Hello World

```
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

// Extract key phrases for each paragraph

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

@keyphrase =
    SELECT No,
            Year,
            Book,
            Chapter,
            Text,
            SqlArray.Create(KeyPhrase.Split(';')) AS KeyPhraseArray  
    FROM @keyphrase;

@split_keyphrases =
    SELECT No,
            Year,
            Book,
            Chapter,
            Text,
            KeyPhrase  
    FROM @keyphrase
        CROSS APPLY EXPLODE (KeyPhraseArray) AS r(KeyPhrase);

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

OUTPUT @split_keyphrases 
    TO "/split_keyphrases.csv"
    USING Outputters.Csv();

OUTPUT @sentiment 
    TO "/sentiment.csv"
    USING Outputters.Csv();

```
