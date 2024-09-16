# Automatic Prompt Engineering for classification

Given only (text -> label), this notebook generates and optimizes system and user prompts.

This is how classification is intended to be done.
- (system prompt, user prompt prefix + text + user prompt suffix) -Haiku-> bot response -extract-> label

The notebook will produce
- the system prompt
- the user prompt prefix
- the user prompt suffix

You can simply run this notebook with just
- an Anthropic API key and an OpenAI API key

If you want to change the classification task, you will need to
- provide a dataset (text -> label)

This is how prompt tuning is done
- Sample from the full dataset.
- Haiku takes in (system prompt, user prompt prefix + text + user prompt suffix) and produces model_response
- Extract the label from the model_response.
- Sample from the mistakes and the correct results.
- o1-mini summarizes the mistakes and update the prompts (model parameters).
- Repeat.

You will need to have these Python modules installed
- pandas
- scikit-learn
- anthropic
- openai

This notebook will also produce
- The [classification](https://tonghuikang.github.io/automatic-prompt-engineer/html_output/iteration-classification-002.html) (or just the [mistakes](https://tonghuikang.github.io/automatic-prompt-engineer/html_output/iteration-classification-002-diff.html)) at each iteration of the prompt.
- The [history](https://tonghuikang.github.io/automatic-prompt-engineer/html_output/prompt-history-classification.html) of the prompt and relevant metrics.
- (These will be saved locally as HTML files)


# References

I took inspiration from these works.

- [DSPy](https://dspy-docs.vercel.app/docs/building-blocks/solving_your_task) for describing how tuning a prompt engineering pipeline mirrors that tuning the parameters of a neural network.
- [Matt Shumer](https://twitter.com/mattshumer_/status/1770942240191373770) for showing that Opus is a very good prompt engineer, and Haiku is sufficiently good at following instructions.


# Design Decisions

- I require the LLM to produce the reasoning.
  Having the reasoning provides visibility to the thought process, which helps with improving the prompt.
- I minimized the amount of required packages.
  As of commit, you will only need to install `scikit-learn`, `pandas`, and `anthropic` Python libraries.
  If you want to use this in a restricted environment, there are fewer packages to audit.
- I chose visibility over abstraction.
  There is only one Python notebook with no helper Python functions.
  You can easily find out where to edit the individual functions.


# Exercises

### Run this notebook

Just add your with your Anthropic API key and you should be able to run `classification.ipynb`.

You will need to clone the repository for a copy of `qiqc_truncated.csv`.

It will cost approximately 40 cents per iteration, and `NUM_ITERATIONS` is 5.

You may need to change `NUM_PARALLEL_FORWARD_PASS_API_CALLS` depending on the rate limit your API key can afford.


### Change the dataset

As you can see, the labeling of whether a question is insincere is inconsistent.

There are some wrong labels in the dataset. You can either change the labels or exclude them from the dataset.

I recommend initializing `dataset` in the notebook with probably a copy of 50 samples where you can easily change the label, or comment out the sample.


### Change the sampling method

For the forward pass, we currently sample 100 positive and negative samples.
For the mistake summary, we sample 10 false positives and false negatives, 5 true positives, and 5 true negatives.
Otherwise, the sample is chosen totally at random.

We can improve the sampling method so that the model learns better.

For example, you can
- retain the wrong samples for the next iteration (or more) to make sure it is judged correctly
- inject canonical examples for every forward pass or mistake summary that you want to absolutely get correct


### Add notes for the classifier

Instead of the dataset being just (sample, label), we can have (sample, label, note) instead.

In the note, you note the reason why the sample is classified in a certain way.
You do not need to label every data with a note.

The note can be used as an input to gradient calculation and parameter updates, so that the sample is classified for the correct reasons.


### Get o1 to put words in Claude's mouth

Claude allows you to specify the prefix of the response (see [Putting words in Claude's mouth](https://docs.anthropic.com/claude/reference/messages-examples#putting-words-in-claudes-mouth))

Besides `system_prompt`, `user_prompt_prefix`, and `user_prompt_suffix`, you can also add `bot_reply_prefix` as a field that o1 should produce.

You will need to update the `user_message` in `update_model_parameters`.
You also need to describe in `PROMPT_UPDATE_SYSTEM_PROMPT` what `bot_reply_prefix` is for.


### Try a different classification task

You can try out a different dataset.

Another dataset I recommend is the [Quora Question Pairs dataset](https://www.kaggle.com/c/quora-question-pairs/data).

You will need to need to
- Change how the dataset is being loaded (since each sample is now a pair of questions, you need to change the delimiters)
- Change `predict_from_final_layer` to match a different string
- Change `PROMPT_UPDATE_SYSTEM_PROMPT` to describe what is being matched


### Try a non-classification task

You are either wrong or correct at a classification task.
For non-classification tasks, it is more difficult to evaluate how good your output is.
You need to think of how to evaluate whether the model is making a mistake, and how to update the prompts.
I think it is still useful to keep most of the structure of the notebook.
