# Streamlit_test
Streamlit_test



import streamlit as st
import yaml
from streamlit_flow import streamlit_flow
from streamlit_flow.elements import StreamlitFlowNode, StreamlitFlowEdge
from streamlit_flow.state import StreamlitFlowState
from streamlit_flow.layouts import TreeLayout

st.set_page_config("Data Product Flow", layout="wide")
st.title("Data Product Flow (YAML-based with Hyperlinks & Names)")

# Function to load YAML file
def load_yaml(file_path):
    with open(file_path, "r") as file:
        return yaml.safe_load(file)

# Function to parse YAML into StreamlitFlow objects
def parse_yaml(yaml_data):
    nodes = []
    edges = []

    # Extract outputPorts as nodes
    for port in yaml_data.get("outputPorts", []):
        node_id = port["id"]
        node_name = port.get("name", f"Node {node_id}")  # Default name if not provided
        url = port.get("url", "#")  # Use URL if available, otherwise use "#"
        
        # Create a clickable name using Markdown syntax
        content = f"[{node_name}]({url})\n({port['environment']})\nStatus: {port['status']}"
        
        node_type = "default"
        if port["status"] == "active":
            node_type = "input"
        elif port["status"] in ["deprecated", "retired"]:
            node_type = "output"

        # Create a node
        nodes.append(
            StreamlitFlowNode(
                node_id, (0, 0), {"content": content}, node_type, "right", "left"
            )
        )

    # Add edges from YAML
    for edge in yaml_data.get("edges", []):
        edges.append(
            StreamlitFlowEdge(
                edge["id"], edge["source"], edge["target"], animated=edge.get("animated", False)
            )
        )

    return StreamlitFlowState(nodes, edges)

# Load YAML and Initialize State
if "curr_state" not in st.session_state:
    yaml_data = load_yaml("flowchart.yaml")  # Load YAML file
    st.session_state.curr_state = parse_yaml(yaml_data)  # Parse and store in session state

# Button to reload YAML file
if st.button("Reload YAML"):
    yaml_data = load_yaml("flowchart.yaml")
    st.session_state.curr_state = parse_yaml(yaml_data)
    st.rerun()

# Render Flowchart
st.session_state.curr_state = streamlit_flow(
    "data_product_flow",
    st.session_state.curr_state,
    layout=TreeLayout(direction="right"),
    fit_view=True,
    height=500,
    enable_node_menu=True,
    enable_edge_menu=True,
    enable_pane_menu=True,
    show_minimap=True,
    hide_watermark=True,
    allow_new_edges=True,
    min_zoom=0.1,
)

# Display Nodes and Edges
st.write("### Nodes")
st.write(st.session_state.curr_state.nodes)

st.write("### Edges")
st.write(st.session_state.curr_state.edges)

st.write("### Selected Node/Edge")
st.write(st.session_state.curr_state.selected_id)








outputPorts:
  - id: "1"
    position: [0, 0]
    name: "Orders PII v1"
    description: "A retired PII dataset"
    environment: "prod"
    status: "retired"
    url: "https://example.com/orders_pii_v1"

  - id: "2"
    position: [1, 0]
    name: "Orders PII v2"
    description: "An active PII dataset"
    environment: "prod"
    status: "active"
    url: "https://example.com/orders_pii_v2"

  - id: "3"
    position: [2, 0]
    name: "Orders NPII v2"
    description: "A deprecated non-PII dataset"
    environment: "prod"
    status: "deprecated"
    url: "https://example.com/orders_npii_v2"

edges:
  - id: "1-2"
    source: "1"
    target: "2"
    animated: true

  - id: "2-3"
    source: "2"
    target: "3"
    animated: true
