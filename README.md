# Brown Butter Toffee Chocolate Chip Cookie Comment Classifier

## Community Choice

This project compares a baseline text classifier with a fine-tuned model on comments from Tasty.co's Brown Butter Toffee Chocolate Chip Cookies recipe. I chose this community because the comments are focused on one recipe, unlike a sports channel or fan group where discussion can cover many unrelated topics. This focus keeps the task manageable while still providing variation in baking results, recipe changes, advice, and questions.

The labels used for this are `reviews`, `changes`, `tips`, or `confusion`.

## Community and label definitions

The data comes from comments on one recipe video rather than from a broad forum. This gives the comments a shared subject while retaining useful variation: people report baking results, describe recipe alterations, offer advice, and ask questions about the recipe or video.

| Label | Meaning | Examples |
| --- | --- | --- |
| `reviews` | The comment mainly shares the author's experience, results, or reaction to making or tasting the cookies, including general praise, criticism, or endorsement. | 1. “I baked a couple after 3 hrs this is the result. very good. will bake the rest tomorrow or the next day. hopefully they live up to the expectation a no d the wait.”<br><br>2. “This recipe makes a lot more than 9 cookies” |
| `changes` | The comment mainly reports a specific ingredient substitution or method change the author made. | 1. “so, uh if you're 9 months pregnant and wake in the dead of night NEEDING to make these cookies and (due to pregnancy brain) accidentally leave out 1 of the required cups of flour, these still turn out amazingly well. they spread like crazy and are the thinnest cookies I've ever seen but they are amazingly soft and chewy not crunchy at all!”<br><br>2. “I didn’t refrigerate them at all and they were still the best cookies I’ve ever had!” |
| `tips` | The comment mainly gives actionable, general advice to other bakers about how to prepare the recipe or improve the result. | 1. “Make sure it’s only 2 eggs and not 4. I tried it both ways! ITS TWO. DO NOT DO FOUR. lol. It was bad.”<br><br>2. “Make sure to keep your chocolate in big chunks! And when you make the browned butter, let it cool a little bit before you add the ice cubes because it will foam up & boil over!” |
| `confusion` | The comment mainly asks a question, expresses uncertainty, or points out something unclear about the recipe or video. | 1. “What even is going on here?”<br><br>2. “Anyone else own cookies come out super flat?” |

For mixed-purpose comments, apply these rules:

1. If the main purpose is asking a question or expressing uncertainty, choose `confusion`.
2. Otherwise, if the comment gives direct advice to other bakers, choose `tips`.
3. Otherwise, if it describes a recipe alteration the author made, choose `changes`.
4. Otherwise, choose `reviews`.

General praise such as “I recommend this recipe” is a review, not a tip.

## Data and methodology

- **Source:** Tasty.co comments for the Brown Butter Toffee Chocolate Chip Cookies recipe video.
- **Number of labeled examples:** 203 comments. I collected the most recent comments and omitted comments that were gibberish.
- **Collection and labeling:** Comments were collected from the recipe's comment section and saved with `label`, `text`, and `Notes` fields. I assigned each comment to one of the four categories using its main purpose and the mixed-comment decision rules above. Claude pre-labeled about 20 comments as annotation assistance; I reviewed its suggestions against my own labels.
- **Label distribution:** `reviews`: 87; `tips`: 60; `changes`: 49; `confusion`: 7.

- **Data split:** The dataset was divided into 142 training examples, 30 validation examples, and 31 test examples. The notebook used the validation set during training and reserved the test set for final evaluation.
- **Training label distribution:** 61 `reviews`, 42 `tips`, 34 `changes`, and 5 `confusion`.
- **Validation label distribution:** 13 `reviews`, 9 `tips`, 7 `changes`, and 1 `confusion`.
- **Test label distribution:** 13 `reviews`, 9 `tips`, 8 `changes`, and 1 `confusion`.
- **Baseline model:** Zero-shot `openai/gpt-oss-120b` accessed through Groq. It classified each of the 31 test comments independently using the written prompt. Its predictions were compared with the gold test labels; all 31 responses were parseable as one of the four labels.

- **Fine-tuned model:** `distilbert-base-uncased` fine-tuned using the Hugging Face Transformers `Trainer`. The tokenizer truncated comments at 256 tokens and used dynamic padding. The best checkpoint was selected using validation accuracy, and final metrics were calculated on the test set.
- **Prompt and label guide:** The Groq system prompt named the recipe and classification task, defined all four labels with examples, instructed the model to classify the comment's main purpose using the mixed-comment decision rules, and required exactly one lowercase label as output.

