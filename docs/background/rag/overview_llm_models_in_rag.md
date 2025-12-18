# Overview of various LLM models used in RAG

There are many different kinds of large language models (LLMs) and they can be different from each other in different 
ways. 

Some are built for different purposes and cannot be used interchangeably, but subgroups of LLMs can actually be used 
interchangeably, some of them just perform considerably worse (or better) at a specific task compared to another.

Thus, this might cause a bit of confusion that this overview will try to clarify. The name LLM covers a wide range of 
models. The first "L" (large) does not really mean anything, except maybe that the model is built using techniques from 
after 2017 ("transformer"-models) and not other statistical language models. Although in the last couple of years the 
term "SLM" (small language model) has become popular as well, these models are still transformer-based and in some
subgroups of SLM/LLM they can be used interchangeably with some LLMs 🤯.
The rest of the name, "LM" (language model) simply refers to the fact that the model handles language in some form, 
often text or maybe audio.

## Generative LLM

what in layman terms is a _LLM_.
The purpose that the LLM is trained for is to predict the next word (token) given a text (sequence of tokens). 
This prediction is continued until a stop signal (stop token) is predicted (or some other stop condition is 
encountered). These LLMs come in two main classes, continuation models and intruct models. 
Continuations models are the base class and will just predict the most probable next token. 
Instruct models are continuation models, that have been finetuned to predict the next token in a response to an 
instruction. That is, an instruct model expects the given sequence of tokens to constitute an instruction and possibly 
first part of the response, that the model is incrementally generating.

Continuation and instruct models are in principle interchangable, but in practice their endpoints are designed to
expect different formats to avoid unintended use. But, in principle a continuation model could be set to continue 
this sequence "You are a helpful assistant. Tell me about Mont Matré" but maybe the responses from the model would go
like ",", "the", "Eiffel", "Tower", "and", "other", "important", "sights", "in", "Paris", ".", "Jane", ":", "I", "can",
"recommend" and so on. Instead of responding with an answer.

Examples of generative LLMs range from Llama3 8B, GPT5, Mistral Small 24B and many many more.

## Embedding models

Embedding models transform/generate/represent or simply _embed_ tokens (words) into vectors. 
Vectors are sequences of numbers (usually in the order of 200 to 1000) that represent the meaning of the word in the 
context it appears in.

Embedding models are actually a wider category than just LLM (or transformer) based models. 
Word or sentence embeddings can be constructed in a wide range of ways. 
If the situation calls for a specification then _dense_ embeddings or _dense embedding models_ usually refers to 
transformer based embedding models, although some other embedding methods can also produce _dense vectors_.
This is opposed to _sparse embeddings_ that other non-transformer based methods use to represent words or sentences.
Both sparse and dense embeddings can be used in RAG solutions (in the ingestion/retrieval phases).
Transformer based embedding models are also sometimes referred to as SLMs (small language models) as they often consist 
of much fewer parameters compared to the generative LLMs and thus take up a lot less space in the computer memory.

The purpose of embedding words into a _vector space_ is to be able to quantify which words (tokens) or sentences are
more similar to each other than the rest. It opens the possibility to quantify statements like "king" and "queen" are 
different to each other like "man" and "woman" are.
Very important for RAG, is the fact that the _embedding space_ (aka _vector space_) makes it possible to index text 
pieces in a database (denoted a vector database). Given a new piece of text, it is then possible (using the embedding 
model) to calculate its vector representation. That representation can then be compared to the stored "index" of the 
vector database and it is, thus, possible to find the closest vectors (stored text pieces). 
The idea is then, that text pieces are closest, that is most similar, to the given text. 
"Similar" is here semantically similar, and the semantic meaning of words in a text are what an embedding model is 
trained to represent.

Most often a very strict naming convention will probably distinguish between an encoder model, that encodes the 
semantic meaning of a single token (word) of a sentence. An embedding model is then a model, that encodes the "full" 
semantic meaning of a piece of text. The encoder model is then seens as a base model that is extended into an embedding
model by adding extra layers and training them to distinguish between vectors representing texts that are similar and 
vectors representing texts that are different.

Embedding models are trained to be able to embed pieces of text (called chunks) of length up to 200, 500, 1000 or maybe
even more. The longer the text, though, the more likely it is that the text concerns simantically distinct topics, 
that might make it hard to represent all of it in one vector. 

Examples of embedding models are 
OpenAI's [text-embedding-3-large](https://platform.openai.com/docs/api-reference/embeddings),
intfloat's [multilingual-e5-large](https://huggingface.co/intfloat/multilingual-e5-large), 
Qwen's [Qwen3-Embedding-4B](https://huggingface.co/Qwen/Qwen3-Embedding-4B).

The [Massive Text Embedding Benchmark (MTEB)](https://embeddings-benchmark.github.io/mteb/) project maintains an 
[overview](https://huggingface.co/spaces/mteb/leaderboard)
over a large number of embedding models. To see a list of relevant "danish" models chose "RTEB Multilingual" in the menu
on the left side of the dashboard under the heading "Retrieval".

## Cross-encoder models

Cross-encorder models are similar to embedding models in that they are built from base token encoder models. 
But instead of outputting a vector representating the simantical meaning of a text piece, a cross-encoder
takes two pieces of text and outputs a number. The cross-encoder is then trained to output a number close to one if
the two texts are similar and a number closer to zero if they different.

Like with the embedding models this gives the possiblity to rank a list of texts by how similar they are to a given 
reference text.

This, though, is much more computationally heavy than calculating the distance between vectors and therefore in RAG 
systems the cross encoder is most often used after an embedding models have been used to select e.g. the 50 most similar
(in the embedding model sense) pieces of text from a database of e.g. 10.000 texts. The cross encoder is then being run 
with each of the preselected 50 texts and the reference text. The results from the cross encoder is then used to re-rank
the texts, so only the five most similar (in the cross-encoder sense) texts are passed on for further processing in the 
system.

The principle advantage of cross-encoders compared to embedding models is that the simantical meaning of a piece of text
is not squeezed into a single vector (representing the entire text) instead all the token-vectors from both candidate 
and reference text are processed by the cross-encoder.

Both cross encoder models and embedding models suffers from the fact that you have to be very aware what kind of 
semantical similarity the models are trained for. E.g. are questions about toys and question about foods more similar to
each other, because they are questions or to tweets about childrens play (tweets might have distinct simantics) or to 
recipes, which again have a certain form and way of phrasing. All this can be accounted for during training, but if you 
have not trained the model yourself, you better be aware and test it. 

Examples of cross-encoder models, that can be used for re-ranking 
BAAI's (Beijing Academy of Artificial Intelligence) [bge-reranker-v2-m3](https://huggingface.co/BAAI/bge-reranker-v2-m3)
and KennethTM's [MiniLM-L6-danish-reranker-v2](https://huggingface.co/KennethTM/MiniLM-L6-danish-reranker-v2).

## Classification models

## Translation models
