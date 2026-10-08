# Community Post Classification Plan

## Project Overview
In this project I will build a classifier for posts in a community forum, with the goal of separating posts by their main communicative purpose. I am choosing th Tasty.co baking community because cooking discourse is varied enough to produce genuine differences between categories but still narrow enough that a model can learn useful patterns.

## Community
 I am choosing th Tasty.co baking community because cooking discourse is varied enough to produce genuine differences between categories but still narrow enough that a model can learn useful patterns. Specifically I chose comments under the popular 'Brown Butter Toffee Chocolate Chip Cookies' recipe. Choosing a specific comment section under a recipe also reduces the types of words the tool has to look out for whilst having a varying comment type. A baking review section is also a lot less random than a fandom subreddit for example as people have the specific goal of talking about the cookies rather than general information. There is still variation in tone, questions and type of question which means that the tool can still learn from without it being too broad that the task becomes impossible.


## Labels
I will use 3 labels.

1. Reviews
   This label applies to posts where the author telling readers about their experience making and tasting the cookie and/or presenting their results.
   Example posts:
   - “I baked a couple after 3 hrs this is the result. very good. will bake the rest tomorrow or the next day. hopefully they live up to the expectation a no d the wait.”
   - “This recipe makes a lot more than 9 cookies”

2. Changes
   This label is for posts where authors are commenting  substitutions or changes they made to the recipe which can be good or bad.
   Example posts:
   - “Baked a few pieces after chilling for an hour and it turned out crunchy on the outside, chewy in the inside! Replaced toffee with Walnuts, which suited my need to contrast the sweetness of the cookie dough. Can’t wait to bake the rest after 3days!😍”
   - “so, uh if you're 9 months pregnant and wake in the dead of night NEEDING to make these cookies and (due to pregnancy brain) accidentally leave out 1 of the required cups of flour, these still turn out amazingly well. they spread like crazy and are the thinnest cookies I've ever seen but they are amazingly soft and chewy not crunchy at all!”

3. Tips
   This label is for people providing tips for bakers who might not know the best way to get around struggles. These are people with assumed knowledge over baking.
   Example posts:
   - “espresso powder makes them better but if you dont have any and dont feel like going to the store dont worry! theyll still be amazing (best cookie ive ever had btw)”
   - “If you are impatient but still want the effect of the resting you can do the freezer instead of the fridge. Just do it for a fourth of the time and make sure it’s sealed. Also I the dough does freeze you can thaw it in the fridge before baking.”

3. Confusion
   This label is for people confused about the video or written recipe and who are simply expressing confusion or asking questions.
   -"How come the video calls for 4 eggs and the recipe calls for 2!How many eggs should I use?2, 3 or 4?"
   -"Like if he doubled the recipe cause I’m so confuse don’t like if he didn’t"
   

These labels are distinct enough to be useful and broad enough to cover the kinds of discourse seen in this specific comment section and ones similar. They also map to real community needs: recomendations, preferences, reviews, and baking substitutions.

## Hard Edge Cases
The hardest edge cases will be posts that sit between Tips and Recommendation / Subtitutions and Changes because some recomendations could be considered a change to the recipe. The main difference is that S&C indicate the author happening upon a cool change or substitution they can make which may not turn out well whilst T&R is more experienced bakes sharing good advice. 

Another case that might cause confusion is if a review includes a recommendation or the other way around.

During annotation, I will handle these cases by using the post’s primary rhetorical purpose as the deciding factor. I will ask: what is the author trying to do most strongly in this post? If the main goal is to recommend and it appears as though they are describing general advice it is Tips & Recommendations. If the main goal is to share a personal experience that led to a change based on the authors preferences then it goes under Changes and Substitutions. I will also prioritize Recommendations and Substitution labels over reviews.

If a post is still balanced between two labels after that decision rule, I will annotate it with the label that better matches the dominant action and will flag it for review in the error analysis stage. This prevents the project from conflating ambiguous posts with noisy labels.

## Data Collection Plan
I will collect comments from the review section. I will begin by collecting and labeling examples in batches of 25. After each batch, I will review the class balance and look for underrepresented categories. If a label is underrepresented after 200 examples, I will first seek out this label specifically amongst comments then check whether this is reflective of community pattern. The goal is to keep each label reasonably balanced while still reflecting real community patterns.

I will not force a label to 50 examples if the community itself rarely produces it; instead, the rule will be to keep the label representation above a minimum threshold and document any imbalance clearly in the evaluation report.

## Evaluation Metrics
Accuracy alone is not enough because the classes are not equally distributed and some types of mistakes are more costly than others. A model could have a decent overall accuracy while still failing on the labels that matter most, especially the ones that are less common but harder to classify. For this task, I will use a combination of macro F1, per-class precision and recall, weighted F1, and a confusion matrix.