### Baseline system prompt

This is the system prompt used for the zero-shot Groq baseline:

```text
Classification prompt

You are classifying comments about the Brown Butter Toffee Chocolate Chip
Cookies recipe. Assign each comment exactly one label: `reviews`, `changes`,
`tips`, or `confusion`.

Classify the comment's main purpose, not just a keyword or its first sentence.
Read the whole comment and decide what the author is mainly doing. Do not infer
intent or details that are not in the comment. Ignore spelling, grammar,
capitalization, and emojis when deciding.

Labels

- **`confusion`** — The author asks a question, says they are unsure, or points
  out something unclear about the video or written recipe. Use this label even
  if the author mentions a change they are considering or have already tried;
  the question or confusion is the main purpose. A statement that merely reports
  a clear result is not confusion.
- **`tips`** — The author gives actionable advice to other bakers about what to
  do, avoid, or use to get a result. This includes instructions and general
  alternatives (for example, “make sure to cool the butter” or “if you do not
  have espresso powder, use instant coffee”). Advice may be supported by the
  author's own experience. Use `tips` when the comment's main purpose is to
  guide other bakers, even if it also reviews the cookies or mentions a personal
  mistake or change.
- **`changes`** — The author reports a specific alteration they personally made
  to the ingredients or method, such as adding, omitting, or swapping an
  ingredient, or changing a step. The author is describing what they did and
  what happened, rather than primarily instructing other bakers to do it. A
  change can be successful, unsuccessful, accidental, or just a planned change;
  if the comment's main purpose is asking whether a planned change will work,
  use `confusion` instead.
- **`reviews`** — The author mainly shares their experience, results, or
  reaction to making or tasting the cookies, including praise, criticism,
  quantity or texture observations, and general endorsement. Use this as the
  default when the comment does not primarily ask a question, give baking
  instructions, or report a specific change. Saying “I recommend this recipe”
  as general praise is a review, not a baking tip.

Decision order for mixed comments

Apply these rules in order:

1. If the main purpose is asking a question or expressing uncertainty about the
   recipe, choose `confusion`.
2. Otherwise, if the comment gives actionable, forward-facing baking advice,
   choose `tips`. This includes advice supported by a personal example.
3. Otherwise, if it reports a specific change the author personally made,
   choose `changes`, even when it also praises or criticizes the result.
4. Otherwise, choose `reviews`.

Do not label a comment `tips` just because it contains the word “recommend.”
General praise or recommending the recipe is `reviews`; advice about how to
prepare the recipe is `tips`. Do not label a comment `changes` merely because
it mentions an ingredient: the author must report a personal alteration, unless
the main purpose is asking about it, in which case use `confusion`.
```

### Fine-tuning configuration

The final fine-tuning run used:

| Setting | Final value | Earlier/default value |
| --- | ---: | ---: |
| Epochs | 6 | 3 |
| Learning rate | 3e-5 | 2e-5 |
| Per-device training batch size | 12 | 13 |

Other training settings included a per-device evaluation batch size of 32, weight decay of 0.01, and 50 warmup steps. The model was evaluated and checkpointed after each epoch, with the best checkpoint chosen by validation accuracy. I increased the epochs from 3 to 6 to give training more passes over the examples, raised the learning rate from 2e-5 to 3e-5 to make parameter updates somewhat larger, and reduced the training batch size from 13 to 12. These settings changed together, so the test results cannot show which individual change caused the final performance.


### Overall accuracy

| Model | Accuracy |
| --- | ---: |
| Zero-shot baseline (Groq) | 0.774 |
| Fine-tuned DistilBERT | 0.710 |

Fine-tuning regression: 0.064

### Per-class metrics

The baseline scores are strongest for `reviews` (F1 0.85), followed by `changes` (0.71) and `tips` (0.70). Its perfect score for `confusion` is based on only one test example, so it is not strong evidence of reliable performance on that category. The fine-tuned model performs better on `tips` and `reviews` F1 than on `changes`, but its overall accuracy is lower than the baseline.

| Model | Label | Precision | Recall | F1 | Support |
| --- | --- | ---: | ---: | ---: | ---: |
| Baseline | `tips` | 0.64 | 0.78 | 0.70 | 9 |
| Baseline | `reviews` | 0.85 | 0.85 | 0.85 | 13 |
| Baseline | `changes` | 0.83 | 0.62 | 0.71 | 8 |
| Baseline | `confusion` | 1.00 | 1.00 | 1.00 | 1 |
| Fine-tuned | `tips` | 0.67 | 0.89 | 0.76 | 9 |
| Fine-tuned | `reviews` | 0.71 | 0.92 | 0.80 | 13 |
| Fine-tuned | `changes` | 1.00 | 0.25 | 0.40 | 8 |
| Fine-tuned | `confusion` | 0.00 | 0.00 | 0.00 | 1 |

