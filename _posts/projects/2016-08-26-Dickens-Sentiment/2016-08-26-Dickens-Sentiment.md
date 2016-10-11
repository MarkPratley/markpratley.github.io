---
layout: page-fullwidth
title: "Charles Dickens - Sentiment Analysis"
#subheadline: "Sentiment Analysis - Charles Dickens"
meta_teaser: "A sentiment analysis of Oliver Twist by Charles Dickens"
teaser: "A sentiment analysis of Oliver Twist by Charles Dickens"
breadcrumb: true
comments: true
meta: true
header:
    title: ""
    image_fullwidth: profile.png
    background-color: "#262930"
    caption: Oliver Twist Positive/Negative Plot Analysis
image:
    thumb:  profile.png
    homepage: profile.png
    caption: Oliver Twist Positive/Negative Plot Analysis
categories:
    - projects
tags:
    - r
    - Sentiment Analysis
    - ggplot2
---

## Intro

This is a sentiment analysis of some of Charles Dickens's most famous works using the [nrc lexicon](http://saifmohammad.com/WebPages/lexicons.html) with texts downloaded from [Project Gutenberg](http://www.gutenberg.org/ebooks/search/?query=Charles+Dickens).

We start with a comparison of sentiment during the narrative of Oliver Twist, and will at some point add a comparison of the sentiment in different books.

