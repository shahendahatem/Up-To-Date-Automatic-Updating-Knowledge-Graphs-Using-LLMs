# Up-To-Date-Automatic-Updating-Knowledge-Graphs-Using-LLMs

Maintaining up-to-date knowledge graphs (KGs) is essential for enhancing the accuracy and relevance of artificial intelligence
(AI) applications, especially with sensitive domains. Yet, major KGs are either manually maintained (e.g., Wikidata) or infrequently
rebuilt (e.g., DBpedia & YAGO). Thus, they contain many outdated facts. The rise of Large Language Models (LLMs) reasoning
and Augmented Retrieval Generation approaches (RAG) gives KGs an interface to other trusted sources. This paper introduces
a methodology utilizing Large Language Models (LLMs) to validate and update KG facts automatically. In particular, we utilize
LLM reasoning capabilities to determine potentially outdated facts. After that, we use RAG techniques to generate an accurate
fix for the fact. Experimental results on several LLMs and real-world datasets demonstrate the ability of our approach to propose
accurate fixes. In addition, our experiments highlight the efficacy of few-shot prompts over zero-shot prompts

## Proposed System

![Proposed System](PSysup3.svg)

Fig. 1: Illustration of the proposed system for automatic updating of the knowledge graph based on Retrieval-Augmented Generation (RAG) with large language models (LLMs).