**Macro F1:** Baseline 0.82; fine-tuned 0.49.  
**Weighted F1:** Baseline 0.77; fine-tuned 0.66.

In this dataset, the baseline performed best on `reviews`; `changes` was more difficult because comments often describe a recipe alteration alongside an opinion about the result. The distinction between a personal change and advice to other bakers can also be subtle. The `confusion` score is not meaningful evidence of general performance because that class has only one example in the test set.

### Fine-tuned model confusion matrix

The matrix below is the test-set matrix for the fine-tuned DistilBERT model. Rows are the **actual** labels and columns are the **predicted** labels; diagonal values are correct predictions.

| Actual \ Predicted | `tips` | `reviews` | `changes` | `confusion` |
| --- | ---: | ---: | ---: | ---: |
| `tips` | 8 | 1 | 0 | 0 |
| `reviews` | 1 | 12 | 0 | 0 |
| `changes` | 2 | 4 | 2 | 0 |
| `confusion` | 1 | 0 | 0 | 0 |

### Fine-tuned model error analysis

**Gold label: `reviews`; predicted label: `tips`.** “Don’t make these if you want to be able to enjoy other food. They will blow your tastebuds away.” The comment uses a sarcastic warning to praise the cookies. The model may have interpreted the warning-like phrase “Don’t make these” as baking advice instead of reading the full comment as a humorous review. More varied examples of sarcasm could help the model learn this distinction.

“I made these and everyone loved them!! I mistakenly added a little extra cup of flour and so they aren't flat but the taste was still amazing! Will definitely make them again.” — Gold label: `changes`; predicted label: `reviews`. The comment includes praise, but it also explicitly reports an alteration the author made to the recipe. Under my label definition, accidental changes still count as `changes`, so the gold label is consistent with the annotation rule. The model may have focused more on the positive result and the plan to make the cookies again than on the reported flour change.

**Gold label: `confusion`; predicted label: `tips`.** “Something I would like to know, is how much of the browned butter is actually supposed to go in the mix.” This is a request for clarification, but the model classified it as a tip. The small number of `confusion` examples may have limited what the model could learn about this category. More representative examples would help assess and potentially improve its performance.

### Sample classifications

| Post (truncated) | True label | Predicted | Confidence | Correct? |
|---|---|---|---|---|
| This recipe was super delicious! But it’s not the classic chocolate chip cookie. | reviews | reviews | 0.65 | yes |
| Definitely cool down your batter, even if it’s just for a few hours. I made them same day ... | tips | tips | 0.42 | yes |
| If you’re cooking at altitude you should drop the temperature you cook the toffee to. A si... | tips | tips | 0.70 | yes |
| I highly recommend baking them on a silpat mat to avoid the toffee sticking to the pan or ... | tips | reviews | 0.35 | no |
| For some reason I had to add a lot of extra flour, but they were really good! | changes | reviews | 0.50 | no |


The third comment was correctly classified as `tips` because it directly advises bakers to change the toffee-cooking temperature at high altitude.

#### AI-assisted error-pattern review

**AI tool and request:** I shared the fine-tuned model's incorrect predictions with Copilot in VS Code and asked it to help identify recurring patterns and possible reasons for errors.

Patterns it suggested: 
Copilot pointed to possible reliance on surface wording rather than the comment's overall purpose. For example, imperative wording in “Don’t make these...” may have triggered `tips` despite the comment being sarcastic praise. Another example of this is when “I highly recommend...” may have been treated as a general review even though the rest of the comment gives practical advice. It also noted repeated confusion around comments labeled `changes` but predicted as `reviews` or `tips`, and flagged the very small number of `confusion` examples as a limitation.

My verification: I reread these examples using my label definitions. The sarcastic “Don’t make these...” comment is a review whose wording resembles a warning. The silpat comment is a tip because it gives a specific instruction to other bakers, despite beginning with praise. The browned-butter comment is confusion because the author is asking for clarification, but the model predicted `tips`; one example is not enough to establish a general pattern for that class. I kept the extra-flour example labeled as `changes`: although it mainly praises the result, it explicitly reports a recipe alteration, and my definition includes accidental changes. The model's `reviews` prediction is therefore a genuine error under my stated rules.

