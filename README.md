# counter_machine
## Design doc

Originally developed for docs-to-user recommendation task. Could also be applied to other, similar tasks: doc-to-doc recommendations, search, marketplace recommendations.

Let's assume we are building recommendations for something similar to TikTok: we have users (being possibly both creators and consumers), posts. Users consume the feed of posts and can like posts, watch them, dislike them, share them. We want to build a set of features that aggregate some metric for a given set of keys.  
For example, "total like count for a video" is a feature. "Video like-to-show ratio" is a feature. "Like-to-show ratio of this content creator for this user" is a feature (showing how much this particular user enjoys this particular author).

This repo represents a set of tools for _research_, _production implementation of offline data processing pipelines_, _production implementation of continuous data updating_, and _production feature serving_. Below, we will discuss these stages in more detail, why are they needed and what is important for each of these stages, key design points of the system, trade-offs made, and the final design solution that can be used in all of the stages described above.

### Key stages in ML quality project
#### Research stage
Maybe you have a product (something with recommendation feed), logs and a basic feed algorithm and you want to start gathering some features for your first actual ML thingie. Maybe you already have a lot of ML and other features and want to test an idea about some other event that is not already in your ML engine.  
The process will typically look like this: you have a bunch of ideas about features that you want to try, or at least events and information that you want to use in your model. For example, you want to generate counters based on shows, clicks, watch times for videos, likes, dislikes, shares, comments. For every event, you know ids for some entities (user, author, post, category, tags, genre, user location, ...). You then generate thousands upon thousands of features with different filtering, aggregation, grouping and time decay properties: like-to-show ratios, total likes, shares of videos with more than 30 seconds view time; like ratio aggregation for post_id (how good is this post?), post_id+user_id (has this user seen this post? Has he liked it before?), user+author (does this user enjoy this particular author?), user_country + author (is this author popular in this particular country?); counters for events from the "trending feed" and counters for all events; counters with time decay of 1 hour, 1 day, 1 week, 1 month, 1 year to be able to capture short-term and medium-term interests as well as have a widely-possible aggregate to have the most exhaustive information; than calculate different ratios: like-to-show ratio for post_id grouping to show the "quality" of the post; like-to-show ratio for 1day decay to 1 year decay to show how short-term is this trend; like-to-show ratio overall vs for this country to show how local is this trend. To do that, you write a config that specifies all these features (or rather, write a little bit of code that generates such a config).  Then run a batch-job (like mapreduce) to calculate features and join them to a dataset on a predefined set of historical data.

After some experiments on these exhaustive feature list, some feature selection can be done to select only necessary features (usually less than a hundred) out of all the thousands of them.

Can be done with: apache beam go-sdk

### Production data processing pipeline
After feature selection, we have a smaller config (a subset of the original config), but we now need to add these new features to our production. 