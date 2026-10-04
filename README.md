# Health-Bot: NLP-Based Medical Question Answering System using RAG

Health-Bot is an **NLP-based medical question-answering project** that uses a **Retrieval-Augmented Generation (RAG)** pipeline to generate answers to health-related questions from a medical question-answer dataset.

The project combines **text preprocessing**, **Sentence Transformer embeddings**, **FAISS vector search**, **LangChain**, and a **Llama-2-7B Chat** language model. A simple **Gradio** interface is also included in the notebook as an experimental front end.

> **Important:** This project is intended for educational and research purposes only. It is not a substitute for professional medical advice, diagnosis, or treatment.

---

## Project Overview
The main objective of this project is to build a health-focused question-answering system that can:

- Process and clean medical question-answer data.
- Convert the medical knowledge base into vector embeddings.
- Retrieve relevant medical information for a user's question.
- Provide the retrieved context to a language model.
- Generate a helpful natural-language response using a RAG pipeline.

Instead of relying only on the language model's internal knowledge, the system first searches the prepared medical dataset for relevant information and then uses that context to generate an answer.

---

## System Workflow
## System Workflow

```text
Medical Q&A Dataset
        |
        v
Data Preparation & Text Preprocessing
        |
        v
Train / Test Split (80% / 20%)
        |
        v
Training Q&A Data converted to PDF Knowledge Base
        |
        v
PDF Loading + Text Chunking
        |
        v
Sentence-Transformer Embeddings
(all-MiniLM-L6-v2)
        |
        v
FAISS Vector Database
        |
        v
User Medical Question
        |
        v
Similarity-Based Retrieval (Top-k = 2)
        |
        v
Retrieved Context + Custom Prompt
        |
        v
Llama-2-7B-Chat
        |
        v
Generated Answer
```

---

## Dataset Preparation

The notebook starts with an Excel medical question-answer dataset containing the columns:

- `Description`
- `Patient`
- `Doctor`

The `Description` and `Patient` fields are combined to create a single **Question** field:

```text
Description: <description>; Patient: <patient text>
```

The `Doctor` column is renamed to **Answer**, producing the final two-column structure:

```text
Question | Answer
```

After preprocessing, the notebook performs an **80/20 train-test split** using `random_state=42`.

The recorded notebook output shows:

- **Training samples:** 197,228
- **Testing samples:** 49,308
- **Total after preprocessing:** 246,536 samples

The training and testing DataFrames are also exported to `Training_Dataset.pdf` and `Testing_Dataset.pdf` in the notebook workflow.

---

## Text Preprocessing

A custom preprocessing pipeline is applied to both questions and answers. It includes:

- Converting text to lowercase.
- Expanding common English contractions.
- Removing text enclosed in parentheses.
- Removing selected punctuation and special characters.
- Replacing selected Unicode spaces and hyphens with normal spaces.
- Removing null records.
- Removing duplicate records.

This produces cleaner text before building the retrieval knowledge base.

---

## Embeddings and Vector Database

The training PDF is loaded with LangChain's `PyPDFLoader` and divided into smaller chunks using `RecursiveCharacterTextSplitter`.

### Chunking configuration

```python
chunk_size = 500
chunk_overlap = 50
```

### Embedding model

The project uses:

```text
sentence-transformers/all-MiniLM-L6-v2
```

The embeddings are stored in a **FAISS** vector database, enabling semantic similarity search over the medical knowledge base.

---

## Retrieval-Augmented Generation (RAG)

The main question-answering pipeline is implemented with **LangChain RetrievalQA**.

For each user query:

1. The question is converted into an embedding.
2. FAISS retrieves the **2 most relevant chunks** from the medical knowledge base.
3. The retrieved chunks are inserted into a custom prompt as context.
4. The prompt and user question are passed to the language model.
5. The model generates the final response.

The custom prompt instructs the model to use the provided information and to say that it does not know when the available context is insufficient.

---

## Language Model

The notebook configures the following model through `CTransformers`:

```text
TheBloke/Llama-2-7B-Chat-GGML
```

Main generation settings used in the notebook:

```python
max_new_tokens = 512
temperature = 0.5
```

This model acts as the generation component of the RAG pipeline.

## Example

The notebook tests the RAG system with the question:

