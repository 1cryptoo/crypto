# Lab Exam Reference Programs

One section per question from `lab_exam_questions.docx`, each mapped to its source cell(s) in `Agentic_AI_Full_programs.ipynb`. Code is kept **exactly as in the source, unrun** (no outputs / execution counts) so you can read and memorize it quickly.

## MODULE 1

### Q1. Simple AI Agent using Agno framework to generate NVIDIA report
*(source: cell 4)*

```python
!pip install -U agno groq yfinance python-dotenv
# !pip install -U agno groq yfinance python-dotenv

import os
from agno.agent import Agent
from agno.models.groq import Groq
from agno.tools.yfinance import YFinanceTools
os.environ["GROQ_API_KEY"] = ""
agent = Agent(
    model=Groq(id="openai/gpt-oss-20b"),
    tools=[
        YFinanceTools(
            enable_stock_price=True,
            enable_company_info=True,
            enable_analyst_recommendations=True,
            enable_company_news=True
        )
    ],

    instructions=[
        "Use tables whenever possible.",
        "Generate a professional financial report.",
        "Include current stock price.",
        "Include company information.",
        "Include analyst recommendations.",
        "Include recent company news.",
        "Give a concise summary."
    ],

    markdown=True
)

agent.print_response(
    "Generate a detailed report on NVIDIA (NVDA)",
    stream=True
)
```

```bash
!pip install -U agno groq yfinance python-dotenv
```

### Q2. Designing an AI agent architecture — implementing agentic AI design patterns

#### 2a. Reflection design pattern
*(source: cell 6)*

```python
from groq import Groq

client = Groq(
    api_key=""
)

question = "Explain Agentic AI in one sentence."

# First answer
answer = client.chat.completions.create(
    model="openai/gpt-oss-20b",
    messages=[
        {"role":"user","content":question}
    ]
)

first_response = answer.choices[0].message.content

print("Initial Answer:")
print(first_response)

# Reflection
reflection = client.chat.completions.create(
    model="openai/gpt-oss-20b",
    messages=[
        {
            "role":"user",
            "content":f"""
Review and improve this answer:

{first_response}

Make it clearer for beginners.
"""
        }
    ]
)

print("\nImproved Answer:")
print(reflection.choices[0].message.content)
```

#### 2b. Planning design pattern
*(source: cell 8)*

```python
from groq import Groq

client = Groq(
    api_key=""
)

response = client.chat.completions.create(
    model="openai/gpt-oss-20b",
    messages=[
        {
            "role":"user",
            "content":"""
Goal:
Organize a college technical fest.

Create a detailed step-by-step plan.
"""
        }
    ]
)

print(response.choices[0].message.content)
```

#### 2c. ReAct design pattern
*(source: cell 10)*

```python
from groq import Groq

client = Groq(
    api_key=""
)

response = client.chat.completions.create(
    model="openai/gpt-oss-20b",
    messages=[
        {
            "role":"user",
            "content":"""
Question:
What is 25 × 18?

Show:

Thought
Action
Observation
Final Answer
"""
        }
    ]
)

print(response.choices[0].message.content)
```

#### 2d. Multi-agent design pattern
*(source: cell 12)*

```python
from groq import Groq

client = Groq(
    api_key=""
)

researcher = client.chat.completions.create(
    model="openai/gpt-oss-20b",
    messages=[
        {
            "role":"user",
            "content":"Research Agent: Explain Agentic AI."
        }
    ]
)

writer = client.chat.completions.create(
    model="openai/gpt-oss-20b",
    messages=[
        {
            "role":"user",
            "content":f"""
Writer Agent:
Create a report from:

{researcher.choices[0].message.content}
"""
        }
    ]
)

reviewer = client.chat.completions.create(
    model="openai/gpt-oss-20b",
    messages=[
        {
            "role":"user",
            "content":f"""
Reviewer Agent:
Improve this report:

{writer.choices[0].message.content}
"""
        }
    ]
)

print(reviewer.choices[0].message.content)
```

