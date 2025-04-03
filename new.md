Step-by-Step Implementation (Python)

🔹 1. Set up Cognitive Search Client
from azure.search.documents import SearchClient
from azure.core.credentials import AzureKeyCredential
import os

search_client = SearchClient(
    endpoint=os.getenv("AZURE_SEARCH_ENDPOINT"),
    index_name=os.getenv("AZURE_SEARCH_INDEX"),
    credential=AzureKeyCredential(os.getenv("AZURE_SEARCH_API_KEY"))
)
🔹 2. Generate Query Embedding (using Azure OpenAI)
from openai import AzureOpenAI

client = AzureOpenAI(
    api_key=os.getenv("AZURE_OPENAI_API_KEY"),
    api_version="2023-07-01-preview",
    azure_endpoint=os.getenv("AZURE_OPENAI_ENDPOINT")
)

def get_embedding(text):
    response = client.embeddings.create(
        model="text-embedding-ada-002",
        input=[text]
    )
    return response.data[0].embedding
🔹 3. Perform Vector Search on Azure Search
def vector_search(user_prompt):
    vector = get_embedding(user_prompt)
    results = search_client.search(
        search_text="",  # required but ignored during vector search
        vector={
            "value": vector,
            "fields": "contentVector",
            "k": 5,
        },
        select=["content", "metadata"]
    )
    return [doc["content"] for doc in results]
contentVector is the name of the vector field used in your index schema
🔹 4. Generate Answer with GPT (RAG-style)
def generate_answer(user_prompt, docs):
    context = "\n".join(docs)
    messages = [
        {"role": "system", "content": "You are a helpful assistant. Answer only based on the provided context."},
        {"role": "user", "content": f"Context:\n{context}\n\nQuestion: {user_prompt}"}
    ]
    response = client.chat.completions.create(
        model="gpt-4",
        messages=messages
    )
    return response.choices[0].message.content
🔧 Example Integration
user_prompt = "What is the refund policy?"
docs = vector_search(user_prompt)
answer = generate_answer(user_prompt, docs)

print("Answer:", answer)
🚀 Result

The model gives a grounded, context-aware answer, strictly based on your own data stored in the Azure index.

🛡️ Pro Tips:
✅ Enable semantic re-ranking if you combine vector + keyword search.
✅ Use filters to restrict by metadata (e.g., document type, tenant).
✅ Cache embeddings to avoid repeated compute costs.
✅ Use user_id to log and trace requests for fine-tuning and auditing.
