# Automatic Prompt Engineer

This repository contains a notebook that [generates and optimizes system and user prompts](https://tonghuikang.github.io/automatic-prompt-engineer/html_output/prompt-history-classification.html) for classification purposes.

This is how classification is intended to be done.
- (system prompt, user prompt prefix + text + user prompt suffix) -Haiku-> bot response -function-> label
- The function will be defined by you (which could be just a string match)

The notebook will produce
- the system prompt
- the user prompt prefix
- the user prompt suffix

You can simply run this notebook with just
- an Anthropic API key

If you want to change the classification task, you will need to
- provide a dataset (text -> label)
- define the function bot_response -> label
- description for Opus on what instructions Haiku should follow

This is how prompt tuning is done
- Sample from the full dataset.
- Haiku takes in (system prompt, user prompt prefix + text + user prompt suffix) and produces bot_response.
- The function takes in bot_response and produces the label. The (text -> label) process is analogous to the forward pass.
- Sample from the mistakes.
- Opus takes in the mistakes and summarizes the mistakes (gradient).
- Opus takes in the mistake summary (gradient) and the current prompts (model parameters) updates the prompts.
- Repeat.

This notebook will also produce
- The [classification](https://tonghuikang.github.io/automatic-prompt-engineer/html_output/iteration-classification-002.html) (or just the [mistakes](https://tonghuikang.github.io/automatic-prompt-engineer/html_output/iteration-classification-002-diff.html)) at each iteration of the prompt.
- The [history](https://tonghuikang.github.io/automatic-prompt-engineer/html_output/prompt-history-classification.html) of the prompt and relevant metrics.
- (These will be saved locally as html files)


# References

I took inspiration from these resources.

- [DSPy](https://dspy-docs.vercel.app/docs/building-blocks/solving_your_task) for describing how tuning a prompt engineering pipeline mirrors that tuning the parameters of a neural network.
- [Matt Shumer](https://twitter.com/mattshumer_/status/1770942240191373770) for showing that Opus is a very good prompt engineer, and Haiku is good at following instructions.


# Design Decisions

- I require the LLM to produce the reasoning, and I have a separate function to extract the predicted label.
  Having the reasoning provides visibility to the thought process, which helps with improving the prompt.
- I minimized the packages that you will need to install.
  As of commit, you will only need to install `pandas` and `anthropic` Python libraries.
- I maximized the visibility into the workflows in the abstraction-visibility tradeoff.
  There is only one Python notebook with no helper Python functions.
  You can easily edit the individual function to edit how prompt tuning is done.
