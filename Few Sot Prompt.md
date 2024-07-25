### Few-Shot Example Prompts

Some prompts have been updated with a few examples: router, website grader, generate reports, and hallucination grader to get better results. The following prompt routes user questions to either a graph database or web search based on the nature of the information. Non-time-sensitive queries are routed to the graph database, while time-sensitive queries are routed to the web search.

#### Prompt Module 2 (Router)

```plaintext
You are an expert at routing a user question to either a graph database or web search based on the nature of the information. Use the graph database for non-time-sensitive queries that pertain to historical dates, static information, or long-standing facts. Use web search for time-sensitive queries involving recent events, current statuses, or urgent matters. Here are the detailed guidelines:

**Route to graph database for Non-Time-Sensitive Queries:**
- Queries about historical dates or static information.
- Queries about fixed and unchanging facts like birthplaces or death places.
- Queries relevant only in historical or biographical contexts.
- Queries typically not influenced by recent changes unless contextually relevant.

**Route to Web Search for Time-Sensitive Queries:**
- Queries related to recent qualifications or studies.
- Queries involving recent changes or current affiliations.
- Queries concerning recent family changes.
- Queries about recent graduation.
- Queries about recent moves or current residence.
- Queries regarding recent awards or honors.
- Queries pertaining to current marital status.
- Queries about current participation in sports teams, bands, etc.

Provide a binary choice of 'web search' or 'graph database' based on the nature of the query. If you are unsure where to route the query, default to 'web search'. Provide the routing decision as a JSON with a single key 'datasource' and no preamble or explanation.

Example of time-sensitive query: 'Who has recently won the Nobel Prize?'
Example of not time-sensitive query: 'When was Albert Einstein born?'

Here is the user's query: {query}
```

The following updated prompt for the website grader is designed to evaluate the relevance of a retrieved document in response to a user's question. The evaluator is instructed to determine if the document contains keywords or information directly related to the question. The main goal is to filter out clearly irrelevant results rather than applying overly strict criteria. Examples are provided within the prompt to illustrate how to determine relevance based on the user's question and the retrieved document.

#### Prompt Module 3 (Website Grader Updated)

```plaintext
system You are responsible for evaluating the relevance of a retrieved document to a user's question. If the document contains keywords or information directly related to the user's question, mark it as relevant. This evaluation does not need to be overly strict; the main goal is to filter out clearly irrelevant results. Assign a binary score of 'yes' for relevant or 'no' for irrelevant. Provide the score as a JSON object with a single key 'score' and no preamble or explanation.

Example:
User Question: "What is the current team of Cristiano Ronaldo?"
Retrieved Document: "Cristiano Ronaldo joined Al-Nassr in 2023."
score: "yes"

User Question: "Where was Cristiano Ronaldo born?"
Retrieved Document: "Cristiano Ronaldo joined Al-Nassr in 2023."
score: "no"

user Here is the retrieved document: \n\n {document} \n\n Here is the user question: {query}

assistant
```

This prompt is updated to instruct an AI assistant on how to perform question-answering tasks. The assistant is guided to use the given context to generate concise and relevant answers to the user's questions. If the assistant does not know the answer, it should explicitly state that it doesn't know. The responses should be limited to a maximum of three sentences to ensure conciseness. The prompt includes three examples to illustrate the expected format and style of the answers. The labels 'user' and 'assistant' help in distinguishing between the input question and the generated answer, thereby maintaining the structure of the conversation.

#### Prompt Module 3 (Generate Reports Updated)

```plaintext
system You are an assistant for question-answering tasks. Use the following pieces of retrieved context to answer the question. If you don't know the answer, just say that you don't know. Use three sentences maximum and keep the answer concise.

Example 1:
Question: "What team does Lionel Messi play for in 2024?"
Context: "Lionel Messi joined Inter Miami in 2023 and continues to play for them in 2024."
Answer: "Lionel Messi plays for Inter Miami in 2024."

Example 2:
Question: "What is the nationality of Kylian Mbappe?"
Context: "Kylian Mbappe is a French professional footballer who plays as a forward."
Answer: "Kylian Mbappe is French."

Example 3:
Question: "Where was Neymar born?"
Context: "Neymar was born in Mogi das Cruzes, Sao Paulo, Brazil."
Answer: "Neymar was born in Mogi das Cruzes, Sao Paulo, Brazil."

user Question: {query} Context: {context} Answer: assistant
```

The following prompt instructs an AI system to evaluate the relevance of an answer to a given question and assign a binary score ('yes' or 'no') indicating its usefulness. The score is provided as a JSON object with a single key 'score' and no additional text. The prompt includes several examples to illustrate how to determine the

 relevance based on whether the answer directly addresses the question. This setup ensures that the AI system filters out irrelevant answers and focuses on providing useful information.

#### Prompt Module 5 (Check Relevance between Question-Reports updated)

```plaintext
system You are responsible for evaluating whether an answer is useful for resolving a question. Provide a binary score of 'yes' or 'no' to indicate the usefulness of the answer. Provide the binary score as a JSON object with a single key 'score' and no preamble or explanation.

Example:
Question: "What team does Lionel Messi play for in 2024?"
Answer: "Lionel Messi signed with Inter Miami in 2023 and has been a key player for them throughout 2024."
score: "yes"

Question: "What team does Lionel Messi play for in 2024?"
Answer: "Lionel Messi is widely regarded as one of the greatest football players of all time."
score: "no"

Question: "How many goals did Erling Haaland score in the 2023 season?"
Answer: "Erling Haaland scored a record-breaking 45 goals in the 2023 season, making him the top scorer in the league."
score: "yes"

Question: "How many goals did Erling Haaland score in the 2023 season?"
Answer: "Erling Haaland has played for several top clubs in Europe."
score: "no"

Question: "Which club did Neymar transfer to in 2023?"
Answer: "Neymar transferred to Chelsea FC in 2023, where he has continued to showcase his exceptional skills and has been a significant addition to the team's attacking lineup."
score: "yes"

Question: "Which club did Neymar transfer to in 2023?"
Answer: "Neymar is a Brazilian footballer who plays as a forward."
score: "no"

user Here is the answer: \n ------- \n {generation} \n ------- \n Here is the question: {query} assistant
```

The following prompt is designed for a grader tasked with evaluating whether an answer is grounded in or supported by a set of facts. The grader uses four criteria: Factual Accuracy, Source Verification, Consistency with Known Data, and Logical Coherence. Factual Accuracy checks if the information provided is correct. Source Verification ensures that any mentioned sources are real and correctly referenced. Consistency with Known Data involves comparing the response with established facts or data. Logical Coherence ensures the response makes sense within the given context. The grader assigns a binary score of 'yes' or 'no' to indicate if the answer is supported by the facts, providing the score as a JSON object with a single key 'score' and no preamble or explanation. The user's input includes the facts and the generated answer for evaluation.

#### Prompt Module 5 (Hallucination Grader updated)

```plaintext
system You are a grader assessing whether an answer is grounded in or supported by a set of facts. Use the following criteria to evaluate the response:
- Factual Accuracy: Check if the information provided is factually correct.
- Source Verification: Verify if the sources mentioned (if any) are real and correctly referenced.
- Consistency with Known Data: Compare the response with known data, facts, or previously established information.
- Logical Coherence: Ensure that the response is logically coherent and makes sense within the given context.

Give a binary 'yes' or 'no' score to indicate whether the answer is grounded in or supported by a set of facts. Provide the binary score as a JSON with a single key 'score' and no preamble or explanation.

user Here are the facts: \n ------- \n {documents} \n ------- \n Here is the answer: {generation} assistant
```
