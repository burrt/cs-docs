---
description: Intro notes from https://www.youtube.com/watch?v=7xTGNNLPyMI
---

# AI Notes

## LLM

A **large language model (LLM)** is a type of artificial intelligence program trained on massive amounts of text data to understand, summarize, translate, and generate human-like language

### Steps for building an LLM

#### Pretraining

1. initial data set e.g. download the internet (HuggingFace)
2. tokenisation
   1. reduces the data (e.g. text) down into a unique set of tokens / symbols
   2. essentially compresses this down to reduce the length and&#x20;
3. neural network training
   1. takes a set of sequence (set of tokens) and attempts to predict the next token
   2. feedback loop to "train" into optimal prediction / weights based on a "context" - limited set of tokens
   3. these weights then become the template for an output that comforms to what is seen in the training set and is based on a mathematical expression?
4. inference - generating new data from the model
   1. model at this point is "fixed" at this point
   2. observes the output and validates it against what we expect (e.g. in training)
   3. can easily get to a cascading "unexpected" tokens
   4. final output is a "base model" - it essentially a token similator and cannot be an interactive conversation

Output: base model

It essentially will retrieve from its "compressed" version of the data it's been trainined on e.g. the internet. It will act like an autocomplete and basically "dreams" of the internet. To trick it to behave like an assistant is to construct a prompt that consists of a conversation between a human and assistant - it will then fill in the response!

#### Post training

Turning a base model into an assistant.

Trains it on human-labeled conversations that we validate and want the assistant to behave. This dataset will be much smaller and will be encoded as tokens so that the LLM can be trainined on. This starts to give the assistant a personality.&#x20;

We are now using existing LLMs to create post-training datasets e.g. UltraChat.&#x20;

Output: the model then becomes a SFT model.

#### Reinforcement learning

Technically it can fall under post-training but can be considered as a whole separate field. It essentially is practicing problems and fine tuning to the best possible outputs. Gives the model an opportunity to practice, trial and error and eliminate suboptimal answers. It's more involved than just that as it leans into "thinking" mode of the problem to verify itself on the solution, and then works to format the solution to a human.

Output: the model then becomes a RL model.

#### Reinforcement Learning from Human Feedback

RLHF is not RL!

Some questions or problems are not quantitatively "scoreable" e.g. arts / writing. There are also diminishing returns as there are only so much data that it has been trained on and the feedback loop degrades with "poorer" quality data. Humans are able to bias the scores / weights of the model's weights and it can slowly steer to an output that we might expect.

### Misc.

#### Hallucinations

1. we can create a "judge" LLM to create new conversations that will allow the LLM to identify patterns of when it does not know the answer, it can say it doesn't know.&#x20;
2. we can allow the model to search
   1. model will substitute the search response into the context window&#x20;
3. the model can then know when to search based on the post-training conversations of when it needs to search and how to search (from pre-training)

* knowledge in the parameters = vague recollection
* knowledge in the tokens of the context window = working memory



#### Cost / Unit of work of a token

This is in the context that the cost of work / effort is best spread across many passes of the model / "transformer". That's a token can only consume a finite amount of work and they are all treated as "equal". So that is why it is very difficult for a model to do basic math questions as we expect a singular answer - but the model best operates when the amount of work is spread across multiple tokens, not to a single token or point.

This infleunces the structure of the the output and how some are better and more accurate than others in the context of say math problems. This affects how "labellers" and post-training is important to understand how a model works and how the human interprets solutions.
