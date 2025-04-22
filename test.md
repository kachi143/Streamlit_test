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
