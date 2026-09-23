# My Custom LLM Experiment

Replace the prompts below with your choices, actual outputs, and explanations.

Grading uses deliverable quality **4 points**, testing & evaluation **3 points**,
and working result **3 points**. Your model's eval percentage is not your grade.
Complete, valid eval evidence and a reasoned comparison matter; no minimum pass
rate or numerical improvement is required. Missing evidence earns less credit.

## My choices and prediction

I kept the defaults: CORPUS="classroom", TRAINING_STEPS=3000, LEARNING_RATE=0.001. It's the baseline run before adding more data, and these are the settings the assignment suggests to start with. 3000 steps is enough for the loss to move past its random start while still finishing quickly on CPU. 0.001 is the standard starting learning rate, not too big to be unstable, not too small to barely learn.

I expected the trained model's text to sound a bit more grammatical than the untrained one, even if still simple and repetitive. I expected the validation loss to drop early and then flatten out, since the corpus is small and repeats similar sentences. For word neighbors I checked "customer": before training its neighbors should look random, and after training I expected it to end up close to other similar words from the corpus, like shopper, buyer, or consumer.

For this first experiment I used only the classroom corpus, no extra files. All sources here are the notebook's own generated sentences, so there is no external permission needed and no PDF extraction to check.

## My run

**Experiment 1 (starter corpus)**
- Completed steps: 3000
- Elapsed time: 57.81 seconds
- Hardware: CPU (PyTorch 2.11.0+cpu)
- Parameter count: 111,872 (0.11M)
- Vocabulary size: 136
- Unique documents: 4,592 (train: 4,132, validation: 460)
- Training / held-out unknown-token rate: 0.00% / 0.00%

**Experiment 2 (expanded corpus)**
- Completed steps: 3000
- Elapsed time: 47.31 seconds
- Hardware: CPU
- Parameter count: 116,928 (0.11M)
- Vocabulary size: 215
- Unique documents: 4,649 (train: 4,184, validation: 465)
- New unique passages added: 57 (negation.txt: 40, opposites.txt: 18)
- Training / held-out unknown-token rate: 0.00% / 0.08%

The 509-type vocabulary cap was never reached in either run, so no word was dropped for being too rare. The split is done by short passage, not by whole source file, so this setup does not test how the model handles a completely unseen document, only unseen passages that may share wording with training passages.

## My evidence

![Training curves](evidence/experiment1/training_curves.svg)

### Loss values

| Experiment | Step | Training loss | Validation loss |
|---|---|---|---|
| 1 (starter) | 0 | 4.9263 | 4.9275 |
| 1 (starter) | 1500 | 0.6821 | 0.7182 |
| 1 (starter) | 3000 | 0.6783 | 0.7061 |
| 2 (expanded) | 1500 | 0.7142 | 0.8429 |
| 2 (expanded) | 3000 | 0.6937 | 0.8817 |

Full loss history: [history.json experiment 1](evidence/experiment1/history.json), [history.json experiment 2](evidence/experiment2/history.json)

Evaluation panel size: 20 training documents and 20 validation documents for each measurement, in both experiments.

### Samples over training

**Experiment 1**
Untrained (step 0): mostly random word order, for example "pear professor bond doctor course harvest team physician journey..."

Step 1500: "our school has a question about the new educator and lesson ."

Step 3000 (final): "the report about the nurse explains the health in detail ." / "the consumer compared the offering after checking the price ."

**Experiment 2**
Step 1500: "the new customer was mentioned in the order report yesterday ."

Step 3000 (final): "we learned about the new apple during a discussion of taste ."

Full sample files: [samples/ experiment 1](evidence/experiment1/samples/), [samples/ experiment 2](evidence/experiment2/samples/)

### Token and embedding (word: "customer")

Full data: [tokenization.json experiment 1](evidence/experiment1/tokenization.json), [inspection.json experiment 1](evidence/experiment1/inspection.json), [inspection.json experiment 2](evidence/experiment2/inspection.json)

**Experiment 1**
- Token ID: 28
- Vector before training (first values): -0.0576, -0.0048, 0.0426, 0.0193...
- Vector after training (first values): 0.0366, -0.0182, 0.1330, 0.1060...
- Saved parameter update: before -0.057592, gradient 0.000693, after -0.057602
- Next-word prediction before training (top 5): customer 0.016, bus 0.0107, educator 0.0104, us 0.0103, application 0.0101
- Next-word prediction after training (top 5): reviewed 0.1782, recommended 0.1712, ordered 0.1685, selected 0.1634, compared 0.1597

**Experiment 2**
- Token ID: 45
- Vector before training (first values): -0.0017, -0.0216, 0.0323, -0.0125...
- Vector after training (first values): -0.0010, -0.0416, 0.0869, -0.1676...
- Next-word prediction before training (top 5): customer 0.009, <BOS> 0.0072, explains 0.0068, noisy 0.0068, professor 0.0065
- Next-word prediction after training (top 5): reviewed 0.1765, ordered 0.1765, selected 0.1748, recommended 0.1636, compared 0.1481

