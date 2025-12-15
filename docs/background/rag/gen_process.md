# Response generation using large language models

The simplest part of a RAG system is the generation of the response.

When 

```mermaid
graph TD;
    ri@{ shape: lean-r, label: "Retrieved information" }
    prompt@{ shape: doc, label: "System prompt" }
    input@{ shape: manual-input, label: "User Question\n(possibly rewritten and simplified)" }
    genai@{ shape: subproc, label: "Text generation\ncompletion endpoint" }
    ui@{ shape: display, label: "User interface" }
    input->genai
    ri->genai
    prompt->genai
    genai->ui
```