#### 2e. ReWOO design pattern
*(source: cell 14)*

```python
from groq import Groq

client = Groq(
    api_key=""
)

user_query = """
Create a report about Artificial Intelligence.
"""

planner = client.chat.completions.create(
    model="openai/gpt-oss-20b",
    messages=[
        {
            "role":"user",
            "content":f"""
You are a Planner Agent.

For the task below:

{user_query}

Create a step-by-step plan.

Do NOT execute.
Only generate the plan.
"""
        }
    ]
)

plan = planner.choices[0].message.content

print("\n========== GENERATED PLAN ==========\n")
print(plan)

executor = client.chat.completions.create(
    model="openai/gpt-oss-20b",
    messages=[
        {
            "role":"user",
            "content":f"""
Execute the following plan:

{plan}

Generate the final report.
"""
        }
    ]
)

result = executor.choices[0].message.content

print("\n========== FINAL OUTPUT ==========\n")
print(result)
```

## MODULE 2

### Q3. How to convert the chain to Web API using LangServe framework
*(source: cell 32, confirmed working per notebook note)*

```python
python -m pip install -U langchain langchain-core langchain-groq langserve fastapi uvicorn jsonpatch sniffio requests-toolbelt tenacity

import os
from fastapi import FastAPI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_groq import ChatGroq
from langserve import add_routes

os.environ["GROQ_API_KEY"] = "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"

prompt = ChatPromptTemplate.from_template("Answer briefly: {question}")

llm = ChatGroq(model="openai/gpt-oss-20b")

chain = prompt | llm | StrOutputParser()

app = FastAPI(title="Agentic AI API")

add_routes(app, chain, path="/chat")



#save as app.py
#python -m uvicorn app:app
```

### Q4. Build a Self-correcting Coding Assistant with LangChain
*(source: cells 48–49)*

```python
import os
os.environ["GROQ_API_KEY"] = "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"

import subprocess
import sys
import tempfile
from pathlib import Path

from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_groq import ChatGroq


# LLM
llm = ChatGroq(model="openai/gpt-oss-20b")


# Generate code
generate = ChatPromptTemplate.from_template(
    "Write only Python code for this task: {task}"
) | llm | StrOutputParser()


# Fix code
repair = ChatPromptTemplate.from_template(
    "Fix this Python code. Return code only.\nCode:\n{code}\nError:\n{error}"
) | llm | StrOutputParser()


# Remove ```python ``` from LLM response
def clean(code):
    return code.replace("```python", "").replace("```", "").strip()


# Run Python code
def run(code):
    path = Path(tempfile.gettempdir()) / "agent_answer.py"
    path.write_text(code)

    return subprocess.run(
        [sys.executable, str(path)],
        capture_output=True,
        text=True
    )


# Generate first code
code = clean(
    generate.invoke({
        "task": "Print the factorial of 5."
    })
)


# Self-correction loop
for attempt in range(1, 4):

    print(f"\nAttempt {attempt}:")
    print(code)

    result = run(code)

    if result.returncode == 0:
        print("\nOUTPUT:")
        print(result.stdout)

        print("FINAL CODE:")
        print(code)
        break

    print("\nERROR:")
    print(result.stderr)

    code = clean(
        repair.invoke({
            "code": code,
            "error": result.stderr
        })
    )
```

Run the assistant *(source: cell 49)*

```python
final_code = self_correcting_coding_assistant(
    "Write Python code to calculate factorial of 5 using recursion."
)
```

### Q5. Building a Finance Bot with LangGraph
*(source: cell 51)*

