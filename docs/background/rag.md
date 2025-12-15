# RAG overview

The very high level overview (that makes something not at all simple look simple)
```mermaid
graph TD;
    data@{ shape: lean-r, label: "Data" }
    ingest[["Ingestion"]]
    db[(Database)]
    data-->ingest
    ingest-->db
    
    input@{ shape: manual-input, label: "User Question" }
    agent[["Searching agent/\nretrival system"]]
    input-->agent
    agent<-->db
    
    genai[["generation process"]]
    agent-->genai
```

In the following sections we will go into detail on each of the processes, 
including the important preparation of input data, that are just presented 
assumed present in the above diagram.

- [Datapreparation](rag/datapreparation.md)
- [Ingestion](rag/ingestion.md)
- [Searching agent/retrival system](rag/retrival_system.md)
- [Generation process](rag/gen_process.md)

In addition to this there could/should also be an answer verification process
with a final verification of the provided answer, that it is truthfull according 
to the retrieved information.