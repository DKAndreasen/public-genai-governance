# Task: Improve RAG solution

This document describes the current state and the initial thoughts behind a work-task to improve the RAG capabilities
of os2AI.  

_All statements below are up for discussions, they are based on current best understanding of both technical aspects 
and Steering Commitee preferences_

## Aim

Have a general RAG component for os2AI that enables trained (but non-specialist) municipal workers to create a custom RAG solution.

- _trained_ in the sense that the municipal worker have participated one-two workshops or are being trained by a co-worker.
- The knowledge base for this basic functionality is a handheld copy of information with the RAG creator being responsible
  for it being up-to-date

### Perceived workflow

1 A trained end-user, "creator", uploads a number of documents through a user interface to a "knowledge base". 
  The creator names the knowledge base and describes its intent.
  The creator shared this "RAG solution" with a number of usergroups.  
  End-users from the groups can now chat with a chatbot/language-model/genai that answers based on retrieved content 
  from the knowledge base, if any content is deemed relevant.
2 An end-user uploads one or more documents within a chat-thread. The chatbot now answers based on retrieved content
  from the knowledge base automatically created due to the upload.

## Prerequisites

- Source code must be open source licened according to [INSERT]()
- The code arcitecturel design must be as modular as sensible
  - Current principal/general idea (that is, dont put too much emphasis on the drawing - its just a quick sketch):
    ```mermaid
    flowchart TB
        UI -->|messages| rag[RAG Logic] -->|response| UI
        UI -->|docs| docStore@{ shape: db, label: "Document (and metadata) store" } <--> ingest[Ingestion logic] --> vdb@{ shape: db, label: "Vectorstore"}
        digest@{ shape: procs, label: "Digestion pipes" } <--> ingest
        ingest <--> emb[embedding engine]
        rag <--> llm[LLM inference]
    
        classDef fixed fill:#f6f6ff,stroke:#ffffff
        class UI,llm fixed
    ```
    Greyed out component are fixed/not part of this task. Others are envisioned as seperate containers.
- The UI is openwebUI. The RAG component/solution must integrate with openwebUI
  - response should follow the openAI-compatible [openResponses standard](https://www.openresponses.org/). This is meant
    to ensure that the rag solution can be integrated with OpenWebUI as a "model"
  - Document uploads: Needs to be a sensible endpoint, that it would make sense to integrate with any UI.
- LLM inference is currently rerouted through LiteLLM as an openAI compatible endpoint.
  - Currently the solution revolves around [Mistral Small 3.2](https://huggingface.co/mistralai/Mistral-Small-3.2-24B-Instruct-2506)  
    This will probably change at some point
  - Language models potentially used to drive the RAG logic should be "Open source" at very least to [MOT/MOF](https://isitopen.ai/) level 3 ("Open weights model").
    If a sufficiently able coding or tool-using model living up to [OSAID1.0](https://opensource.org/ai/) could be found
    this would be a very nice bonus, but it is not at all a focus area. Qwen models or OSS120B will do, as long as they
    can be run (in quantized form) on available single gpus (H200 or blackwell 96GB).
  - Open weights models primarily developed by European initiatives/companies are also prefered above the Chinees or American one.
- The more community backed standard components that can be used, the better.

## Current setup

_This is the baseline. A solution need not be best, but just significantly better_

Ingestion and RAG logic is the built-in functionality of openwebUI. 
Document digestion is done in a Tika instance in a seperate container using the xhtml endpoint (as far as I remember).
Chunking is done per document to max 500 characters (or tokens, it should have been tokens) with 100 character/token overlap. 
Dense semantic embedding is done with [intfloat/multilingual-e5-large](https://huggingface.co/intfloat/multilingual-e5-large).
Additionally, I think, BM25 sparse embeddings are used as well, maybe orchestrated on the openwebUI end, maybe delegated
to the vectorstore, which in the current setup is a Qdrant instance in a seperate container.
Retrival logic is also based on openwebUI standards, but I am not sure how it is configed. Either retrival is done from 
LLM generated questions based on last 1-5 messages or maybe it is just retrieved using last message. 
Anyway a more modern setup will hopefully improve the system.

## Initial ideas/thougths 

Replace the openwebUI bound RAG-logic with a seperate RAG component where an agent is given access retrieval tools or 
functions (depending on it being a coding agent or tool-using agent).
Use a planning/action (ReAct) approach let the agent decide if the retrieved content is sufficient and then answer the
question.
Build it using Haystack (modularity and community) or smolagents (simplicity)

- Another framework can be chosen, but in our experience LangChain is overengineered, Llamaindex are too focused on science