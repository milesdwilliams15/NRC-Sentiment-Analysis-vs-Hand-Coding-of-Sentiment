# NRC-Sentiment-Analysis-vs-Hand-Coding-of-Sentiment

## Can Human Understanding of Text Be Replaced by a Computer?
The question asked in the very title of this section's heading gets to the heart of an important debate. I doubt the below example will offer a deffinitive resolution to this debate, but I hope it will prove insightful, at the very least.

Using a dataset of open-ended survey responses regarding customer satisfaction at an undisclosed business, I'll show below that, sometimes, a human is a much better judge of the positve vs negative sentiment in a body of text.

## The Method
One very simple way to do a sentiment analysis of a corpus of text is to use the get_nrc_sentiment() function in the syuzhet package in R.

    library(syuzhet)
    df$text<-as.character(df$text)

    # Do sentiment analysis using the NRC Emotion Lexicon:
    dfSentiment <- get_nrc_sentiment(df$text)

    # Combine with Data Frame:
    df <- cbind(df, dfSentiment)

The above will allow for the estimation of sentiment (positive and negative) as well as eight emotions identified via the NRC Emotion Lexicon for a vector of character strings where each element in the set is a unique document (or in this case, a survey response). The get_nrc_sentiment() function produces a data frame with 10 columns of numeric vectors (sentiment and emotion scores) of equal length to the data frame containing the vector of textual data. These data frames can therefore be combined into a single data frame so that the sentiment and emotions identified for each observation can be easily matched to the corresponding text.

It is also possible, and useful, to use the positive and negative sentiment scores to create a vector of sentiment valence, and then use this variable to compute a sentiment valence index (where scores closer to 1 are more positive and scores closer to 0 are more negative).

    # Compute the Sentiment Valence:
    df$valence <- df$positive-df$negative

    # Create a Sentiment Valence Index:
    df$sentimentIndex <- (-(min(df$valence)) + df$valence)/(-min(df$valence) + max(df$valence))

Now that we have our sentiment valence index, we also need to do some drudgery: we need to go through each survey response and, by hand, categorize it as one of three things: positive, neutral (meaning either lacking in obvious positive or negative sentiment, or having a balanced mix of the two), or negative. Thankfully, the dataset I'm using only has 164, relatively short, survey responses to go through, making this a doable task for a single person (myself). I've given this variable the name "sentiment" in my dataframe.

## How Do Things Compare?
The below boxplot shows a definite correlation between the sentiment valence index and our categorical, hand coded, variable of human identified sentiment.

    ggplot(gailey,aes(sentiment,sentimentIndex)) + 
      geom_boxplot(aes(fill=sentiment)) + theme_classic() +
      geom_jitter(width = .2) + theme(legend.position = "none") +
      ggtitle("Hand Coded Sentiment vs. Sentiment Valence Index",
              subtitle = "NRC Emotion Lexicon Used to Identify Positive and Negative Sentiment Scores\nUsed to Compute the Sentiment Valence Index") +
      theme(text=element_text(family="serif")) +
      xlab("Hand Coded Sentiment") + ylab("Sentiment Valence Index\n<-- More Negative ... More Positive -->") +
      theme(plot.title=element_text(face="bold",size=14)) +
      theme(plot.subtitle=element_text(size=10, hjust=0, face="italic", color="black")) +
      theme(panel.grid.major.y=element_line(colour="grey50",linetype=3))

![hand coded vs computer sentiment](https://cloud.githubusercontent.com/assets/23504082/22128313/7e5e2a40-de65-11e6-8cee-53131af17103.jpeg)

## Conclusion
So it seems the dictionary method has some correlation with human identified sentiment; however, there is a great deal of variation as well. After an overview of the data, I've concluded a great deal of disagreement results from some of the survey responses containing positive words ("good") that are negated ("was not"). The negation of positive terms would obviously indicate a negative comment, but the ability to catch this important contextual clue is a skill a computer relying on a lexicon of certain terms simply cannot grasp. 

This is not to say that computer-based sentiment analysis should be rejected (the dictionary method did have some success); rather, this example should show the necessity of checking the quality of such sentiment analysis by comparing the results to human judgment. Obviously doing so for a very large dataset would be highly impractical, but such a quality check could be done by taking a subset (preferably a random sample) of the data and putting that subset under the microscope.