Both experiments learned the same kind of pattern for "customer": before training the top prediction is barely above random guessing, after training it clearly favors verbs describing what a customer does (reviewed, ordered, selected, compared). The exact numbers differ slightly between experiments because the training data and random batches were not identical, but the pattern learned is the same.

### Nearest neighbors (embedding viewer)

Before training, "customer" has no meaningful neighbors, just random words like bus, educator, and helped, with low similarity scores (around 0.20). After training, its 3 closest neighbors become client, buyer, and shopper, all real business roles related to "customer", confirming the prediction made before training.

![Before training](evidence/embedding_viewer/before%20training.png)
![After training](evidence/embedding_viewer/after%20training.png)

### Temperature comparison

Full data: [temperature_comparison.json experiment 1](evidence/experiment1/temperature_comparison.json), [temperature_comparison.json experiment 2](evidence/experiment2/temperature_comparison.json)

Across temperatures 0.3, 0.8 and 1.2, the samples stay grammatical and close to the training templates. Lower temperature (0.3) gives more repeated, "safe" phrasing. Higher temperature (1.2) introduces slightly more varied word choices, but no temperature makes the model say anything outside the corpus's sentence patterns. This shows the model is not creating new ideas, just sampling more or less predictably from what it learned.

## My fixed language evals

Eval suite: [evals/language_evals.json](evals/language_evals.json)
Full results: [experiment 1 untrained](evidence/experiment1/language_evals/untrained/), [experiment 1 final](evidence/experiment1/language_evals/final/), [experiment 2 untrained](evidence/experiment2/language_evals/untrained/), [experiment 2 final](evidence/experiment2/language_evals/final/)

| Experiment | Stage | Correct / 48 | Scorable / 48 | Accuracy among scorable | Full results |
|---|---|---|---|---|---|
| Starter corpus | Untrained | 9 | 24 | 37.5% | [link](evidence/experiment1/language_evals/untrained/) |
| Starter corpus | Trained | 20 | 24 | 83.3% | [link](evidence/experiment1/language_evals/final/) |
| Expanded corpus | Untrained | 7 | 29 | 24.1% | [link](evidence/experiment2/language_evals/untrained/) |
| Expanded corpus | Trained | 25 | 29 | 86.2% | [link](evidence/experiment2/language_evals/final/) |

### By category

| Group | Experiment 1, untrained | Experiment 1, trained | Experiment 2, untrained | Experiment 2, trained |
|---|---|---|---|---|
| starter_patterns (16 cases) | 6/16 | 16/16 | 3/16 | 16/16 |
| starter_transfer (8 cases) | 3/8 | 4/8 | 2/8 | 6/8 |
| extend_corpus (24 cases) | 0/24, 0 scorable | 0/24, 0 scorable | 2/24, 5 scorable | 3/24, 5 scorable |

### What worked and what did not

starter_patterns went from mostly guessing (6/16) to perfect (16/16) in both experiments. This shows the model clearly learned the exact domain associations from the classroom corpus (customer with service, surgeon with patient, and so on), since these are the same sentence patterns repeated many times in training.

starter_transfer improved too (3/8 to 4/8 in experiment 1, 2/8 to 6/8 in experiment 2), but stayed far from perfect. These cases use the same words and associations but in new sentence order, so this shows the model learned some pattern beyond exact templates, but not a fully general one.

extend_corpus is the most important result. In experiment 1, all 24 cases were unscorable (0 scorable), because the classroom corpus never contained the words these tests need (for example "opposite", "hot", "cold", "tea", "milk"). Adding more training steps could not have fixed this: the vocabulary itself was missing.

For experiment 2, I chose the negation and opposites categories. I added two files, corpus/negation.txt and corpus/opposites.txt, with new sentences using the words these two categories need, written differently from the actual eval prompts. This raised scorable cases from 0 to 5 out of the 6 negation and opposites cases, and correct answers from 0 to 3. The other 18 cases (grammar, reference, sequence, spatial_relations, everyday_knowledge, categories_and_analogies) stayed unscorable, since I did not add vocabulary for those categories.

The one remaining unscorable case among negation/opposites is likely due to the random 90/10 train/validation split: with a small number of new sentences, some key word may have ended up only in the validation set and never reached training.

Eval separation: [eval_separation.json experiment 1](evidence/experiment1/eval_separation.json), [eval_separation.json experiment 2](evidence/experiment2/eval_separation.json). The notebook automatically removes any generated classroom sentence containing a reserved eval prefix before training, and rejects any imported file containing an exact eval prompt. This check only catches exact text matches, not paraphrases, so I also manually reviewed my negation.txt and opposites.txt files to confirm they use different sentence structure from the actual eval prompts, teaching the same words and idea without copying the test itself.

