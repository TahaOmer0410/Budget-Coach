# Budget Coach

**Live demo:** [budgetcoach.streamlit.app](https://budgetcoach.streamlit.app/)

Budget Coach is a financial literacy assistant built in Python for people with no formal financial education. You enter your monthly income and expenses, and the app breaks your budget into needs, wants, and savings, checks your debt-to-income risk, and then explains the result in plain language. You can also ask follow-up questions like "what is an emergency fund?" and get answers grounded in a curated knowledge base instead of generic AI output.

It was built during the 1st Generative & Agentic AI Hackathon organized by [Pak Angels](https://www.linkedin.com/company/pakangels/), as part of the Pak Angels Generative & Agentic AI Training (Cohort 11) under [ASPIRE Pakistan](https://www.linkedin.com/company/aspirepk/). It is designed around the Pakistani context: amounts are in PKR, and the categories include things like committee (informal group savings) and mobile recharge.

The core design rule: **the AI never does the math.** All totals, percentages, and risk flags are computed by plain, deterministic code. The language model only explains numbers that are already correct, so a model hallucination can never produce a wrong financial figure.

## Features

- Web application built with Streamlit
- Budget analysis that splits expenses into needs, wants, and savings, with a bar chart and percentage breakdown
- Debt-to-income check that warns when debt repayment reaches 33% of income or more
- Plain-language explanation of your results, written by an LLM in under 200 words
- Follow-up Q&A powered by retrieval-augmented generation (RAG) over a financial literacy knowledge base
- One-click example budget ("Amina's budget") to try the app instantly
- Flags expenses it does not recognize instead of silently guessing a category
- Graceful failure: if the AI service is down, the budget math still works and a retry button appears
- Answer caching so repeated questions do not use extra API calls

## Project Structure

```
Budget-Coach/
├── app.py                      # Streamlit interface and main flow
├── budget_calc.py              # Deterministic budget and debt-risk calculations
├── rag_utils.py                # Chunking, embedding, and retrieval (FAISS)
├── financial_literacy_kb.txt   # Knowledge base used to answer questions
├── requirements.txt            # Python dependencies
├── .devcontainer/              # Dev container / Codespaces configuration
└── README.md
```

## Tech Stack

| Part | Tool |
|---|---|
| Interface | Streamlit |
| LLM | `openai/gpt-oss-120b`, served through the Groq API |
| Embeddings | `all-MiniLM-L6-v2` (sentence-transformers) |
| Vector search | FAISS (`IndexFlatIP` over normalized embeddings) |
| Budget logic | Plain Python with a dataclass result |

## Requirements

- Python 3.11 recommended (matches the dev container)
- A free [Groq API key](https://console.groq.com/) for the explanation and Q&A features
- The packages in `requirements.txt`

The budget calculation works without an API key. Only the plain-language explanation and the follow-up answers need one.

## Running the Project

Clone the repository:

```bash
git clone https://github.com/TahaOmer0410/Budget-Coach.git
```

Move into the project directory:

```bash
cd Budget-Coach
```

Install the dependencies:

```bash
pip install -r requirements.txt
```

Set your Groq API key:

```bash
# macOS / Linux
export GROQ_API_KEY="your-key-here"

# Windows (PowerShell)
$env:GROQ_API_KEY="your-key-here"
```

On Streamlit Community Cloud, add `GROQ_API_KEY` under **Secrets** instead.

Run the application:

```bash
streamlit run app.py
```

The first question you ask will take a few seconds longer, because the embedding model is downloaded and the knowledge base index is built once and then cached.

## How It Works

**Budget analysis**

1. You enter your monthly income and pick a category for each expense (or type your own label).
2. `budget_calc.py` matches each label to needs, wants, or savings using a keyword map. Anything it cannot match is reported as uncategorized and excluded from the breakdown.
3. It computes the totals, the leftover amount, and the debt-to-income ratio (debt and loan repayments divided by income).
4. If that ratio is 33% or higher, the app shows a risk warning. This threshold comes from the common banking rule of thumb that a ratio above one third is a warning sign.
5. The finished numbers are passed to the LLM, which is instructed not to recalculate anything and only to explain them gently in plain language.

**Follow-up questions (RAG)**

1. At startup, the knowledge base is split into chunks (paragraphs, capped at 120 words each) and embedded with `all-MiniLM-L6-v2`.
2. The embeddings are stored in a FAISS index, so similarity search is fast.
3. When you ask a question, it is embedded the same way and the 3 most similar chunks are retrieved.
4. Those chunks are given to the LLM with the instruction to answer using only the reference material, in short and simple language.

## What I Learned

Building this project helped me understand:

- Why calculations and language generation should be separated: code for the numbers, AI for the explanation
- How retrieval-augmented generation grounds answers in a trusted source instead of the model's memory
- How text embeddings and vector search (FAISS) find relevant passages by meaning, not keywords
- How to design for failure: never exposing raw API errors, offering retries, and keeping core features working without the AI
- How to design for a specific audience, including local currency, local savings habits, and plain language for first-time users
- How to build and deploy a working AI application with Streamlit in a hackathon timeframe

## Future Improvements

- Expand the category map and knowledge base using feedback from real users
- Let users correct an uncategorized item instead of just flagging it
- Add Urdu language support for explanations and answers
- Add a conversation history so follow-up questions can build on each other
- Track budgets across several months to spot trends instead of a single snapshot
- Add automated tests for the budget logic

## Team

- [Taha Omer](https://github.com/TahaOmer0410)
- Shazil Zia
- Muhammad Shameer
- Muhammad Omer

## Acknowledgements

Thanks to Pak Angels, iCodeGuru, ASPIRE Pakistan, HEC Pakistan, NCEAC, and the Pakistan Engineering Council, as well as our trainers and mentors, for their guidance throughout the program.

## Disclaimer

Budget Coach is an educational tool. It does not provide licensed financial advice.

## License

This project was created for educational purposes as part of the Pak Angels Generative & Agentic AI Training.
