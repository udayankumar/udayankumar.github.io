---
layout: post
title: Reddit comments
comments: true
tags: [reddit, comments analysis]
---
*TLDR*: The faster you comment on a post, more likely it is going to be top scoring comment.

There are several social networks, where anonymous people put out their views (in form of comments) on a topic. Other users can upvote(agree), downvote (disagree), or do nothing (no opinion, desire, or lack of interest among other reasons ). I want to test if time to post a comment has any relationship with the net score (upvote - downvote) received by a comment. 

This can have several implications - for social network users, they can achieve better scores and credibility  (Karma on Reddit) if they post comments at a particular time. PR managers can quickly control a PR nightmare  before it gets out of control, if they know when to post views from their perspective. Readers can know if they should visit a topic multiple times to read the most popular comments or would they be able to get all the popular comments, if they read a post after ‘x’ hours it’s publish time.


Metrics I will be looking into:

- Correlation between comment posting time and score received
- Correlation between comment length and score.
- Time difference between the post and the most popular comment
- Time difference between most popular and second most popular comment
- ECDF total score received by all the comments in the post with time
- Are there user accounts that always get higher scores (not yet done)
- Distribution of comment scores in a thread (not yet done)


Someone already looked at:
[When to post on reddit/HN to increase chances of reaching the front page]( http://arxiv.org/pdf/1501.07860.pdf)


### Data:

One month of Reddit comments made available to public by  [a Reddit User](https://www.reddit.com/user/Stuck_In_the_Matrix). This data set has more than 53 Million comments made on more than 3 Million posts/threads. I also downloaded their meta-info including date of posting for all the thread used in this post.. 

Most of analysis is restricted to top 3000 threads selected by the number of comments

{% highlight bash %}
grep -o 'link_id.\{0,13\}' RC_2015-01| cut -f 3 -d '"' | gsort --parallel=4 | uniq -c | gsort --parallel=4 > ThreadIdCommentCount

tail -n 3000 ThreadIdCommentCount | cut -f 2 -d ' '> popularThread_linkId
{% endhighlight %}

**Processing**: I have used my laptop for all the processing, I could have used Amazon/Google but I did not mind waiting 8Hrs to run a task as I was anyways going to bed. I am adding the code to the github repo - [add link]. 

One of the main challenges after identifying the thread that I was interested in was to identify all the comments belong to each of those thread. I created a grep based script for each of the linkId (Id for a thread) but it was taking just too much time to go through the datafile. To make this search a little faster, I created an index file that stored start and end position of each comment and its corresponding linkId. 

Now the process was much faster! It took only 8hrs to get all the comments for each of the 3000 threads!!

Steps:

- Create index on the comments file based on the link ID 

{% highlight bash %}
python createIndex.py RC_2015-01 ff > RC_2015-01_index
{% endhighlight %}

- get all the comments for popular threads in separate files  (8hr)

{% highlight bash %}
cat  popularThread_linkId | gxargs --verbose -d '\n' -n 1 -P 3 -I {} sh -c 'python readFileLinesAtIndex.py RC_2015-01 RC_2015-01_index {} > {}_comments.json'
{% endhighlight %}


## Metrics 0 :

For top 3000 post, I looked at the mean and median values for time to post a comment, score, and length of comment:

- Median time of posting a comment after thread’s post - 6 Hrs 6 Mins 
- Mean time of posting a comment after thread’s post - 25Hrs 24 Mins 
- Median comment score = 1 
- Mean comment score = 12 
- Median Length of comments - 66 Characters
- Mean Length of comments - 135 Characters

All the 3 fields are right skewed. Very long tail

## Metric 1:
`Correlation between comment posting time and score received`

For this I took the top 3000 most commented threads. For each thread, I considered the time difference between the thread post and comment post time. Now there are two ways to measure this correlation - 1.  For each thread measure correlation separately and then look at the  aggregate statistics  and 2. look at all the measurements at one go and then measure the correlation. I looked at both. Below are the details with processing steps. 

### Way 1
`Look at each threads correlation separately`

Steps :

Download the comment meta info if needed and then process and store correlation scores in a file (~2hrs)

{% highlight bash %}
cat  popularThread_linkId | gxargs --verbose -d '\n' -n 1 -I {} sh -c 'python RedditCommentsPostingTimeDelta_DownloadThreadInfo.py -o  cor {}_comments.json kk >> CorrelationScores.csv'
{% endhighlight %}

![help]({{ site.url }}/assets/images/correlation_time_score_onePerThread.png)
The above plots are for correlation measurements on time to post comment with score received. We have 3000 such scores that have been plotted using a histogram - coefficient and P-values. One can clearly see that almost all the coefficients are below zero - indicating a negative correlation. The P values are transformed so that values less than 0 indicates the 5% confidence interval where null hypothesis is rejected and the alternative hypothesis is accpeted. Implying that there is a correlation between time to post comment and the score received; the correlation is negative.   

Similar to above methodology, now we look at correlation between length of comment and score received in the plot below. There is no clear concensus in this case for the alternative hypothesis from the P-values although most of the correlation coefficients are positive. 

![help]({{ site.url }}/assets/images/correlation_len_score_onePerThread.png)


### Way 2
`Combine data from all the threads`


{% highlight bash %}
cat  popularThread_linkId | gxargs --verbose -d '\n' -n 1 -I {} sh -c 'python RedditCommentsPostingTimeDelta_DownloadThreadInfo.py -o all {}_comments.json kk >> CorrelationScores.csv'
{% endhighlight %}

Total comments from 3000 threads -  7.5 Million!  We see that correlation between score and posting time is -0.011 and the p value is 2.2e-16. So null hypothesis can be rejected and we can say that there is a correlation between comment posting time and score received.  Also, we see that there is correlation with length of the comment and the score - coefficient = 0.0268 and pvalue = 2.2e-16. 

BONUS: below is a plot of time to post and score received. Almost all the top scoring comments are posted with a few hundred minutes.
![help]({{ site.url }}/assets/images/Scatterplot_score_time_allThreadsTogether.png)

## Metric 2 

Now I will be looking only at most popular comments in each of the 3000 threads and study the distribution of time to post, scores, and length of comments. Below are some of the charts 

![help]({{ site.url }}/assets/images/histEcdfTime.png)

75% of the highest scoring comments have been posted in less than 100 minutes of thread posting. Faster you post the higher are your chances of getting highest score.

![help]({{ site.url }}/assets/images/histEcdfScore.png)
Scores are almost uniformly distributed for up to 2000. After that the probability to get higher score drops dramatically. From the CDF it can be seen that almost 50% of the top comments in  a thread get less than 2000 karma points. 

![help]({{ site.url }}/assets/images/histEcdfLen.png)
The length of comments distribution is again highly right skewed and almost 75% of the top scoring comments in top 3000 threads were less than 500 characters. Reddit users appreciate succinct comments (unlike this current post).


