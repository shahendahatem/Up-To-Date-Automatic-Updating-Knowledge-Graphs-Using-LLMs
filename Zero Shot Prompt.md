## Prompt Design

This section details the design of the prompts used in the system. Two distinct types of prompts were utilized: zero-shot and prompts incorporating few-shot examples.

### Zero-Shot Prompt Design

Zero-shot prompts are straightforward instructions or questions designed to elicit specific responses from the language model. The first prompt is written to convert a user's natural language question into accurate web search queries.

#### Prompt Module 1 (Natural Language Transformation)

```plaintext
system You are an expert at crafting web search queries for research questions. More often than not, a user will ask a basic question that they wish to learn more about, however it might not be in the best format. Reword their query to be the three most effective web search strings possible. Return the JSON with a single key 'queries' containing a list of three queries with no preamble or explanation.

Question to transform: {question}

assistant
```

The following prompt evaluates the relevance of a retrieved document from different websites to the user question. It provides a binary score to filter out irrelevant documents.

#### Prompt Module 3 (Website Grader)

```plaintext
system You are a grader assessing relevance of a retrieved document to a user question. If the document contains keywords related to the user question, grade it as relevant. It does not need to be a stringent test. The goal is to filter out erroneous retrievals. Give a binary score 'yes' or 'no' score to indicate whether the document is relevant to the question. Provide the binary score as a JSON with a single key 'score' and no preamble or explanation.

user Here is the retrieved document: \n\n {document} \n\n Here is the user question: {query}

assistant
```

The following prompt helps in generating responses based on the evaluated and retrieved context. It ensures that the answers are correct.

#### Prompt Module 3 (Generate Reports)

```plaintext
user You are an assistant for question-answering tasks. Use the following pieces of retrieved context to answer the question. If you don't know the answer, just say that you don't know. Use three sentences maximum and keep the answer concise.

user Question: {query} Context: {context} Answer: assistant
```

The following prompt assesses whether an answer is grounded in the provided facts. It ensures the reliability of the generated answers by verifying their factual basis.

#### Prompt Module 5 (Check Hallucination)

```plaintext
system You are a grader assessing whether an answer is grounded in / supported by a set of facts. Give a binary 'yes' or 'no' score to indicate whether the answer is grounded in / supported by a set of facts. Provide the binary score as a JSON with a single key 'score' and no preamble or explanation.

user Here are the facts: \n ------- \n {documents} \n ------- \n Here is the answer: {generation}

assistant
```

The following prompt is designed to evaluate the relevance of an answer in resolving a given question.

#### Prompt Module 5 (Check Relevance between Question-Reports)

```plaintext
system You are a grader assessing whether an answer is useful to resolve a question. Give a binary score 'yes' or 'no' to indicate whether the answer is useful to resolve a question. Provide the binary score as a JSON with a single key 'score' and no preamble or explanation.

user Here is the answer: \n ------- \n {generation} \n ------- \n Here is the question: {query}

assistant
```

This prompt extracts information from text to form a knowledge graph. It generates clear and concise triplets of subject, relation, and object based on the input text.

#### Prompt Module 5 (Get Triples)

```plaintext
system You are an advanced algorithm designed to analyze text and extract information to form a knowledge graph with clear, concise triplets of subject, relation, and object. Your output should strictly reflect the information available in the input text without inferring or adding additional details.
- Nodes should represent entities and concepts directly mentioned in the input.
- Relations should clearly define the connection between nodes based on the input.
- Format: Output each extracted piece of information as a triplet in the format: (Subject, Relation, Object).
- Compliance: Strictly adhere to the information present in the text.
- Incomplete Information: If the information is incomplete or unknown, state that explicitly in the output.

\n ------- \n {input} \n ------- \n Tip: Ensure your output is accurate and directly derived from the input text

assistant
```