Macro F1 will be the main metric because it gives each label equal weight, which is important when the comment classes are imbalanced. If one label appears more often than the others, a model could look strong on weighted metrics while still performing poorly on minority categories. Per-class precision and recall will show whether the model is over-predicting a label or missing it entirely. For example, if the model confuses Substitutions and Changes with Tips and Recommendations, precision and recall will show whether that is a systematic issue and whether the problem is mostly false positives or false negatives.

Weighted F1 will be reported as a secondary metric because it reflects general performance across the dataset while still accounting for class frequency. Finally, I will inspect the confusion matrix to identify which labels are most often mixed up. This is the most important tool for this project because the hardest boundaries are between Tips and Recommendations and Substitutions and Changes, and between Results and Reviews and Questions and Confusion. If these pairs are frequently confused, then the label definitions or annotation rules need to be tightened before the model is considered reliable.

## Definition of Success
A classifier would be genuinely useful if it can identify the dominant purpose of a comment with enough consistency that a human reviewer can trust it for pre-labeling or triage. For this project, I would consider the model successful if it achieves at least 0.75 macro F1 on a held-out validation set and at least 0.70 F1 for each individual label. I would also want the confusion matrix to show that the model is not repeatedly mixing the closest labels, especially Tips and Recommendations with Substitutions and Changes.

For deployment in a real community tool, I would consider the model functional if it is accurate enough to save a human reviewer time on the easier cases without creating confusing false positives. In practice, that means the model should handle typical recipe comments reliably while leaving the genuinely ambiguous ones for manual review. I would not accept a model that only does well on the dominant class or that performs well on clear cases but fails on the edge cases that often appear in cooking comment threads.

 
## AI Tool Plan

### Label stress-testing
Before starting annotation I will give my AI agent 7 labels to chech if it annotates accuratelty acordding to my labeling. I will then shift accordingly if it does not do well.

| Comment | Label | Description |
| --- | --- | --- |
| "Always chill the dough at least 24 hours. I only did 12 because I was impatient and they spread all over the pan, so now I never skip it." | Tips & Recommendations | Edge case: Tips & Recommendations vs. Changes & Substitutions. It starts as a personal shortcut but ends as general advice. The main goal is to warn other bakers, and the personal story only supports the advice. |
| "I didn't have espresso powder so I used a teaspoon of instant coffee dissolved in a little water, and honestly I couldn't tell the difference." | Changes & Substitutions | Edge case: Changes & Substitutions vs. Tips & Recommendations. The author is sharing a swap, but it could also be read as a tip for anyone without espresso powder. The author is describing their own substitution and result, not advising others. |
| "Just use less sugar. I cut it by a third and they came out perfect, not overly sweet at all." | Tips & Recommendations | Edge case: Tips & Recommendations vs. Changes & Substitutions. It has an imperative opening but is backed by a personal change. The imperative is the dominant action, but this one is nearly balanced and should be flagged for error analysis. |
| "Best cookies I've ever made!! The brown butter is a game changer. Definitely let it cool before adding the ice though or it'll bubble over." | Tips & Recommendations | Edge case: Results & Reviews vs. Tips & Recommendations. It opens as a review and closes with specific advice. The rule prioritizes recommendations over reviews. |
| "Loved these! Skipped the toffee since I'm not a fan and they were still super chewy and rich." | Changes & Substitutions | Edge case: Results & Reviews vs. Changes & Substitutions. The review is the larger part, but it includes a substitution. The substitution label outranks the review. |
| "These were so good, I would 100% recommend this recipe to anyone, even total beginners!" | Results & Reviews | Edge case: Results & Reviews vs. Tips & Recommendations. The word “recommend” appears, but the author is endorsing the recipe, not giving baking advice. This shows recommendation priority applies to advice about how to bake, not general praise. |
| "Does anyone know if I can use salted butter? I'm planning to just skip the added salt but I'm not sure if that works." | Questions & Confusion | Edge case: Questions & Confusion vs. Changes & Substitutions. The author describes a planned change, but it has not been tried yet. The primary purpose is asking and the change is only hypothetical. |

### Annotation assistance
I will use claude to pre-label a small batches of examples before I review them. If the label does not align with my assesment I will make that clear. Every example will still be reviewed by me, and I will note where the pre-label was wrong to help refine the labeling guide.

### Failure analysis
I will give the AI tool a list of the model’s incorrect predictions and ask it to identify recurring error patterns. For this project, I will specifically look for cases where the model confuses the closest label pairs: Tips & Recommendations vs. Changes & Substitutions, Results & Reviews vs. Tips & Recommendations, and Questions & Confusion vs. Changes & Substitutions. I will also look for cases where the model relies too heavily on keywords such as “recommend,” “use,” “skip,” or “worked,” instead of identifying the author’s main communicative purpose.

I will then verify the AI’s suggestions by reading the misclassified comments myself and checking whether the problems are due to label definitions or annotation ambiguity. If the same issue repeats across multiple examples, I will revise the annotation guide or move borderline examples into the edge-case review set.