```python
!pip install -q langgraph langchain langchain-core langchain-groq
import os
from typing import TypedDict
from langgraph.graph import StateGraph, START, END
from langchain_groq import ChatGroq
from IPython.display import Image, display
os.environ["GROQ_API_KEY"] = "YOUR KEY"
llm = ChatGroq(model="llama-3.1-8b-instant")
# 1. Define State
class FinanceState(TypedDict):
    question: str
    stock: str
    price: float
    answer: str
# 2. Node 1: Extract stock symbol
def extract_stock(state: FinanceState):
    question = state["question"].upper()
    if "APPLE" in question or "AAPL" in question:
        stock = "AAPL"
    elif "TESLA" in question or "TSLA" in question:
        stock = "TSLA"
    elif "MICROSOFT" in question or "MSFT" in question:
        stock = "MSFT"
    else:
        stock = "UNKNOWN"
    return {"stock": stock}
# 3. Node 2: Get stock price
def get_stock_price(state: FinanceState):
    fake_prices = {
        "AAPL": 195.50,
        "TSLA": 180.25,
        "MSFT": 420.10,
        "UNKNOWN": 0.0
    }
    price = fake_prices[state["stock"]]
    return {"price": price}
# 4. Node 3: Generate answer
def generate_answer(state: FinanceState):
    prompt = f"""
    You are a finance assistant.
    User question: {state['question']}
    Stock symbol: {state['stock']}
    Stock price: {state['price']}
    Give a simple financial response.
    Do not give investment advice.
    """
    response = llm.invoke(prompt)
    return {"answer": response.content}
# 5. Build LangGraph
graph = StateGraph(FinanceState)
graph.add_node("extract_stock", extract_stock)
graph.add_node("get_stock_price", get_stock_price)
graph.add_node("generate_answer", generate_answer)
graph.add_edge(START, "extract_stock")
graph.add_edge("extract_stock", "get_stock_price")
graph.add_edge("get_stock_price", "generate_answer")
graph.add_edge("generate_answer", END)
finance_bot = graph.compile()
display(
    Image(finance_bot.get_graph().draw_mermaid_png())
)
# 6. Run Finance Bot
result = finance_bot.invoke({
    "question": "What is the current price of MICROSOFT stock?"
})
print(result["answer"])
```

## MODULE 3

### Q6. Create an AI-Powered Sales Report Analyzer with LlamaIndex
*(source: cell 63)*

```python
%pip install llama-index llama-index-llms-openai-like
import os
import pandas as pd
from llama_index.core import Document, VectorStoreIndex, Settings
from llama_index.core.embeddings import MockEmbedding
from llama_index.llms.openai_like import OpenAILike
# Groq API key
os.environ["GROQ_API_KEY"] = ""
# Use Groq with OpenAI-compatible endpoint
Settings.llm = OpenAILike(
    model="openai/gpt-oss-120b",
    api_base="https://api.groq.com/openai/v1",
    api_key=os.environ["GROQ_API_KEY"],
    is_chat_model=True,
    is_function_calling_model=False
)
# Simple embedding for classroom demo
Settings.embed_model = MockEmbedding(embed_dim=384)
# Sales report data
sales_df = pd.DataFrame({
    "Month": ["Jan", "Feb", "Mar", "Apr", "May"],
    "Product": ["Laptop", "Mobile", "Tablet", "Laptop", "Mobile"],
    "Region": ["South", "North", "East", "West", "South"],
    "Revenue": [50000, 40000, 25000, 60000, 55000],
    "Profit": [10000, 8000, 4000, 12000, 11000]
})
display(sales_df)
# Convert table into document
sales_text = sales_df.to_string(index=False)
document = Document(
    text=f"""
Sales Report:
{sales_text}
"""
)
# Create LlamaIndex index
index = VectorStoreIndex.from_documents([document])
# Create query engine
query_engine = index.as_query_engine()
# Ask question
response = query_engine.query(
    "Which product generated the highest total revenue? Show calculation."
)
print(response)
```

### Q7. Create a Market Research Agent with RAG & Cohere
*(source: cell 68)*