There is quite a bit of r code so you could skip to the Oliver Twist Sentiment graph [here](#Graph_OT) if you'd prefer.

## Data Preparation


{% highlight r %}
library(tidyr)
library(zoo)
library(dplyr)
library(stringr)
library(reshape2)
library(ggplot2)
library(readr)
library(purrr)
library(tidytext)
library(syuzhet)
library(viridis)
{% endhighlight %}

## Read books into r


{% highlight r %}
# read books
rawText <- read_lines("books/AChristmasCarol.txt", skip = 165, n_max = 3321) # skip non-book bits
CC <- data.frame(book="A Christmas Carol", line=rawText)

rawText <- read_lines("books/BleakHouse.txt", skip = 213, n_max = 39660) # skip non-book bits
BH <- data.frame(book="Bleak House", line=rawText)

rawText <- read_lines("books/DavidCopperfield.txt", skip = 190, n_max = 38032) # skip non-book bits
DC <- data.frame(book="David Copperfield", line=rawText)

rawText <- read_lines("books/DombeyAndSon.txt", skip = 115, n_max = 39088) # skip non-book bits
DaS <- data.frame(book="Dombey and Son", line=rawText)

rawText <- read_lines("books/GreatExpectations.txt", skip = 43, n_max = 20013) # skip non-book bits
GE <- data.frame(book="Great Expectations", line=rawText)

rawText <- read_lines("books/HardTimes.txt", skip = 153, n_max = 11514) # skip non-book bits
HT <- data.frame(book="Hard Times", line=rawText)

rawText <- read_lines("books/LittleDorrit.txt", skip = 195, n_max = 36712) # skip non-book bits
LD <- data.frame(book="Little Dorrit", line=rawText)

rawText <- read_lines("books/MartinChuzzlewit.txt", skip = 182, n_max = 37298) # skip non-book bits
MC <- data.frame(book="Martin Chuzzlewit", line=rawText)

rawText <- read_lines("books/NicholasNickleby.txt", skip = 234, n_max = 36991) # skip non-book bits
NN <- data.frame(book="Nicholas Nickleby", line=rawText)

rawText <- read_lines("books/OliverTwist.txt", skip = 150, n_max = 18682) # skip non-book bits
OT <- data.frame(book="Oliver Twist", line=rawText)

rawText <- read_lines("books/OurMutualFriend.txt", skip = 145, n_max = 38419) # skip non-book bits
OMF <- data.frame(book="Our Mutual Friend", line=rawText)

rawText <- read_lines("books/TaleofTwoCities.txt", skip = 104, n_max = 15796) # skip non-book bits
ToTT <- data.frame(book="Tale of Two Cities", line=rawText)

rawText <- read_lines("books/TheMysteryOfEdwinDrood.txt", skip = 46, n_max = 11424) # skip non-book bits
ED <- data.frame(book="The Mystery of Edwin Drood", line=rawText)

rawText <- read_lines("books/TheOldCuriosityShop.txt", skip = 38, n_max = 23659) # skip non-book bits
OCS <- data.frame(book="The Old Curiosity Shop", line=rawText)

rawText <- read_lines("books/ThePickwickPapers.txt", skip = 283, n_max = 36437) # skip non-book bits
PP <- data.frame(book="The Pickwick Papers", line=rawText)

# Add all books to a single df - also coerces factor to character
df <- bind_rows(CC, BH, DC, DaS, GE, HT, LD, MC, NN, OT, OMF, ToTT, ED, OCS, PP)

# delete raw data
to.delete <- 0
to.delete <- ls()
to.delete <- to.delete[to.delete!="df"]
rm(list = to.delete)
{% endhighlight %}

## Process the data


{% highlight r %}
# delete empty rows
df <- 
    df %>% 
    filter(line!="")

# keep line number
df <- 
    df %>% 
    group_by(book) %>% 
    mutate(line.no=row_number()) %>% 
    ungroup()

# get group - groups of 10
df <- 
    df %>% 
    arrange(book, line.no)
df$group <- ((seq(1,nrow(df))-1) %/% 10) + 1

# split line into a list of words in temp d.f.
df.temp <- df$line %>% str_extract_all(boundary("word"))

# turn list of words into separate columns of words
df.temp <- plyr::ldply(df.temp, rbind)

# add to df with book, line.no etc
df <- cbind(df, df.temp)

# gather all columns of words (wide form) into rows of words (long form)
#  keeping book, line.no, line etc
df <- 
    df %>% 
    gather(key=word.pos, value=word, -c(book, line, line.no, group), na.rm = T) %>% 
    mutate(word.pos=as.integer(word.pos)) %>% 
    arrange(book, line.no, word.pos) %>% 
    group_by(book) %>% 
    mutate(word.pos=1:n())    # change word.pos to be the word order
{% endhighlight %}



{% highlight text %}
## Warning: attributes are not identical across measure variables; they will
## be dropped
{% endhighlight %}

## Remove Stop Words

Stop words are the relatively unimportant, often connecting, words with no sentiment value. Here we remove them to aid analysis.


{% highlight r %}
data("stop_words")
df <- 
    df %>%
    anti_join(stop_words) %>% 
    arrange(book, line.no, word.pos)
{% endhighlight %}



{% highlight text %}
## Joining, by = "word"
{% endhighlight %}

## Analyse Text


{% highlight r %}
# analyse sentiment
sentiment <- get_nrc_sentiment(df$word)

# add together
df <- bind_cols(df, sentiment)
{% endhighlight %}

Here we create a single sum value for positive/negative.


{% highlight r %}
# make negative negative
df$negative <- -df$negative

# get a pos-neg sum figure
df$p.n <- df$positive + df$negative
{% endhighlight %}

## Oliver Twist

Filtering the data to just show Oliver Twist, and adding some rolling mean values for positive/negative sentiment.


{% highlight r %}
# Get OT
OT <- 
    df %>% 
    filter(book=="Oliver Twist")

# by line
OT.l <- 
    OT %>% 
    group_by(line.no) %>% 
    summarise(p.n=sum(p.n)) %>% 
    mutate(roll.mean.50=rollmean(p.n, 50, fill = NA)) %>% 
    mutate(roll.mean.100=rollmean(p.n, 100, fill = NA)) %>% 
    mutate(roll.mean.200=rollmean(p.n, 200, fill = NA)) %>% 
    mutate(roll.mean.300=rollmean(p.n, 300, fill = NA)) %>% 
    mutate(roll.mean.400=rollmean(p.n, 400, fill = NA)) %>% 
    mutate(roll.mean.500=rollmean(p.n, 500, fill = NA)) %>% 
    mutate(roll.mean.600=rollmean(p.n, 600, fill = NA)) %>% 
    mutate(roll.mean.700=rollmean(p.n, 700, fill = NA)) %>% 
    mutate(roll.mean.800=rollmean(p.n, 800, fill = NA)) %>% 
    mutate(roll.mean.900=rollmean(p.n, 900, fill = NA)) %>% 
    mutate(roll.mean.1000=rollmean(p.n, 1000, fill = NA)) %>%
    mutate(roll.mean.1500=rollmean(p.n, 1500, fill = NA)) %>% 
    mutate(roll.mean.2000=rollmean(p.n, 2000, fill = NA)) %>% 
    mutate(roll.mean.2500=rollmean(p.n, 2500, fill = NA)) %>% 
    mutate(roll.mean.3000=rollmean(p.n, 3000, fill = NA)) %>% 
    mutate(roll.mean.3500=rollmean(p.n, 3500, fill = NA)) %>% 
    mutate(roll.mean.4000=rollmean(p.n, 4000, fill = NA)) %>% 
    mutate(roll.mean.4500=rollmean(p.n, 4500, fill = NA)) %>% 
    mutate(roll.mean.5000=rollmean(p.n, 5000, fill = NA))
{% endhighlight %}

Now we find the start point for each chapter


{% highlight r %}
rawText <- read_lines("books/OliverTwist.txt", skip = 150, n_max = 18682) # skip non-book bits
OTr <- data.frame(book="Oliver Twist", line=rawText)

# get chapter
OTr$chapter <- str_match(OTr$line, "CHAPTER ([:alnum:]+)")[,2]

# propogate chapter number
OTr$chapter <- na.locf(OTr$chapter)

# remove unneeded info
OTr <- 
    OTr %>% 
    filter(line!="") 

# get line no
OTr <- 
    OTr %>% 
    mutate(line.no=row_number())

# convert factors to char vectors
OTr$book <- as.character(OTr$book)
OTr$line <- as.character(OTr$line)

# convert chapter to number
OTr$chapter <- as.numeric(as.roman(OTr$chapter))

# reduce to single words
OTr <- 
    OTr %>% 
    unnest_tokens(word, line)

# get rid of stop words
data("stop_words")
OTr <- 
    OTr %>%
    anti_join(stop_words)
{% endhighlight %}



{% highlight text %}
## Joining, by = "word"
{% endhighlight %}



{% highlight r %}
# turn back into lines
OTr <- 
    OTr %>% 
    group_by(line.no) %>% 
    summarise(chapter=mean(chapter))

# merge with OT.l
OT.l <- left_join(OT.l, OTr, by="line.no")

# propogate chapter number
OT.l$chapter <- na.locf(OT.l$chapter)

# convert chapter to fraction
OT.l$chapter <- OT.l$chapter / 100.0

# get first line.no of each chapter
chapter.line.nos <- 
    OT.l %>% 
    mutate(index=row.names(.)) %>% 
    group_by(chapter) %>% 
    summarise(first.line.no=min(index)) %>% 
    mutate(first.line.no=as.numeric(first.line.no))
{% endhighlight %}

## Graphing Positive/negative Sentiment in Oliver Twist

Here we graph the positive/negative sentiment in Oliver Twist by creating a number of rolling mean values throughout the text to create a graph that shows both the general feel of a section of text without losing all the of the shorter highs and lows of the book.


{% highlight r %}
g.min <- -0.5
g.max <-  0.5
g.max.gap <- 0.03

p <- ggplot(data = OT.l) +
    geom_bar(aes(x = 1:nrow(OT.l), y = roll.mean.50,   fill=roll.mean.50),
             stat = "identity", position = "dodge", alpha=0.1, width=1) +
    geom_bar(aes(x = 1:nrow(OT.l), y = roll.mean.100,  fill=roll.mean.100),
             stat = "identity", position = "dodge", alpha=0.1, width=1) +
    geom_bar(aes(x = 1:nrow(OT.l), y = roll.mean.200,  fill=roll.mean.200),
             stat = "identity", position = "dodge", alpha=0.1, width=1) +
    geom_bar(aes(x = 1:nrow(OT.l), y = roll.mean.300,  fill=roll.mean.300),
             stat = "identity", position = "dodge", alpha=0.1, width=1) +
    geom_bar(aes(x = 1:nrow(OT.l), y = roll.mean.400,  fill=roll.mean.400),
             stat = "identity", position = "dodge", alpha=0.1, width=1) +
    geom_bar(aes(x = 1:nrow(OT.l), y = roll.mean.500,  fill=roll.mean.500),
             stat = "identity", position = "dodge", alpha=0.1, width=1) +
    geom_bar(aes(x = 1:nrow(OT.l), y = roll.mean.600,  fill=roll.mean.600),
             stat = "identity", position = "dodge", alpha=0.1, width=1) +
    geom_bar(aes(x = 1:nrow(OT.l), y = roll.mean.700,  fill=roll.mean.700),
             stat = "identity", position = "dodge", alpha=0.1, width=1) +
    geom_bar(aes(x = 1:nrow(OT.l), y = roll.mean.800,  fill=roll.mean.800),
             stat = "identity", position = "dodge", alpha=0.1, width=1) +
    geom_bar(aes(x = 1:nrow(OT.l), y = roll.mean.900,  fill=roll.mean.900),
             stat = "identity", position = "dodge", alpha=0.1, width=1) +
    geom_bar(aes(x = 1:nrow(OT.l), y = roll.mean.1000, fill=roll.mean.1000),
             stat = "identity", position = "dodge", alpha=0.1, width=1) +
    geom_bar(aes(x = 1:nrow(OT.l), y = roll.mean.1500, fill=roll.mean.1500),
             stat = "identity", position = "dodge", alpha=0.1, width=1) +
    geom_bar(aes(x = 1:nrow(OT.l), y = roll.mean.2000, fill=roll.mean.2000),
             stat = "identity", position = "dodge", alpha=0.1, width=1) +
    geom_bar(aes(x = 1:nrow(OT.l), y = roll.mean.2500, fill=roll.mean.2500),
             stat = "identity", position = "dodge", alpha=0.1, width=1) +
    geom_bar(aes(x = 1:nrow(OT.l), y = roll.mean.3000, fill=roll.mean.3000),
             stat = "identity", position = "dodge", alpha=0.1, width=1) +
    theme_minimal() +

    geom_linerange(aes(x = chapter.line.nos$first.line.no[1], ymin = g.min, ymax = g.max), colour="grey") +
    geom_linerange(aes(x = chapter.line.nos$first.line.no[5], ymin = g.min, ymax = g.max), colour="grey") +
    geom_linerange(aes(x = chapter.line.nos$first.line.no[10], ymin = g.min, ymax = g.max), colour="grey") +
    geom_linerange(aes(x = chapter.line.nos$first.line.no[15], ymin = g.min, ymax = g.max), colour="grey") +
    geom_linerange(aes(x = chapter.line.nos$first.line.no[20], ymin = g.min, ymax = g.max), colour="grey") +
    geom_linerange(aes(x = chapter.line.nos$first.line.no[25], ymin = g.min, ymax = g.max), colour="grey") +
    geom_linerange(aes(x = chapter.line.nos$first.line.no[30], ymin = g.min, ymax = g.max), colour="grey") +
    geom_linerange(aes(x = chapter.line.nos$first.line.no[35], ymin = g.min, ymax = g.max), colour="grey") +
    geom_linerange(aes(x = chapter.line.nos$first.line.no[40], ymin = g.min, ymax = g.max), colour="grey") +
    geom_linerange(aes(x = chapter.line.nos$first.line.no[45], ymin = g.min, ymax = g.max), colour="grey") +
    geom_linerange(aes(x = chapter.line.nos$first.line.no[50], ymin = g.min, ymax = g.max), colour="grey") +
    geom_linerange(aes(x = chapter.line.nos$first.line.no[53], ymin = g.min, ymax = g.max), colour="grey") +

    annotate(geom="text", x=chapter.line.nos$first.line.no[1], y=g.max+g.max.gap, label="1", color="grey") +
    annotate(geom="text", x=chapter.line.nos$first.line.no[5], y=g.max+g.max.gap, label="5", color="grey") +
    annotate(geom="text", x=chapter.line.nos$first.line.no[10], y=g.max+g.max.gap, label="10", color="grey") +
    annotate(geom="text", x=chapter.line.nos$first.line.no[15], y=g.max+g.max.gap, label="15", color="grey") +
    annotate(geom="text", x=chapter.line.nos$first.line.no[20], y=g.max+g.max.gap, label="20", color="grey") +
    annotate(geom="text", x=chapter.line.nos$first.line.no[25], y=g.max+g.max.gap, label="25", color="grey") +
    annotate(geom="text", x=chapter.line.nos$first.line.no[30], y=g.max+g.max.gap, label="30", color="grey") +
    annotate(geom="text", x=chapter.line.nos$first.line.no[35], y=g.max+g.max.gap, label="35", color="grey") +
    annotate(geom="text", x=chapter.line.nos$first.line.no[40], y=g.max+g.max.gap, label="40", color="grey") +
    annotate(geom="text", x=chapter.line.nos$first.line.no[45], y=g.max+g.max.gap, label="45", color="grey") +
    annotate(geom="text", x=chapter.line.nos$first.line.no[50], y=g.max+g.max.gap, label="50", color="grey") +
    annotate(geom="text", x=chapter.line.nos$first.line.no[53], y=g.max+g.max.gap, label="53", color="grey") +
    ylab("Sentiment") + 
    ggtitle(sprintf("Positive and Negative Sentiment in %s", "Oliver Twist")) +
    theme(plot.title = element_text(size = 16, face = "bold")) +
    theme(legend.title=element_blank()) + 
    theme(legend.position = "none") +
    theme(axis.title.x=element_blank()) +
    theme(axis.ticks.x=element_blank()) +
    theme(axis.text.x=element_blank()) +
    theme( legend.justification=c(1,1))
{% endhighlight %}
<a id="Graph_OT"></a>

![center](/figs/Dickens-Sentiment/graphing.OT-1.png)

The github repository for this project can be viewed [here](https://github.com/MarkPratley/Dickens-Sentiment-Analysis).




