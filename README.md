# appliedAI
Individual project completed for the module Applied AI under NTU MSc Information Systems

This project experimented with a hybrid approach for identifying dataset mentions in scientific literature, combining rule-based methods, feature engineering, SciBERT embeddings with assistance from LLMs for verification of ambiguous cases. The automatic detection of dataset references is crucial for tracking data usage, enhancing reproducibility, and properly attributing dataset creators. The solution is a multi-stage pipeline that identifies potential dataset mentions through a combination of named entity recognition (NER), pattern matching or pattern-based extraction, classification of candidates using contextual features, neural embeddings, and large language models to verify ambiguous cases via queries. The current approach implements batch processing and checkpointing for resilience, allowing processing of large document collections. 

This proposed AI solution was developed and evaluated in response to the "Show Us the Data" competition by the Coleridge initiative (https://www.kaggle.com/c/coleridgeinitiative-show-us-the-data), which tasked participants with developing models to automatically identify dataset mentions in academic papers. In response to the challenge, this project proposes a solution which employs a combination of natural language processing techniques, neural embeddings, and machine learning models, treating dataset mention detection as a hybrid pipeline consisting of:
  1.	Candidate mention extraction using NER and pattern matching
  2.	Named Entity Recognition (NER) through spaCy
  3.	Feature engineering to capture relevant characteristics of mentions and for further contextual analysis 
  4.	Neural embedding-based classification
  5.	Large language model verification for ambiguous cases

The development followed an iterative process: 
1.	First load the documents and ensure a basic extraction pipeline with rule-based candidate identification. 
Integrate NER through spaCy for improved candidate generation
2.	Add feature engineering for better candidate filtering
3.	Integrate SciBERT for embedding-based classification
4.	Add LLM verification for ambiguous cases
5.	Implement batch processing and checkpointing for robustness
6.	Evaluate and refine the system based on performance metrics

In particular, for ambiguous cases where the classifier's confidence was between 0.4 and 0.6, an LLM-based verification step was implemented using OpenAI's GPT-4o-mini. This enhancement step improved precision by leveraging the language model's understanding of context and dataset mention to make nuanced judgements about ambiguous candidates.
During this step, the LLM is used to verify whether a given candidate text indeed refers to a dataset or not, based on the context in which it appears. For each ambiguous candidate, the model retrieves the surrounding context from the document. This context typically includes the 100 characters before and after the candidate mention, which helps provide additional clues to the model about whether the mention refers to a dataset. Once the context and the candidate text are extracted, a prompt is formulated to query the LLM. The prompt is structured in a way that provides the LLM with both the surrounding context and the candidate text to evaluate. For example:
_Context from a scientific paper: "..." (this is the extracted context around the candidate mention)
Question: Does the phrase "National Health Survey" refer to a research dataset or data resource?
Answer "Yes" or "No" and provide a confidence between 0 and 1._

The LLM processes the prompt and generates a response. This way, it determines whether the phrase in question refers to a research dataset or data resource based on the surrounding context and its training. The LLM is expected to return an answer ("Yes" or "No") and a confidence score indicating how confident it is in its judgment.
The response from the LLM then paseed to parsed to Dataset Verification, checking if the answer contains "Yes" to confirm that the candidate text indeed refers to a dataset. The confidence score is then extracted from the LLM's response. This score represents how confident the model is in its decision (on a scale of 0 to 1). The system then calculates a final confidence score for each candidate by averaging the embedding-based confidence (from the classifier) and the LLM's confidence score. This combined confidence score is used to make the final decision on whether the candidate is a dataset mention or not.
By incorporating the LLM at this stage, the system can increase its accuracy by leveraging the broader contextual understanding of the language model, helping to improvement the treatment of ambiguous cases.

The top_k parameter (set to 3) limits the number of API calls per document, managing costs and time usage while focusing on the most promising candidates. The prompt design is intentionally simple and direct, asking specifically about dataset likelihood.
