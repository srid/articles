% Feelings on blackboard
% 
% May 09, 2015

## Motivation

As humans we love to discuss and debate what makes us happy. But can this detached intellectual activity be relied upon when psychological denial is a mark of being human? No matter how sincere we aim to be there are nooks and corners in the psyche we tend to not acknowledge much less address. The plethora of beliefs, continually formed and reinforced by both [social conditioning](https://en.wikipedia.org/wiki/Social_conditioning) and [memory biases](https://en.wikipedia.org/wiki/List_of_memory_biases), makes it harder to identify the facts. However, it *is* practical to sincerely answer the question "*How am I feeling right now?*" as it depends on attentiveness and not recall. By asking this each and every moment, the data is accumulated over time (this is called [self-tracking](https://en.wikipedia.org/wiki/Quantified_Self)). Giving each data point a context (location, event, people) can lead to identifying insights as to what *actually* makes us happy or unhappy as opposed to what we remember (involving memory biases) or are taught (via social conditioning) to be causing them.

## Blackboard

Since about two months ago I have been using a blackboard to keep a calendar with colored marks on each day. A blue circle indicates that I generally felt good throughout that day. Red circle indicates having had a generally bad day. Half-blue/Half-red is mixed. My inspiration for doing this came from the productivity hack called [Don't Break The Chain](http://www.writersstore.com/dont-break-the-chain-jerry-seinfeld/). The purpose of doing it however is slightly different; it is to get insights as to how one is generally doing in the affective health arena. I won't get into the details other than say that I found this to be pretty instructive.

## Context and fine-grained recording

The blackboard calendar had no context to the individual days other than three colored marks. Without context, it is hard to say what actually caused a bad day. Context is essentially the people, things, events, location involved when recording the "data" (one of the coloured marks). Furthermore, the data was only being collected on a daily basis; so I decided to do it frequently. As such I wrote a little Go web app called [feelings](https://github.com/srid/feelings). I have been using it for more than two weeks now. At various intervals, especially when something causes me to feel better or worse, I enter the data into the web app - recording *how* I am feeling (good, neutral, bad) and optionally *what* is the feeling and what *triggered* it, with additional comments. It didn't take more than two days to see the usefulness of it. I quickly came to the experiential realization that recall of how, say, the day before yesterday affectively happened is more often than not inaccurate. Many times I did fill in the "trigger" field. Over time I got a better sense of what caused me to feel worse or better. The things I *thought* made me feel good don't *actually* do! 

## Problems with the web app

Although I designed the web app to be mobile friendly with authentication built in (many a times I would access it from my iPhone's web browser), the whole process felt less slick than using a native app. A number of things tend to discourage me from frequently recording data:

* Mobile browsers periodically prompt for basic auth credentials. Entering this info is slower.
* HTML page loading is slow compared to native app view loading
* Form submission and getting the response back is also slow

Not being *greatly* experienced with mobile app development, my plan is to use [React Native](https://facebook.github.io/react-native/) for writing a minimal iOS app in JavaScript that does little more than allow me enter the data on frequent basis, storing them possibly in [Heroku Postgres](https://www.heroku.com/postgres). And then write a server-side app, or even script, to process this data to present visualization. Later on as I gain more experience the mobile app can be made more comprehensive.

I have further ideas about this whole thing, but that will have to wait till the next blog post. I decided to write this (somewhat random) post only to record my current attempts for future reference - because, you know, human memory cannot be totally relied upon.
