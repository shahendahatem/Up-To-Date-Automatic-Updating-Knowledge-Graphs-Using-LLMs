    
\section{SPARQL Query}\label{sec:appendixA}
This appendix contains the SPARQL query used to collect triples from Wikidata:

\begin{small}
{\small
\begin{lstlisting}
[language=SPARQL, basicstyle=\ttfamily\footnotesize, frame=single]
SELECT ?player ?playerLabel ?birthDate ?birthPlaceLabel ?nationalityLabel 
       ?memberOf2017Label ?memberOf2020Label ?memberOf2023Label ?memberOf2024Label
       ?spouseLabel ?homeLocationLabel ?genderLabel ?deathDate ?deathPlaceLabel ?childrenLabel WHERE {
  ?player wdt:P31 wd:Q5;                      (*@\#@*) Instance of human
          wdt:P106 wd:Q937857;               (*@\#@*) Occupation football player
          wdt:P569 ?birthDate.               (*@\#@*) Birth date

  OPTIONAL { ?player wdt:P19 ?birthPlace. }  (*@\#@*) Birth place
  OPTIONAL { ?player wdt:P27 ?nationality. } (*@\#@*) Nationality
  
  OPTIONAL { 
    ?player p:P54 ?teamMembership2017.
    ?teamMembership2017 ps:P54 ?memberOf2017.
    ?teamMembership2017 pq:P580 ?start_date2017.
    FILTER(YEAR(?start_date2017) = 2017)
  }                                          (*@\#@*) Member of sports team with start date in 2017
  
  OPTIONAL { 
    ?player p:P54 ?teamMembership2020.
    ?teamMembership2020 ps:P54 ?memberOf2020.
    ?teamMembership2020 pq:P580 ?start_date2020.
    FILTER(YEAR(?start_date2020) = 2020)
  }                                          (*@\#@*) Member of sports team with start date in 2020

  OPTIONAL { 
    ?player p:P54 ?teamMembership2023.
    ?teamMembership2023 ps:P54 ?memberOf2023.
    ?teamMembership2023 pq:P580 ?start_date2023.
    FILTER(YEAR(?start_date2023) = 2023)
  }                                          (*@\#@*) Member of sports team with start date in 2023

  OPTIONAL { 
    ?player p:P54 ?teamMembership2024.
    ?teamMembership2024 ps:P54 ?memberOf2024.
    ?teamMembership2024 pq:P580 ?start_date2024.
    FILTER(YEAR(?start_date2024) = 2024)
  }                                          (*@\#@*) Member of sports team with start date in 2024

  OPTIONAL { ?player wdt:P26 ?spouse. }      (*@\#@*) Spouse
  OPTIONAL { ?player wdt:P551 ?homeLocation.}(*@\#@*) Home location
  OPTIONAL { ?player wdt:P21 ?gender. }      (*@\#@*) Gender
  OPTIONAL { ?player wdt:P570 ?deathDate. }  (*@\#@*) Death date
  OPTIONAL { ?player wdt:P20 ?deathPlace. }  (*@\#@*) Death place
  OPTIONAL { ?player wdt:P40 ?children. }    (*@\#@*) Children

  FILTER(BOUND(?memberOf2017) || BOUND(?memberOf2020) || BOUND(?memberOf2023) || BOUND(?memberOf2024))

  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
LIMIT 1000  (*@\#@*) Adjust this to manage return size
OFFSET 0     (*@\#@*) Adjust this in subsequent queries to get new batches of data
\end{lstlisting}}
\end{small}
\section{Prompt Design}\label{sec:appendixB}

This section details the design of the prompts used in the system. Two distinct types of prompts were utilized: zero-shot and prompts incorporating few-shot examples.

\subsection{Zero-Shot Prompt Design}

Zero-shot prompts are straightforward instructions or questions designed to elicit specific responses from the language model. 
The first prompt is written to convert a user's natural language question into accurate web search queries. 

\begin{small}
\lstset{
    basicstyle=\ttfamily,
    frame=single,
    breaklines=true,
    showstringspaces=false
}
\begin{lstlisting}

Prompt Module 1 (Natural Language Transformation):

""" system You are an expert at crafting web search queries for research questions.
More often than not, a user will ask a basic question that they wish to learn more about, however it might not be in the best format.
Reword their query to be the three most effective web search strings possible.
Return the JSON with a single key 'queries' containing a list of three queries with no preamble or explanation.

Question to transform: {question}

assistant """
\end{lstlisting}
\end{small}
The following prompt evaluates the relevance of a retrieved document from different websites to the user question. It provides a binary score to filter out irrelevant documents.

\begin{small}
\lstset{
    basicstyle=\ttfamily,
    frame=single,
    breaklines=true,
    showstringspaces=false
}
\begin{lstlisting}
Prompt Module 3 (Website Grader):

""" system You are a grader assessing relevance 
of a retrieved document to a user question. If the document contains keywords related to the user question, 
grade it as relevant. It does not need to be a stringent test. The goal is to filter out erroneous retrievals.
Give a binary score 'yes' or 'no' score to indicate whether the document is relevant to the question.
Provide the binary score as a JSON with a single key 'score' and no preamble or explanation.

user
Here is the retrieved document: 
\n\n {document} \n\n
Here is the user question: {query} 
assistant """
\end{lstlisting}
\end{small}

The following prompt helps in generating responses based on the evaluated and retrieved context. It ensures that the answers are correct.
\begin{small}
\lstset{
    basicstyle=\ttfamily,
    frame=single,
    breaklines=true,
    showstringspaces=false
}
\begin{lstlisting}
Prompt Module 3 (Generate Reports):

"""user You are an assistant for question-answering tasks. 
Use the following pieces of retrieved context to answer the question. If you don't know the answer, just say that you don't know. 
Use three sentences maximum and keep the answer concise. 

user
Question: {query}
Context: {context}
Answer:
assistant"""
\end{lstlisting}
\end{small}
The following prompt assesses whether an answer is grounded in the provided facts. It ensures the reliability of the generated answers by verifying their factual basis. 
\begin{small}
\lstset{
    basicstyle=\ttfamily,
    frame=single,
    breaklines=true,
    showstringspaces=false
}
\begin{lstlisting}
Prompt Module 5 (Check Hallucination):

"""system You are a grader assessing whether 
an answer is grounded in / supported by a set of facts. Give a binary 'yes' or 'no' score to indicate 
whether the answer is grounded in / supported by a set of facts. Provide the binary score as a JSON with a 
single key 'score' and no preamble or explanation. 

user
Here are the facts:
\n ------- \n
{documents} 
\n ------- \n
Here is the answer: {generation} 

assistant"""
\end{lstlisting}
\end{small}
The following prompt is designed to evaluate the relevance of an answer in resolving a given question. 

\begin{small}
%modify all the prompt to be like this
\lstset{
    basicstyle=\ttfamily,
    frame=single,
    breaklines=true,
    showstringspaces=false
}
\begin{lstlisting}
Prompt Module 5 (Check Relevance between Question-Reports):

""" system You are a grader assessing whether an answer is useful to resolve a question. 
Give a binary score 'yes' or 'no' to indicate whether the answer is useful to resolve a question. 
Provide the binary score as a JSON with a single key 'score' and no preamble or explanation.
user Here is the answer:
\n ------- \n
{generation} 
\n ------- \n
Here is the question: {query} 
assistant """
\end{lstlisting}
\end{small}
This prompt extracts information from text to form a knowledge graph. It generates clear and concise triplets of subject, relation, and object based on the input text.
\begin{small}
\lstset{
    basicstyle=\ttfamily,
    frame=single,
    breaklines=true,
    showstringspaces=false
}
\begin{lstlisting}
Prompt Module 5 (Get Triples):

""" system You are an advanced algorithm designed to analyze text and extract information to form a knowledge graph with clear, concise triplets of subject, relation, and object. Your output should strictly reflect the information available in the input text without inferring or adding additional details.
- Nodes should represent entities and concepts directly mentioned in the input.
- Relations should clearly define the connection between nodes based on the input.
- Format: Output each extracted piece of information as a triplet in the format: (Subject, Relation, Object).
- Compliance: Strictly adhere to the information present in the text.
- Incomplete Information: If the information is incomplete or unknown, state that explicitly in the output.
\n ------- \n
{input} 
\n ------- \n
Tip: Ensure your output is accurate and directly derived from the input text
assistant"""
\end{lstlisting}
\end{small}
\subsection{Few-Shot Example Prompts}

Some prompts have been updated with a few examples:router, website grader, generate reports, and hallucination grader to get better results. The following prompt routes user questions to either a graph database or web search based on the nature of the information. Non-time-sensitive queries are routed to the graph database, while time-sensitive queries are routed to the web search.
\begin{small}
format \lstset{
    basicstyle=\ttfamily,
    frame=single,
    breaklines=true,
    showstringspaces=false
}
\begin{lstlisting}

Prompt Module 2 (Router):
""" You are an expert at routing a user question to either a graph database or web search based on the nature of the information. Use the graph database for non-time-sensitive queries that pertain to historical dates, static information, or long-standing facts. Use web search for time-sensitive queries involving recent events, current statuses, or urgent matters. Here are the detailed guidelines:

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
    Here is the user's query: {query} """
\end{lstlisting}
\end{small}

The following updated prompt for the website grader is designed to evaluate the relevance of a retrieved document in response to a user's question. The evaluator is instructed to determine if the document contains keywords or information directly related to the question. The main goal is to filter out clearly irrelevant results rather than applying overly strict criteria. Examples are provided within the prompt to illustrate how to determine relevance based on the user's question and the retrieved document.
\begin{small}   
\lstset{
    basicstyle=\ttfamily,
    frame=single,
    breaklines=true,
    showstringspaces=false
}
\begin{lstlisting}
Prompt Module 3 (Website Grader Updated):

""" system You are responsible for evaluating the relevance 
of a retrieved document to a user's question. If the document contains keywords or information directly related to the user's question, 
mark it as relevant. This evaluation does not need to be overly strict; the main goal is to filter out clearly irrelevant results. 
Assign a binary score of 'yes' for relevant or 'no' for irrelevant. 
Provide the score as a JSON object with a single key 'score' and no preamble or explanation.

Example:
User Question: "What is the current team of Cristiano Ronaldo?"
Retrieved Document: "Cristiano Ronaldo joined Al-Nassr in 2023."
score: "yes"

User Question: "Where was Cristiano Ronaldo born?"
Retrieved Document: "Cristiano Ronaldo joined Al-Nassr in 2023."
Score: "no"

user
Here is the retrieved document: \n\n {document} \n\n
Here is the user question: {query} 
assistant """
\end{lstlisting}
\end{small}
This prompt is updated to instruct an AI assistant on how to perform question-answering tasks. The assistant is guided to use the given context to generate concise and relevant answers to the user's questions. If the assistant does not know the answer, it should explicitly state that it doesn't know. The responses should be limited to a maximum of three sentences to ensure conciseness. The prompt includes three examples to illustrate the expected format and style of the answers. The labels 'user' and 'assistant' help in distinguishing between the input question and the generated answer, thereby maintaining the structure of the conversation.
\begin{small}
\lstset{
    basicstyle=\ttfamily,
    frame=single,
    breaklines=true,
    showstringspaces=false
}
\begin{lstlisting}
Prompt Module 3 (Generate Reports Updated):

system You are an assistant for question-answering tasks. 
Use the following pieces of retrieved context to answer the question. If you don't know the answer, just say that you don't know. 
Use three sentences maximum and keep the answer concise.

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

user
Question: {query}
Context: {context}
Answer:
assistant
\end{lstlisting}
\end{small}

The following prompt instructs an AI system to evaluate the relevance of an answer to a given question and assign a binary score ('yes' or 'no') indicating its usefulness. The score is provided as a JSON object with a single key 'score' and no additional text. The prompt includes several examples to illustrate how to determine the relevance based on whether the answer directly addresses the question. This setup ensures that the AI system filters out irrelevant answers and focuses on providing useful information.
\begin{small}
\lstset{
    basicstyle=\ttfamily,
    frame=single,
    breaklines=true,
    showstringspaces=false
}
\begin{lstlisting}
Prompt Module 5 (Check Relevance between Question-Reports updated):

""" system You are responsible for evaluating whether an 
answer is useful for resolving a question. Provide a binary score of 'yes' or 'no' to indicate the usefulness of the answer. 
Provide the binary score as a JSON object with a single key 'score' and no preamble or explanation.

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

user Here is the answer:
\n ------- \n
{generation} 
\n ------- \n
Here is the question: {query} 
assistant """
\end{lstlisting}
\end{small}
The following prompt is designed for a grader tasked with evaluating whether an answer is grounded in or supported by a set of facts. The grader uses four criteria: Factual Accuracy, Source Verification, Consistency with Known Data, and Logical Coherence. Factual Accuracy checks if the information provided is correct. Source Verification ensures that any mentioned sources are real and correctly referenced. Consistency with Known Data involves comparing the response with established facts or data. Logical Coherence ensures the response makes sense within the given context. The grader assigns a binary score of 'yes' or 'no' to indicate if the answer is supported by the facts, providing the score as a JSON object with a single key 'score' and no preamble or explanation. The user's input includes the facts and the generated answer for evaluation.
\begin{small}
\lstset{
    basicstyle=\ttfamily,
    frame=single,
    breaklines=true,
    showstringspaces=false
}
\begin{lstlisting}
Prompt Module 5 (Hallucination Grader updated):    

system You are a grader assessing whether an answer is grounded in or supported by a set of facts. 
Use the following criteria to evaluate the response:
- Factual Accuracy: Check if the information provided is factually correct.
- Source Verification: Verify if the sources mentioned (if any) are real and correctly referenced.
- Consistency with Known Data: Compare the response with known data, facts, or previously established information.
- Logical Coherence: Ensure that the response is logically coherent and makes sense within the given context.

Give a binary 'yes' or 'no' score to indicate whether the answer is grounded in or supported by a set of facts. 
Provide the binary score as a JSON with a single key 'score' and no preamble or explanation.

user
Here are the facts:
\n ------- \n
{documents} 
\n ------- \n
Here is the answer: {generation} 
assistant
\end{lstlisting}
\end{small}
