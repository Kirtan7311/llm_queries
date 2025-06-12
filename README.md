# LLM-Based Code Generation on Energy Dataset

## Name
Kirtan Gangani

## LLM Used
Llama (llama-3.3-70b-versatile)

## Summary of Tasks Completed

This project explores the use of Large Language Models (LLMs) for generating Python code to analyze the UCI Household Power Consumption Dataset. The core objective is to translate natural language queries into executable pandas code, focusing on understanding energy consumption patterns. The `llm_queries.ipynb` notebook details the process, including the natural language queries, the LLM prompts used, the generated code, and the corresponding outputs.

---

## Types of Prompts Used

In this implementation, two primary prompt strategies were employed:

### A) Global Approach:

This approach involves crafting a comprehensive "global prompt" that provides the LLM with a foundational understanding of the dataset and the overall task. Specific queries are then appended to this pre-contextualized prompt.

The global prompt incorporates the following elements:

* **DataFrame column names and their units:** This ensures the LLM is aware of the available data fields and their respective scales.
* **Problem statement:** Clearly defines the objective (generate pandas code, generate graphs if possible, prioritize memory-efficient operations).
* **Information on active energy consumption calculation:** Provides context for potential future queries related to energy consumption metrics.

**Variations within the Global Approach:**

1.  **Deeper understanding of the dataset:** This variation of the global prompt includes more detailed descriptions, relationships between columns, or specific nuances of the data. The aim is to enable the model to generate more sophisticated and accurate responses by providing richer contextual information.
2.  **Lower level understanding of the dataset:** This variation aims for a more concise prompt by eliminating information deemed less critical for common queries. This can be beneficial for reducing token usage or improving response time, especially if the model tends to struggle with overly extensive inputs.

### B) Local Approach:

In contrast to the global approach, the "Local Approach" involves creating a prompt that is precisely tailored to address a single, specific query. This method focuses on providing only the information directly relevant to the current question, minimizing extraneous context.

---

## LLM Interaction Details

The `groq_llama` function is used to interact with the Llama 3.3 70B Versatile model via the Groq API. This function takes a natural language `content` string as input, sends it to the LLM, and extracts the generated Python code for execution.

Here's a breakdown of the key parameters used in the `client.chat.completions.create` call:

* **`model`**: Set to `"llama-3.3-70b-versatile"` to specify the particular LLM version used for code generation.
* **`messages`**: Contains the user's prompt. We use a single message with the role `user` and the `content` provided to the function.
* **`temperature`**: Set to `0.15`. This low temperature value encourages the model to produce more deterministic and focused output, which is crucial for consistent and accurate code generation. A higher temperature would lead to more creative but potentially less reliable code.
* **`max_completion_tokens`**: Set to `1024`. This parameter controls the maximum length of the generated response from the LLM, ensuring responses are not excessively long.
* **`top_p`**: Set to `1`. This parameter, combined with temperature, influences the diversity of token selection. A value of 1 means all tokens are considered (up to the cumulative probability threshold), but with a low `temperature`, its effect on output consistency is minimal.
* **`stream`**: Set to `True`. This enables streaming responses, allowing for real-time processing of the LLM's output chunks as they are generated.
* **`stop`**: Set to `None`, indicating no specific stop sequences are used to terminate the generation.

After receiving the streamed output, the function intelligently **extracts the Python code** from markdown code blocks (` ```python ` or ` ``` `) and then **executes** it directly using `exec(code, globals())`. This allows for immediate verification of the generated code's correctness.

---

## Required Natural Language Queries and Solutions

The following natural language queries were posed to the LLM, and the generated pandas code was executed and verified for correctness. The `llm_queries.ipynb` notebook contains the full details, including prompts, generated code, and outputs for each query.

1.  What was the **average active power consumption in March 2007**?
2.  What hour of the day had the **highest power usage on Christmas 2006**?
3.  Compare **energy usage (Global_active_power) on weekdays vs weekends**.
4.  Find days where **energy consumption exceeded 5 kWh**.
5.  Plot the **energy usage trend for the first week of January 2007**.
6.  Find the **average voltage for each day of the first week of February 2007**.
7.  What is the **correlation between global active power and sub-metering values**?

---

## Conclusion

