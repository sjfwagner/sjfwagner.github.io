---
layout: post
title: Examining Gender Bias in 'Stuff You Missed in History Class'
date: '2016-06-01 20:18:39 -0400'
categories: datascience
published: true
---
#### One listener knows not what he says.

## A few notes: 

Throughout this post I will assume that women and men are equal in capabilities and unequal in opportunities. Opportunity is highly dependent on many things in society, including religion, skin color, socioeconomic status, and (as examined here) gender; whereas capability is not.

Additionally, this post will describe gender overall as a binary male and female -- despite the fact that this demonstrably false and that gender exists on a spectrum. This post will therefore exclude transgender and non-binary persons. This is for a few reasons, first) the data I have from Pantheon 1.0 that was compiled by outside sources does not include such considerations and most analysis requires the data to comparable (a line I am already stretching) second) for most of history non-binary and transgender individuals were not recognized (with a few notable exceptions) by their societies. It is best to assume that by male and female in this post, I refer to sex. In every other way I will strive to be as inclusive as possible, but please feel free to comment if I have missed anybody, my intention is to learn as much as possible.

Finally, the same day that this blog post was finally done, the ladies at SYMIH also posted a blog post on the same topic -on the same day!!! You can find their analysis [here](http://www.missedinhistory.com/blog/our-final-answer-on-too-many-women/). 

## Now, The Post

“Stuff You Missed in History Class” is a delightfully educational podcast by Tracy V. Wilson and Holly Frey of “How Stuff Works.” It’s purpose is to provide an overview of historical people and events that you may have missed in in your history class either because they weren’t taught, or they were underexamined. They are deservingly popular and readers often write in to suggest topics or provide comments on recent episodes and interesting pieces of this listener mail are read aloud to the general audience during the “Listener Mail” section of the podcast. I was shocked to listen, some months ago in [their episode on Liszt](http://www.missedinhistory.com/podcasts/lisztomania/ "SYMIH"), to a listener Ross who wrote to the women thus: 

> “[In your episode about Calamity Jane] you complained about the man who gave you a one star review 
> because he said that you had too many episodes about women. I have two comments about your
> discussion about this. One, you said that you only have 5 episodes about women in the last
> twenty and therefore you were not doing enough episodes about women, while the 5 number is true
> it is also unrepresentative because in those twenty you had a number of episodes that were about
> neither men nor women -- like peanut butter and hippos -- I count exactly five episodes about
> men. More importantly I would have guessed that you had too many episodes about women too; the
> reason for this is that **when you guys do episodes about women you lose all objectivity**, for
> men you present facts, for women you celebrate her life, get excited, and offer commentary about
> how much you love her. This makes these episodes much more noticeable and for that reason I
> sometimes skip your episodes that are likely to be like that. In other words **episodes about**
> **men are history** while **episodes about women become a story** and all event. I would agree
> with the commentator that you discussed, although certainly not the 1-star review. And suggest
> that either episodes about men should be more exciting to you or you should become more
> objective when you discuss women, and **you should certainly not do even more episodes about**
> **women when the ratio of men to women in the twenty episodes that you chose was one-to-one.**”

Holly and Tracy refuted him far more politely than he deserved, but the accusation has been burning in my mind for the last few months.  They mostly refute some of the listener’s comments by clarifying the 20 episode sample they examined and by saying that they were equally excited about men and they were about women (something I certainly believe). They also brought up a really important note on the perception of the equality of genders, namely that in media a crowd with 30% women will be evaluated by viewers as containing 50% women, and a real ratio of 50% will be perceived as much higher (something that extends beyond media).  I believe this listener’s misguided request points to a larger societal problem about the visibility and treatment of women, and how we teach history in this country. Below is my examination of Gender Bias in “Stuff you missed in History Class.” I’ll be investigating a few key questions from the reader’s comment, as well as those implied. 

##  _Men & Women: Who gets episodes on Stuff You Missed in History Class?_  

I have classified each podcast throughout Stuff You Missed in History Class’s history into 8 categories, two major categories (‘Loose’, & ‘Tight’) and four subcategories (‘Male’,’Female’,’Neutral’, and ‘Both’). 

- Loose - this is a general or colloquial understanding, where labels are applied more liberally. For instance, if a podcast on the Titanic mostly describes the men who designed, built, and crashed her, it would be coded as ‘Male’
	- Male - podcast is explicitly about men or covers mostly male achievements
    - Female - podcast is explicitly about women or covers mostly female achievements
    - Neutral - podcast is about an place, thing, or event that is not about either male or female achievement
    - Both - podcast is about both men and women, roughly equally
- Tight - this is a strict understanding, where labels are placed according to the topic of the podcast, and gender only for explicitly male or female topics, all others neutral or both
	- Male - podcast is explicitly about men
    - Female - podcast is explicitly about women
    - Neutral - podcast is about an place, thing, or event that is not about either male or female achievement
    - Both - podcast is about both men and women, roughly equally

Using these categories, I have classified the nearly 900 podcasts since 2008. I have tried to be objective, but there will be a level of error introduced where a particular person may agree or disagree with me on my subjective determinations. 

### _Men & Women in SYMIH’s episode archive_

I used the archive provided at their website, and the descriptions at the episode’s profile (using some light webscraping, notebook with code [here](https://github.com/sjfwagner/Blog-code-snippets-and-notebooks/blob/master/SYMIH_Scrapes.ipynb)), to collect the title, date published, and the synopsis of the episode. After collecting this into a single google sheet, I used the synopsis or listened to the episode where necessary, to determine the Gender (as described in the previous section) for each the nearly 1,000 episodes (DAMN LADIES) (see my data [here](https://github.com/sjfwagner/Blog-code-snippets-and-notebooks/blob/master/SYMIH.csv)). 

**In whole, men get more episodes than women by far on SYMIH**

<a href="http://imgur.com/KlcB0R2"><img src="http://i.imgur.com/KlcB0R2.jpg" title="source: imgur.com" /></a>

If we examine the 816 episodes that I covered in my analysis through the lens of the ‘gender loose’ examination (see above). We can see that MOST of the episodes done by Stuff you Missed are either explicitly about men or about subjects that mostly involve male achievement. Women or their achievements make up just a fifth of all episode subjects (more three fifths are about men a difference of roughly 40%). As seen below, when women do have episodes it tends to be biography rather than about their achievements, as most female episodes remain female in the tight categorization. 

<a href="http://imgur.com/5wpIe2j"><img src="http://i.imgur.com/5wpIe2j.jpg" title="source: imgur.com" /></a>

From loose to tight the percentage of episodes about men go down roughly 15%, where the percentage about episodes about women go down by only 2%. The difference is mostly in the movement of podcasts about male projects or achievements into the “Neutral”* category. Still over half of the 816 episodes are about men (that’s 360 episodes). So Ross, over the long run it you’ve had your wish -- most of the episodes SYMiH have done are about men! 

**Over time, men _still_ get more episodes than women by far on SYMIH**

In fact, on average over time SYMIH hosts have had roughly 5.5 male podcasts and 1.8 female podcasts per month (loosely), though the precise numbers move up and down depending on the month.

<a href="http://imgur.com/ymNK42i"><img src="http://i.imgur.com/ymNK42i.jpg" title="source: imgur.com" /></a>

<a href="http://imgur.com/Fwu29q5"><img src="http://i.imgur.com/Fwu29q5.jpg" title="source: imgur.com" /></a>

The numbers for men go down again, to roughtly 4.4 for male podcasts per month and 1.6 female podcasts per month when evaluately tightly. Perhaps the strongest argument for the "we need more male episodes" crowd comes from this -- when evaluated tightly men's episodes go down far more than female episodes. I expect that the male listeners writing in have a tendency to view male-oriented episodes as neutral rather than male-coded. As a woman living in a 'male-default' world, this is baffling.

##  _Men & Women: How does historical documentation actually break out?_ 
This data, from [Pantheon 1.0](http://arxiv.org/ftp/arxiv/papers/1502/1502.07310.pdf), represents information on the most globally popular biographies on Wikipedia. Manually verified, it represents data collected on the 11,341 biographies which have been translated into 25 languages. I will use this as a stand in for the interest of these people in history, and how we study them, divided across gender lines. Despite making up half of the human population\** women are far less represented in the number of biographies, at most representing 23.5%of the biographies in any 10 year period of history.  

<a href="http://imgur.com/loIhArQ"><img src="http://i.imgur.com/loIhArQ.jpg" title="source: imgur.com" /></a>

The long tail on the left demonstrates that for both men and women, that for most of history there were few historical figures of much prominence today (but fewer women), and that around the 1800s we see the number of notable personages of historical significance start to climb exponentially.  I believe that this can be generically ascribed to two phenomena 1) increased access to opportunity (democratization of government, increasingly permeable social strata, and more educational opportunities) and 2) increased attention on accurate documentation of history. Further, since this data represents only ‘popular’ biographies it may be that readers are simply more interested in people in history from the relatively recent past. There is a stark drop off around the 1990s and 2000s, this is entirely expected given that few people from these last 30 years have attained popular historical significance yet (i.e. few 30 year olds are mentioned in the history books right now). 

<a href="http://imgur.com/d9DXPi2"><img src="http://i.imgur.com/d9DXPi2.jpg" title="source: imgur.com" /></a>

As history moves into more modern era (1900s and forward) we may anecdotally expect a few possible outcomes in the ratio of men to women in history, that the number of female biographies would grow at the same rate as men, maintaining their lower ratio, or that women may actually start to catch up with men as feminism begins to balance the scales between the two genders. Though the ratio of men to women becomes less egregious as we reach the modern era, over the time period the average ratio is 10:1 (men: women) . The ratio of men to women starts in the early 1800s*\*\* at  15:1 (male:female) and increases to a peak of  39:1 during the late 1800s. Steadily declining from the peak it finally reaches a low of 1:1 in the late nineties (though averages 5:1 during the period). 

Given the historical systematic oppression of women, the deficit of female biographies to male may not be surprising even in the more recent past , as few women had access to  opportunity when compared to men. However, the fact that we do not see women begin to “catch-up” with men, especially in light of the many advancements that feminism has made is disheartening. It shows that we have a long distance to cover in terms of gender equality in historical documentation. 

**So you’re telling me is that “What You Missed in History Class” should be mostly about men then, right?**

“History is written by the the winners,” and human history has been more or less been written about men by men. Why doesn’t this mean that “Stuff you Missed in History Class” should focus on men? Mostly because history is also taught by the winners. All throughout Western Civilization we teach the history based mostly on the lives and actions of men (white men). So when it comes to considering what’s “missed in history class,” there will reasonably be just cause to talk more about women and minorities, since you were far more likely to miss their history in class. How can we examine my claim? At this juncture it’s worth asking ourselves “Who do we read about today?” rather than “Who was documented?”\*\*\*\*

<a href="http://imgur.com/2b6wZE5"><img src="http://i.imgur.com/2b6wZE5.jpg" title="source: imgur.com" /></a>

From this graph we can see that men still far out strip women in pageviews, women never (but once) even reaching parity. In this graph the ratio of men to women becomes barely more equal. Viewers are particularily interested in biographies of women born recently, but that biographies of women born before 1900 recieve almost no views compared to male biographies. 

### _So if we're talking about covering what you *missed* in history class, we probably need more podcast episodes about women rather than men -- even though finding content on women and female achievements in history is difficult._


_Interested in more? Part II is coming soon! I'll expand on the "who history is written by" argument, as well as providing a more in-depth statistical analysis of some of these comments. Thanks for reading!_



\* Examples: “How Presidential Salaries Work” -- no president has ever been female, and for much of the Republic they haven’t been allowed to be female, theoretically neutral but male oriented in reality. “How the Louisiana Purchase Worked” -- purchased by Thomas Jefferson (M) from Napoleon (M) but about a purchase of territory (N), “How the Titanic Worked” -- a ship (N) designed by men, constructed by men, and captained and sailed by men. 

\** There is a gender skew in human birth rate, roughly 101 males to 100 females. In some countries sex ratio is further skewed by gender-selective abortion, infanticide, and other societal effects. In practice we may generally assume that throughout history women and men have had roughly equal birth and survival rates, and that they represent an equal number of deceased humans. This assumption is valid as long as we consider the world as a whole, and not specific countries (see here for data on how specific countries break out in terms of gender ratio). 

\*\**  I chose to begin my examination of the ratio of men to women because during this period we see the number of biographies per birth year for each gender begin to consistently be above 0.  

\*\*\** I really would like to examine this in context of what is ACTUALLY taught in real history books. However I was not able to find good data on topics/people taught in US history classes. I have a lot of ideas about how actually to scrape and grab this data, but none of them are fast. So for now I’ll focus on the data I have available. If you have suggestions please share.
