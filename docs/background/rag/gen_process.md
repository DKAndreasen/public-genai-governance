# Response generation using large language models

The simplest part of a RAG system is the generation of the response.

When [information/context has been retrieved](./retrival_system.md), the system prompt together with the user question are combined and 
passed to the text generation API. 

Notice that the user question needs not be the actual message the user wrote to the system, it might represent a part 
of it and a more focused/lean version of it.

```mermaid
graph TD
    ri@{ shape: lean-r, label: "Retrieved information" }
    prompt@{ shape: doc, label: "System prompt" }
    input@{ shape: manual-input, label: "`User Question
    (possibly rewritten and simplified)`" }
    collect@{ shape: hex, label: "Collect into message struct" }
    caller@{ shape: process, label: "Call generation API" }
    genai@{ shape: subproc, label: "Text generation
    completion endpoint" }
    ui@{ shape: display, label: "User interface" }
    
    input --> collect
    ri --> collect
    prompt --> collect
    collect --> caller
    genai <--> caller
    caller --> ui
```

if this is just part of a loop to collect answers that should be combined into a complete response, then the response 
(and possibly request) needs to be passed on to a process/agent that collects information from all iterations. 

When calling the endpoint of the generation API, there are some constraints to be aware of:
- obviously different languages models have different levels of capabilities in different languages and in different 
  domains within a language.
- language models are trained with different maximal context lengths - and maybe only in certain languages.

  This sets an upper limit on the amount of context that can be passed to the generation API.
- Longer context means larger memory footprint, thus depending on the setup the context length might be reduced 
  compared to what the model could in principle handle
- [Research](https://arxiv.org/abs/2307.03172 "Old research paper: Lost in the Middle: How Language Models Use Long Contexts") 
  have shown that at least some generation models suffer from a "lost in the middle" phenomenon, where 
  information in the middle of a long the context is overlooked by the model. 

  Along the seem lines, other [research](https://arxiv.org/abs/2404.18655 "Work of Isabelle Augenstein") have shown a 
  tendency in (at least some) models to prefer information present at
  training time over information in the context.