```text
What is the recommended treatment for hypertension?
```

The system retrieves hypertension-related information from the training knowledge base and generates a response using the retrieved context.

This demonstrates the complete flow from **semantic retrieval to answer generation**.

---

## Evaluation Approach

The notebook contains an experimental evaluation section based on **semantic similarity** between generated answers and reference answers.

The intended evaluation process is:

1. Generate answers for test questions.
2. Encode generated and expected answers using `all-MiniLM-L6-v2`.
3. Calculate cosine similarity between corresponding answers.
4. Use a similarity threshold of `0.7` to classify an answer as sufficiently similar.
5. Derive evaluation measures from the thresholded results.
## User Interface

The notebook includes a basic **Gradio** interface titled **Health-Bot** with a text input and text output.

```text
Title: Health-Bot
Description: Ask any medical related question
```

## Technologies Used

| Technology | Purpose |
|---|---|
| Python | Core programming language |
| Pandas | Dataset loading and manipulation |
| Scikit-learn | Train/test splitting and cosine similarity utilities |
| Regular Expressions (`re`) | Text preprocessing |
| ReportLab | Converting Q&A data into PDF files |
| LangChain | RAG pipeline, document loading, splitting, prompting, and RetrievalQA |
| Sentence Transformers | Semantic text embeddings |
| `all-MiniLM-L6-v2` | Embedding model |
| FAISS | Vector storage and semantic retrieval |
| CTransformers | Running the Llama model |
| Llama-2-7B-Chat-GGML | Answer generation |
| PyPDF / PyPDFLoader | Loading the PDF knowledge base |
| Gradio | Prototype user interface |
| Kaggle Notebook | Main notebook execution environment used by the project |


## Installation

The notebook installs or uses the following main packages:

```bash
pip install pandas openpyxl scikit-learn reportlab
pip install langchain
pip install faiss-gpu
pip install sentence-transformers
pip install ctransformers
pip install pypdf
pip install gradio
```

`PyDrive` is also installed in the notebook, although it is not required by the main RAG flow shown in the final implementation.

> **Note:** `faiss-gpu` and the embedding configuration in the main RAG cells assume a CUDA-capable environment. If you are running locally without a compatible GPU, the FAISS/embedding configuration may need to be changed to CPU-compatible alternatives.
## How to Run

The project was developed as a Jupyter/Kaggle notebook. To reproduce the notebook workflow:

1. Clone this repository.
2. Open `NLP_Healthhbot_Project_final.ipynb` in Jupyter Notebook, JupyterLab, Google Colab, or Kaggle.
3. Add the medical dataset and update the dataset path if your environment differs from the original Kaggle path:

```python
/kaggle/input/dataset/dataset.xlsx
```

4. Run the data preparation and preprocessing cells.
5. Generate the training/testing PDF files.
6. Install the required dependencies.
7. Run the embedding cell to create the FAISS vector database.
8. Run the RAG implementation cells to load Llama-2 and initialize the retrieval pipeline.
9. Pass a medical question to `final_result(query)`.

Example:

```python
query = "What is the recommended treatment for hypertension?"
result = final_result(query)
print(result)
```

Because several paths in the notebook are environment-specific, update Kaggle/Colab paths according to where your dataset and generated files are stored.

---

## Repository Structure

A minimal GitHub repository can be organized as:

```text
Health-Bot/
|
|-- NLP_Healthhbot_Project_final.ipynb
|-- README.md
|-- requirements.txt        # Recommended addition
|-- .gitignore              # Recommended addition
```

Large generated files such as the training/testing PDFs, FAISS indexes, or model files generally do not need to be committed if they can be reproduced from the source dataset.

---
## Medical Disclaimer

Health-Bot is an academic/NLP project created to demonstrate retrieval-augmented question answering over medical text. The generated responses may be incomplete, inaccurate, or inappropriate for an individual patient's circumstances.

**Do not use this project as a replacement for a qualified healthcare professional.** For medical concerns, diagnosis, medication decisions, or emergencies, consult an appropriate medical professional or emergency service.

---

## Author

**Komal Wahid**

Data Science / Machine Learning Project

---

## Acknowledgements

This project uses open-source tools and models including LangChain, FAISS, Sentence Transformers, CTransformers, Gradio, and the Llama-2 ecosystem.