## Reflection: intended behavior versus learned behavior

I intended the model to classify each comment by its main communicative purpose, especially distinguishing a personal recipe change from advice aimed at other bakers. The fine-tuned model showed some success with common `tips` and `reviews` examples: its recall was 0.89 for `tips` and 0.92 for `reviews`. However, its precision for those labels was lower (0.67 and 0.71), so it also assigned those labels to comments from other categories. Its weakest result was `changes`: it correctly identified only 2 of 8 test comments in that class (recall 0.25), assigning four to `reviews` and two to `tips`. The single `confusion` test example was predicted as `tips`, but one example is too little evidence to judge how well the model handles that category generally.

Overall, the fine-tuned model did not outperform the zero-shot Groq baseline: its accuracy was 0.710 compared with 0.774, and its macro F1 was 0.49 compared with 0.82. The errors suggest that it sometimes responded to surface wording instead of the comment's full purpose. For example, it labeled the sarcastic “Don’t make these…” review as `tips`, apparently treating the warning-like opening as advice. It also labeled the silpat comment as `reviews` even though it gives specific practical advice. These examples suggest difficulty with sarcasm and mixed-purpose comments, while the confusion matrix shows a broader difficulty separating `changes` from `reviews` and `tips`.

The extra-flour example also shows how a mixed-purpose comment can be difficult: it reports an accidental change and then emphasizes the successful result. I labeled it `changes` because the comment explicitly describes altering the ingredients, consistent with my rule that accidental changes count. The model predicted `reviews`, suggesting it may have prioritized the praise and outcome over the change itself. To improve the model, I would add more examples where a personal change is described alongside a positive or negative review, as well as more varied examples of advice and confusion if the community provides enough genuine examples.

## Spec reflection

The spec helped guide my project by requiring me to define the labels and explain how I would evaluate the classifier. I used the main purpose of a comment to distinguish reviews, changes, tips, and confusion, and I used per-class metrics and a confusion matrix to see which categories the model struggled with and not just overall accuracy.

One way my implementation diverged from an ideal balanced dataset was that `confusion` remained rare in the comments for this specific recipe. I looked for more examples but chose not to add many that would misrepresent the discussion. This kept the dataset closer to the comments people actually posted, but left only one `confusion` example in the test set, so the model's score for that label is not substantial enough to support a useful conclusion. With more time, I would collect comments from a broader set of recipe discussions or a longer time period to get better-represented data.

## AI usage

### Label stress-testing

I asked Copilot in VS Code to generate seven comments that would be difficult to distinguish using my labels. One example was: “Always chill the dough at least 24 hours. I only did 12 because I was impatient and they spread all over the pan, so now I never skip it.” I used the examples to test whether my label definitions and decision rules gave a clear answer for comments mixing personal experience and advice. This exercise also highlighted that questions and uncertainty did not fit well under the original categories, so I added the `confusion` label. I reviewed the generated examples and assigned their labels using the main-purpose rule rather than treating the AI's labels as authoritative.

### Annotation assistance

I used Claude to pre-label about 20 comments and compared its suggestions with my own labels. The suggestions generally aligned with my decisions, which gave me a check on the consistency of my approach. I treated Claude's labels as suggestions, not ground truth, and retained responsibility for the final labels. **Disclosure:** AI assistance was used during annotation.

### Failure analysis

I shared the fine-tuned model's misclassified examples with Copilot in VS Code and asked it to suggest patterns. It highlighted possible keyword or wording shortcuts, the overlap between `changes`, `tips`, and `reviews`, and the limited evidence for `confusion`. I checked these suggestions by rereading the comments and applying my decision rules. I retained the surface-wording observation for the sarcastic warning and the advice-like review wording. I also kept the extra-flour example labeled as `changes`, since accidental alterations count under my label definition, and treated the model's `reviews` prediction as an error.

## Confidence calibration

I checked whether the model was more accurate on the examples where it was more confident. Of the two examples with confidence below 0.50, one was correct. Of the three examples with confidence from 0.50 to 0.74, two were correct. So, in this small sample, the higher-confidence predictions were a little more accurate.

This is only a rough check because I chose these five examples to show a mix of correct and incorrect predictions. It does not tell me whether confidence matches accuracy across all 31 test comments. To check that properly, I would need to compare the confidence and correctness of every test prediction. For now, I can only say the selected examples show a small difference, not that the model is well calibrated.

## Demo video
https://youtu.be/2h5pcMupD3g