```python
!pip install -U cohere numpy pandas
import os
import cohere
import numpy as np
import pandas as pd
# 1. Set Cohere API Key
os.environ["COHERE_API_KEY"] = ""
co = cohere.ClientV2(api_key=os.environ["COHERE_API_KEY"])
# 2. Market research knowledge base
documents = [
    "Electric vehicles are growing rapidly due to rising fuel prices, government incentives, and environmental awareness.",
    "The Indian EV market is driven by two-wheelers, three-wheelers, and urban mobility demand.",
    "Major challenges in EV adoption include charging infrastructure, battery cost, range anxiety, and supply chain limitations.",
    "Key competitors in the EV market include Tata Motors, Mahindra Electric, Ola Electric, Ather Energy, and MG Motor.",
    "Customers prefer EVs because of low running cost, reduced emissions, and government subsidy benefits.",
    "Battery swapping and fast charging networks are emerging as important opportunities in the EV ecosystem.",
    "The market is expected to expand with support from renewable energy integration and smart mobility policies."
]
# 3. Create document embeddings
doc_embed_response = co.embed(
    model="embed-v4.0",
    texts=documents,
    input_type="search_document",
    embedding_types=["float"]
)
doc_embeddings = np.array(doc_embed_response.embeddings.float)
# 4. Similarity search function
def cosine_similarity(a, b):
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))
def retrieve_documents(query, top_k=5):
    query_embed_response = co.embed(
        model="embed-v4.0",
        texts=[query],
        input_type="search_query",
        embedding_types=["float"]
    )
    query_embedding = np.array(query_embed_response.embeddings.float[0])
    scores = [
        cosine_similarity(query_embedding, doc_embedding)
        for doc_embedding in doc_embeddings
    ]
    ranked_results = sorted(
        zip(documents, scores),
        key=lambda x: x[1],
        reverse=True
    )
    return ranked_results[:top_k]
# 5. Rerank with Cohere Rerank
def rerank_documents(query, retrieved_docs, top_n=3):
    docs_only = [doc for doc, score in retrieved_docs]
    rerank_response = co.rerank(
        model="rerank-v3.5",
        query=query,
        documents=docs_only,
        top_n=top_n
    )
    reranked_docs = [
        docs_only[result.index]
        for result in rerank_response.results
    ]
    return reranked_docs
# 6. Market Research Agent
def market_research_agent(query):
    retrieved_docs = retrieve_documents(query, top_k=5)
    final_docs = rerank_documents(query, retrieved_docs, top_n=3)

    context = "\n".join([f"- {doc}" for doc in final_docs])

    prompt = f"""
You are a professional Market Research Agent.

User Query:
{query}

Relevant Market Information:
{context}

Prepare a structured market research report with:
1. Market Overview
2. Key Drivers
3. Challenges
4. Competitor Landscape
5. Opportunities
6. Final Recommendation
"""

    response = co.chat(
        model="command-a-03-2025",
        messages=[
            {
                "role": "user",
                "content": prompt
            }
        ]
    )

    return response.message.content[0].text

# 7. Run the agent
query = "Prepare a market research report on electric vehicles in India"
answer = market_research_agent(query)

print(answer)
```

### Q8. Design a Data Analysis Agent with Phidata
*(source: cell 66)*

```python
%pip install -U phidata
import os
import pandas as pd

from phi.agent import Agent
from phi.model.groq import Groq

# Use your NEW Groq API key
os.environ["GROQ_API_KEY"] = ""

# Sample sales data
df = pd.DataFrame({
    "Month": ["Jan", "Feb", "Mar", "Apr", "May"],
    "Product": ["Laptop", "Mobile", "Tablet", "Laptop", "Mobile"],
    "Region": ["South", "North", "East", "West", "South"],
    "Revenue": [50000, 40000, 25000, 60000, 55000],
    "Profit": [10000, 8000, 4000, 12000, 11000]
})

display(df)

# Convert dataframe to text
data_text = df.to_string(index=False)

# Create Phidata agent
data_agent = Agent(
    name="Data Analysis Agent",
    model=Groq(id="openai/gpt-oss-120b"),
    instructions=[
        "You are a data analysis assistant.",
        "Analyze the given sales data carefully.",
        "Show calculations clearly.",
        "Give short business insights."
    ],
    markdown=True
)

# Question
question = "Which product has the highest total revenue? Show calculation."

# Run agent
response = data_agent.run(
    f"""
Here is the sales data:

{data_text}

Question:
{question}
"""
)

print(response.content)
```

