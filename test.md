# llm_remediation_prompt_generator.py

from typing import List

# --------------------
# STEP 1: Extract IDs
# --------------------
def extract_finding_and_control_ids(doc_text: str):
    print("Extracting Finding ID and Control ID...")
    lines = doc_text.split('\n')
    finding_id = None
    control_id = None
    for line in lines:
        if "Finding ID" in line:
            finding_id = line.split(":")[-1].strip()
        elif "Control ID" in line:
            control_id = line.split(":")[-1].strip()
    return finding_id, control_id

# --------------------
# STEP 2: Simulated embedding model
# --------------------
def embed_query_text(text: str):
    print(f"Generating embedding for: {text}")
    return f"embedding::{text.strip().replace(' ', '_')}"

# --------------------
# STEP 3: Simulated vector DB with control + finding details
# --------------------
VECTOR_DB = {
    "embedding::FND-1234": ["FND-1234: Data retention policy violated due to prolonged storage beyond legal limits."],
    "embedding::AB-XX32.42.4-2": ["AB-XX32.42.4-2: Control mandates quarterly review of access logs for compliance."]
}

# --------------------
# STEP 4: Retrieve vector context
# --------------------
def retrieve_vector_context(finding_id: str, control_id: str) -> List[str]:
    print("Retrieving context from vector DB...")
    context = []
    embedding_finding = embed_query_text(finding_id)
    embedding_control = embed_query_text(control_id)

    context.extend(VECTOR_DB.get(embedding_finding, [f"No context found for {finding_id}"]))
    context.extend(VECTOR_DB.get(embedding_control, [f"No context found for {control_id}"]))

    return context

# --------------------
# STEP 5: Prepare final prompt for LLM
# --------------------
def prepare_llm_prompt(remediation_text: str, finding_id: str, control_id: str, context: List[str]):
    print("Composing final prompt for LLM...")
    return f"""You are reviewing a newly submitted remediation plan. Below are the details:

--- Remediation Plan Content ---
{remediation_text}

--- Referenced IDs ---
- Finding ID: {finding_id}
- Control ID: {control_id}

--- Related Historical Context from Knowledge Base ---
{chr(10).join(f"- {item}" for item in context)}

Based on the above, provide an enhanced and aligned remediation recommendation.
"""

# --------------------
# MAIN PIPELINE FUNCTION
# --------------------
def generate_remediation_prompt(doc_text: str) -> str:
    finding_id, control_id = extract_finding_and_control_ids(doc_text)
    if not finding_id or not control_id:
        raise ValueError("Missing Finding ID or Control ID.")

    context = retrieve_vector_context(finding_id, control_id)
    prompt = prepare_llm_prompt(doc_text, finding_id, control_id, context)
    return prompt

# --------------------
# Example Simulation
# --------------------
if __name__ == "__main__":
    uploaded_remediation_doc = """Remediation Plan Submission:
Remediation ID: RMD-101
Finding ID: FND-1234
Control ID: AB-XX32.42.4-2
Details: Our team proposes implementing an automated log rotation and deletion process in alignment with retention requirements.
"""

    final_prompt = generate_remediation_prompt(uploaded_remediation_doc)
    print("\nGenerated Prompt for LLM:")
    print(final_prompt)






















def vector_search_by_ids(search_client, embedding, fnd_ids, abc_ids):
    filter_clauses = []
    if fnd_ids:
        fnd_filter = " or ".join([f"finding_id eq '{fid}'" for fid in fnd_ids])
        filter_clauses.append(f"({fnd_filter})")
    if abc_ids:
        abc_filter = " or ".join([f"control_id eq '{cid}'" for cid in abc_ids])
        filter_clauses.append(f"({abc_filter})")
    odata_filter = " and ".join(filter_clauses) if filter_clauses else None

    vector_query = [{
        "kind": "vector",
        "vector": embedding,
        "fields": "contentVector",
        "k_nearest_neighbors": 5
    }]

    results = search_client.search(
        search_text="",
        vector_queries=vector_query,
        select=["content", "sourcefile"],
        filter=odata_filter
    )
    return list(results)






(finding_id eq 'FND-1234' or finding_id eq 'FND-5678') and (control_id eq 'ABC-1111' or control_id eq 'ABC-2222')





from azure.search.documents import SearchClient
from azure.core.credentials import AzureKeyCredential

# --- Assume the below are already defined ---
endpoint = "https://<YOUR-SEARCH-SERVICE>.search.windows.net"
index_name = "<YOUR-INDEX-NAME>"
api_key = "<YOUR-ADMIN-KEY>"
search_client = SearchClient(endpoint, index_name, AzureKeyCredential(api_key))

embedding = [0.1, 0.2, 0.3, ...]   # Replace with your actual embedding

# --- Extract IDs from your document/text line ---
sample_text = "Here are FND-1234, FND-5678 and also ABC-1111, ABC-2222."
fnd_ids, abc_ids = extract_all_ids(sample_text)

# --- Dynamically build the OData filter for all IDs ---
filter_clauses = []
if fnd_ids:
    fnd_filter = " or ".join([f"finding_id eq '{fid}'" for fid in fnd_ids])
    filter_clauses.append(f"({fnd_filter})")
if abc_ids:
    abc_filter = " or ".join([f"control_id eq '{cid}'" for cid in abc_ids])
    filter_clauses.append(f"({abc_filter})")

odata_filter = " and ".join(filter_clauses) if filter_clauses else None

# --- Vector query definition ---
vector_query = [{
    "kind": "vector",
    "vector": embedding,
    "fields": "contentVector",      # Replace with your vector field name
    "k_nearest_neighbors": 5
}]

# --- Run the search with filter ---
results = search_client.search(
    search_text="",
    vector_queries=vector_query,
    select=["content", "sourcefile"],
    filter=odata_filter  # This filters by all extracted IDs
)

for result in results:
    print("Content:", result.get("content"))
    print("Sourcefile:", result.get("sourcefile"))
    print("---")