By experimenting with two types of prompts (Global and Local), it was observed that the **Global prompt** consistently enabled the LLM to correctly generate code and provide accurate answers to the initial set of queries. However, the **Local prompt** demonstrated limitations, failing for some queries and often requiring more detailed phrasing to generate correct responses.

The LLM successfully provided correct answers and generated appropriate graphs for all seven core queries.

A particularly insightful observation arose when posing a query outside the dataset's time range: when asked to tell the average active power consumption in March 1999 (a period for which no data exists in the dataset):
* The **Global prompt** (specifically the one with more comprehensive information) correctly identified that no data exists for that period and, as a fallback, utilized the earliest available data.
* In contrast, the **other prompt** (likely the simpler Global variant or the Local approach) did not recognize the non-existent data and instead produced an empty graph.

This highlights that both types of prompts have their own pros and cons, making the choice dependent on the specific task and desired level of robustness.

### Global Approach (with its variations)

**Advantages:**
* **Comprehensive Context:** Provides the LLM with a rich understanding of the dataset, problem, and constraints upfront. This can lead to more accurate, relevant, and robust code generation or analysis, especially for complex or nuanced queries.
* **Reduced Ambiguity:** With all the necessary information readily available, the LLM is less likely to misinterpret queries or make assumptions.
* **Consistency:** Helps maintain consistency in the LLM's responses across different queries, as the core understanding remains stable.
* **Easier for Complex Tasks:** For tasks that involve multiple steps, specific domain knowledge (like "how to calculate active energy consumed"), or require adherence to strict guidelines (like "prioritize memory-efficient operations"), a global prompt provides the necessary scaffolding.
* **Better for Iterative Development:** If you're going to ask a series of related questions about the same dataset, the global prompt sets the stage effectively for all of them without needing to repeat context.

**Disadvantages:**
* **Token Usage/Cost:** Longer prompts consume more tokens, which can increase API costs and potentially impact response time.
* **Model Capacity Limits:** LLMs have a maximum context window (the number of tokens they can process at once). A very long global prompt might exceed this limit, leading to truncation or degraded performance.
* **Irrelevant Information Overhead:** For very simple queries, a global prompt might include a lot of information that isn't strictly necessary, potentially "diluting" the focus of the current query for the LLM.
* **Potential for "Prompt Fatigue":** If the LLM has to constantly re-evaluate a very long global prompt for every short query, it might become less efficient or even lose focus.

### Local Approach

**Advantages:**
* **Efficiency:** For simple, straightforward queries, a concise local prompt can be processed much faster and consume fewer tokens, leading to quicker and cheaper responses.
* **Targeted Responses:** The LLM focuses precisely on the information needed for that specific query, potentially leading to more direct and less verbose answers.
* **Reduced Cognitive Load for LLM:** Without a lot of extraneous information, the LLM can dedicate its processing power to solving the immediate problem.
* **Flexibility:** Allows for easy adaptation to new types of queries without needing to re-engineer a large global prompt.

**Disadvantages:**
* **Lack of Context:** For complex tasks, the absence of broader context can lead to incomplete, inaccurate, or generic responses.
* **Increased Prompt Engineering per Query:** You need to carefully craft each local prompt to ensure it contains all the necessary information for that specific query, which can be time-consuming for many different queries.
* **Potential for Inconsistency:** Without a shared global understanding, responses to similar queries might vary more.
* **Difficulty with Dependencies:** If a query depends on information from a previous step or requires domain-specific knowledge not included in the local prompt, the LLM might struggle.

---

## How to Run

To replicate the analysis, follow these steps:

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/Kirtan7311/llm_queries.git
    cd llm_queries
    ```
2.  **Download the dataset:**
    Download `household_power_consumption.txt` from the [UCI Household Power Consumption Dataset](https://archive.ics.uci.edu/ml/datasets/individual+household+electric+power+consumption) and place it in the root directory of this repository.
3.  **Install dependencies:**
    ```bash
    pip install pandas matplotlib seaborn jupyter groq
    ```
4.  **Run the Jupyter Notebook:**
    ```bash
    jupyter notebook llm_queries.ipynb
    ```
    Open `llm_queries.ipynb` in your browser and run all cells to see the queries, LLM interactions, generated code, and outputs.