## MODULE 4

### Q9. Simple Customer Support Chatbot using LangGraph Multi-Agent Workflow
*(source: cell 84)*

```python
%pip install -U langgraph langchain langchain-core langchain-groq ipython
import os
from typing import TypedDict
from langgraph.graph import StateGraph, END
from langchain_groq import ChatGroq
from langchain_core.messages import HumanMessage
from IPython.display import Image, display

# Add your Groq API key
os.environ["GROQ_API_KEY"] = ""

llm = ChatGroq(
    model="openai/gpt-oss-120b",
    temperature=0
)

class ChatState(TypedDict):
    query: str
    category: str
    response: str

def router_agent(state):
    query = state["query"]

    prompt = f"""
Classify this customer query into one category:
faq, order, or complaint.

Query: {query}

Return only one word.
"""
    result = llm.invoke([HumanMessage(content=prompt)])
    category = result.content.strip().lower()

    if "order" in category:
        category = "order"
    elif "complaint" in category:
        category = "complaint"
    else:
        category = "faq"

    return {"category": category}

def faq_agent(state):
    query = state["query"]

    prompt = f"""
You are an FAQ support agent.
Answer this customer question politely:

{query}
"""
    result = llm.invoke([HumanMessage(content=prompt)])
    return {"response": result.content}

def order_agent(state):
    query = state["query"]

    prompt = f"""
You are an order support agent.
Help the customer with order-related query.
If order ID is missing, ask for it politely.

Query: {query}
"""
    result = llm.invoke([HumanMessage(content=prompt)])
    return {"response": result.content}

def complaint_agent(state):
    query = state["query"]

    prompt = f"""
You are a complaint support agent.
Apologize politely and suggest the next step.

Complaint: {query}
"""
    result = llm.invoke([HumanMessage(content=prompt)])
    return {"response": result.content}

def route(state):
    if state["category"] == "order":
        return "Order_Agent"
    elif state["category"] == "complaint":
        return "Complaint_Agent"
    else:
        return "FAQ_Agent"

workflow = StateGraph(ChatState)

workflow.add_node("Router_Agent", router_agent)
workflow.add_node("FAQ_Agent", faq_agent)
workflow.add_node("Order_Agent", order_agent)
workflow.add_node("Complaint_Agent", complaint_agent)

workflow.set_entry_point("Router_Agent")

workflow.add_conditional_edges(
    "Router_Agent",
    route,
    {
        "FAQ_Agent": "FAQ_Agent",
        "Order_Agent": "Order_Agent",
        "Complaint_Agent": "Complaint_Agent"
    }
)

workflow.add_edge("FAQ_Agent", END)
workflow.add_edge("Order_Agent", END)
workflow.add_edge("Complaint_Agent", END)

app = workflow.compile()
display(Image(app.get_graph().draw_mermaid_png()))

# Test chatbot
query = "I received a damaged product"

result = app.invoke({
    "query": query,
    "category": "",
    "response": ""
})

print("User Query:", query)
print("Category:", result["category"])
print("Bot Response:", result["response"])
```

### Q10. Design a Stock Analysis Agent with CrewAI
*(source: cells 86–87)*

