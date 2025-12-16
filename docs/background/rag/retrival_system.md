# Information retrieval system

Together with the [data ingestion pipeline](./ingestion.md) the retrival system constitute the two chambers of the heart 
of a RAG system.

The retrival system is responsible for retrieving relevant context from the knowledge base, that the ingestion system
has constructed.

The retrival system can be very simple, like:
```mermaid
---
title: Simplest retrieval system
---
graph TD;
    db[(Database)]
    input@{ shape: manual-input, label: "User message" }
    embed["Create embedding"]
    retriever["`Retrieve _k_ most similar entries`"]
    output@{ shape: in-out, label: "context struct with parts of 
    documents and possibly metadata" }
    input --> embed
    embed --> retriever
    retriever <==> db
    retriever --> output
```

where the embedding could be a semantic embedding (aka dense embedding), a sparse BM25 embedding or something along 
those lines and were _k_ is on the order of 3-5.

This can be extended by a re-ranking process. Which can be done using a dedicated cross-encoder model 
```mermaid
---
title: Simple retrieval system with re-ranking
---
graph TD;
    db[(Database)]
    input@{ shape: manual-input, label: "User message" }
    embed["Create embedding"]
    retriever["`Retrieve _k_ ~ 50 most similar entries`"]
    output@{ shape: in-out, label: "context struct with parts of 
    documents and possibly metadata" }
    input --> embed
    input ---> crossEncoder
    embed --> retriever
    retriever <==> db
    retriever --> re-ranker
    subgraph re-ranker
        loopCondition@{ shape: decision, label: "k #lt k_max"}
        crossEncoder["`Calculate cross-encoder score between
        k'th retrieved document and user message`"]
        storage@{ shape: das, label: "Store cross-encoder scores"}
        sorter["Sort retrieved documents by cross-encoder score"]
        loopCondition -- yes --> crossEncoder
        
        crossEncoder --> loopCondition
        loopCondition --- no ---> sorter
        crossEncoder -.-> storage
        storage -.-> sorter
    end
    sorter --> output
```
or by asking a
generative model to sort (portions of) the retrieved documents or just indicate whether a document is relevant or not.
#TODO examples of prompts in jinja2


_Note_: For english language solutions or other high ressource languages another retrival issue becomes relatively 
relevant, that is ow to search a 
vector index space. This is usually done using approximate algorithms to estimate cosine-similarities, but can also 
(in principle) be brute force calculated. 
Thus, it is a fact that the approximate algorithms add an error on the 
retrieval process, but especially in low-resource languages like Danish, it is a much bigger issue how the embedding is
done and how the document chunking/embedding context is constructed. Thus, that is the focus for this documentation.
The aspect of vector space search will not be covered anymore here.

## Training or generel semantic embedding models

The semantic embedding models can be choosen from off-the-shelf models or trained/fine-tuned.
Off the shelf models can be proportary models like openAIs embedding endpoint or locally hosted open weights models. 

#TODO add examples