These evals are a public development benchmark I used to guide my corpus choices, not an unseen final test, since I looked at the results before deciding what to add.

## My chat interface

To chat with the trained model, open custom_llm.ipynb in Colab, run the notebook up to section 10 (or run the whole notebook with Run All), then edit the CHAT_PROMPT variable in section 10 and re-run that cell for each new message. A terminal alternative is included in the repository: `python chat.py --model path/to/model.pt`

This uses the model trained in experiment 2 (expanded corpus, 3000 steps). Each prompt starts a fresh context, there is no memory between messages, and chatting does not retrain the model or add anything to the corpus.

Full transcript: [chat_transcript.json](evidence/experiment2/chat_transcript.json)

**Interaction 1**
You: the coffee was
Model: learning , now it is discussed the hospital .
![the coffee was](evidence/experiment2/chat_screenshots/the%20coffee%20was.png)

This is a clear limitation. "Coffee" only appears in a few sentences in opposites.txt, so the model has very little data to learn what usually follows it. The output mixes fragments from stronger, more repeated patterns instead of forming a real sentence about coffee.

**Interaction 2**
You: the customer reviewed the
Model: brand after checking the price .
![the customer reviewed the](evidence/experiment2/chat_screenshots/the%20customer%20reviewed%20the.png)

This uses vocabulary from the original, heavily repeated classroom corpus, and the result is coherent and grammatical.

**Interaction 3**
You: the box is not
Model: open .
![the box is not open](evidence/experiment2/chat_screenshots/the%20box%20is%20not%20open.png)

This uses the negation.txt pattern added in experiment 2, and the model correctly continues it with a sensible opposite word, even outside the multiple-choice eval format.

The model only continues text, it does not answer questions. It knows only the 215 words in its vocabulary; any other word becomes <UNK>. It can only see the last 48 tokens of context, so longer prompts get cut to their most recent part.

## What I learned

**My corpus:** synthetic business sentences (customers, products, banks, hospitals, schools) built from fixed templates. It can teach word co-occurrence and order inside those templates, not general knowledge outside them. I held out 10% of documents as validation, to check loss on unseen sentences from the same templates, not on truly unseen topics.

**Token, ID, vector, embedding:** a token is one word or punctuation mark. Its ID is just a row number in the vocabulary, with no built-in meaning (customer = ID 28 in experiment 1). The vector is the 64 numbers in that row. The embedding is the full table of vectors, shape (136, 64) in experiment 1. The ID stays fixed, the vector changes as training updates it.

**Why this is a neural network:** each token's embedding, plus position, passes through attention and feed-forward layers to predict the next token. Loss measures how wrong that prediction was. PyTorch computes the gradient, how much each weight should change to lower the loss, and AdamW updates the weights using that gradient plus momentum and adaptive scaling. Real example for "customer": weight started at -0.057592, gradient 0.000693, after one step -0.057602.

**Attention:** lets each token combine information from earlier tokens, weighted by relevance. A causal mask blocks it from seeing future tokens, so prediction only uses what came before, same as generation.

**Probabilities to text:** the model gives a probability for every vocabulary word being next. Generation samples one word from these probabilities, then repeats. Temperature controls how sharp that sampling is: 0.3 sticks closer to the top choice, 1.2 picks less likely words more often. No weights change during generation.

**Did my prediction hold up:** mostly yes. Text went from random word lists to full sentences, as predicted. Loss dropped early then flattened in both experiments, though experiment 2's validation loss flattened higher than training loss, more than in experiment 1 (see next section). "Customer" ended up near shopper and buyer, exactly as predicted. Overall conclusion: the model learned strong, narrow patterns from repeated templates, not general language understanding, and low loss here says nothing about skills outside this corpus.

## One limitation and my next experiment

The main limitation is the extend_corpus vocabulary gap explained above: the starter corpus has no words for negation or opposites, so training steps alone cannot fix it, only new data can.

A second, smaller issue: in experiment 2, validation loss flattened at 0.88, clearly above training loss at 0.69 (a bigger gap than experiment 1's 0.71 vs 0.68). This suggests the model may be fitting the new negation/opposites sentences a bit too closely, since they are few and repeat similar structure.

**Next experiment:** add more varied sentences to negation.txt and opposites.txt, with different subjects and structures, not just more repeats. This should lower the validation gap and raise extend_corpus scores further.

## Reproduce and inspect

1. Open `custom_llm.ipynb` in Colab. Default CPU runtime is enough.
2. Experiment 1: run all cells with `CORPUS = "classroom"` and no files in `corpus/`.
3. Experiment 2: upload `negation.txt` and `opposites.txt` from this repo's `corpus/` folder into Colab's `corpus/` folder, then run all cells.
4. All evidence is in `evidence/`, split into `experiment1/` and `experiment2/`.
5. Chat interface: notebook section 10, or `python chat.py` from the terminal.

This repository is public and was checked to be visible while signed out before submitting.