```python
# Simple Stock Analysis Agent using CrewAI
!pip install -q -U crewai yfinance nest_asyncio
import os
import yfinance as yf
import nest_asyncio
from crewai import Agent, Task, Crew, Process, LLM

nest_asyncio.apply()

os.environ["OPENAI_API_KEY"] = "YOUR KEY"

llm = LLM(
    model="gpt-4o-mini",
    api_key=os.environ["OPENAI_API_KEY"]
)

# Tool function
def get_stock_info(ticker):
    stock = yf.Ticker(ticker)
    info = stock.info

    return f"""
Company: {info.get('longName')}
Sector: {info.get('sector')}
Current Price: {info.get('currentPrice')}
Market Cap: {info.get('marketCap')}
P/E Ratio: {info.get('trailingPE')}
52 Week High: {info.get('fiftyTwoWeekHigh')}
52 Week Low: {info.get('fiftyTwoWeekLow')}
"""

# Get stock data
ticker = "AAPL"
stock_data = get_stock_info(ticker)

# Agent 1
analyst = Agent(
    role="Stock Analyst",
    goal="Analyze stock data and explain it simply",
    backstory="You are a financial analyst who explains stock performance clearly.",
    llm=llm,
    verbose=True
)

# Agent 2
writer = Agent(
    role="Report Writer",
    goal="Write a simple stock analysis report",
    backstory="You write beginner-friendly financial reports.",
    llm=llm,
    verbose=True
)

# Tasks
analysis_task = Task(
    description=f"Analyze this stock data:\n{stock_data}",
    expected_output="A short analysis with strengths and risks.",
    agent=analyst
)

report_task = Task(
    description="Write a simple stock analysis report using the analysis.",
    expected_output="A structured report with overview, analysis, risks, and conclusion.",
    agent=writer
)

# Crew
crew = Crew(
    agents=[analyst, writer],
    tasks=[analysis_task, report_task],
    process=Process.sequential,
    verbose=True
)

# Run
result = await crew.kickoff_async()

print(result)
```

### Q11. Develop an AI Research Agent with Autogen
*(source: cell 96)*

```python
import os
from autogen import AssistantAgent, UserProxyAgent


os.environ["GROQ_API_KEY"] = ""


llm_config = {
    "config_list": [
        {
            "model": "openai/gpt-oss-20b",
            "api_key": os.environ["GROQ_API_KEY"],
            "base_url": "https://api.groq.com/openai/v1"
        }
    ]
}

research_agent = AssistantAgent(
    name="Research_Agent",
    system_message="""
You are an AI Research Agent.

Your task is to:
1. Study the given research topic.
2. Identify important concepts and recent trends.
3. Summarize key methods and applications.
4. Discuss advantages and limitations.
5. Present the findings in a clear research-report format.
""",
    llm_config=llm_config,
)


user_proxy = UserProxyAgent(
    name="User_Proxy",

    # No human input during execution
    human_input_mode="NEVER",

    code_execution_config={
        "work_dir": "research",
        "use_docker": False
    }
)

user_proxy.initiate_chat(
    research_agent,
    message="""
Research the topic:

"Applications of Generative AI in Healthcare"

Prepare a short research report containing:
- Introduction
- Important applications
- Benefits
- Challenges
- Future scope
- Conclusion
"""
)
```

## MODULE 5

### Q12. AI Observability with LangSmith — trace one LLM call and view it in the LangSmith dashboard
*(source: cell 99)*

```python
# Install:
# !pip install -U langsmith langchain-groq

import os
from langchain_groq import ChatGroq


os.environ["GROQ_API_KEY"] = ""
os.environ["LANGSMITH_API_KEY"] = ""


os.environ["LANGSMITH_TRACING"] = "true"


os.environ["LANGSMITH_PROJECT"] = "AI-Observability-Demo"


llm = ChatGroq(
    model="openai/gpt-oss-20b"
)

response = llm.invoke(
    "Explain Artificial Intelligence in two sentences."
)

print(response.content)
```
