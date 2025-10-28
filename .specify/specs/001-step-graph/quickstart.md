# Quickstart Guide: Patent Step Graph Extraction (Minimal Viable)

**Feature**: 001-step-graph  
**Last Updated**: 2025-10-28  
**Related**: [plan.md](./plan.md), [research.md](./research.md)

---

## Overview

This guide shows how to extract patent step graphs using a **human-in-the-loop workflow** with GitHub Copilot. You'll manually process 10-20 patents through 4 stages, using VS Code + Copilot for LLM-assisted analysis (no API calls).

**Philosophy**: Quality over automation. **You are the curator**, Copilot is your assistant.

---

## Prerequisites

### System Requirements

- **Operating System**: macOS, Linux, or Windows with WSL2
## Prerequisites

- **Python**: 3.12 or higher
- **Neo4j**: 5.x (optional - for visualization only)
- **VS Code**: With GitHub Copilot enabled
- **Memory**: Minimum 8GB RAM
- **Storage**: ~2GB free space for artifacts

### Required Software

1. **Python 3.12+**

   ```bash
   python --version  # Should show 3.12 or higher

2. **Neo4j 5.x**
   - **Recommended for researchers**: Neo4j Desktop
     - Download from https://neo4j.com/download/
     - Create local database (2GB heap is sufficient)
     - Easier visualization with built-in browser
   
   - **Alternative - Docker**:
   ```bash
   docker run -d \
     --name neo4j-step-graph \
     -p 7474:7474 -p 7687:7687 \
     -e NEO4J_AUTH=neo4j/password \
     neo4j:5-community3. **VS Code with GitHub Copilot**
   - Install VS Code: https://code.visualstudio.com/
   - Enable GitHub Copilot extension
   - **No API keys needed** - Copilot handles LLM calls

### Verify Setup

```bash
# Check Python
python --version

# Check Neo4j is running
# Option 1 (Desktop): Open Neo4j Desktop and start database
# Option 2 (Docker): docker ps | grep neo4j

# Check VS Code and Copilot
code --version
# Open VS Code → Extensions → Verify "GitHub Copilot" is enabled
```

---

## Installation

### 1. Clone Repository

```bash
git clone https://github.com/your-org/step-graph.git
cd step-graph
```

### 2. Create Virtual Environment

```bash
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
```

### 3. Install Minimal Dependencies

**No Poetry, no complex dependencies - just pip:**

```bash
# Install only essentials: Neo4j driver + CLI framework
pip install neo4j click

# Optional: For data validation (can skip for MVP)
pip install pydantic
```

**That's it!** No OpenAI SDK, no Anthropic SDK, no caching libraries.

### 4. Configure Environment

Create a `.env` file with **just Neo4j credentials**:

```bash
# Copy minimal template
cat > .env << EOF
# Neo4j Configuration
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=password123

# Logging (optional)
LOG_LEVEL=INFO
LOG_DIR=.logs
EOF
```

**No API keys needed** - Copilot handles LLM interactions.

### 5. Initialize Neo4j Schema

```bash
# Create minimal constraints and indexes
python -m step_graph init-db --neo4j-uri $NEO4J_URI

# Expected output:
# ✓ Created constraint: patent_id_unique
# ✓ Created constraint: step_id_unique
# ✓ Created index: step_run_id_idx
# ✓ Neo4j schema initialized
```

---

## Workflow Overview

You'll process patents through 4 stages, using CLI commands + Copilot assistance:

1. **Stage 1 (Automated)**: Parse patent JSON → Extract steps → Write to Neo4j
2. **Stage 2 (Copilot-Assisted)**: Characterize each step along dimensions (energy/matter/information)
3. **Stage 3 (Copilot-Assisted)**: Identify and split composite steps into atomic units
4. **Stage 4 (Manual + Copilot)**: Find similar steps across patents, create clusters

**Expected time**: ~30-40 minutes per patent (includes human review).

---

## Stage 1: Identify Steps (Automated)

### Select Sample Patents

Start with 5-10 patents from `IG_start/data/`:

```bash
# List available patents
ls IG_start/data/raw_patent_data/*.json | head -10

# Create a pilot batch
mkdir -p pilot-batch
cp IG_start/data/raw_patent_data/CN-100342579-C.json pilot-batch/
cp IG_start/data/raw_patent_data/CA-2895358-C.json pilot-batch/
# Add 3-8 more...
```

### Run Stage 1 Parser

```bash
# Generate a run ID for this batch
export RUN_ID=$(python -c "import uuid; print(uuid.uuid4())")
echo "Run ID: $RUN_ID"

# Parse patents and write to Neo4j
python -m step_graph identify \
  --input pilot-batch/ \
  --output .artifacts/$RUN_ID/ \
  --run-id $RUN_ID \
  --neo4j-uri bolt://localhost:7687

# Expected output (~5 min for 10 patents):
# Processing patents: 100%|███████████| 10/10 [00:15<00:00]
# ✓ Processed 10 patents, extracted 87 steps, 54 parameters
# ✓ Written JSONL to .artifacts/{run_id}/stage1/steps.jsonl
# ✓ Confirm write to Neo4j? [Y/n]: y
# ✓ Stage 1 complete
```

**What just happened**:
- Parsed `technical_process` field (split by `->`)
- Extracted `technical_parameters`
- Created Step and Patent nodes in Neo4j
- Saved JSONL files (source of truth)

### Verify in Neo4j Browser

1. Open Neo4j Browser: http://localhost:7474
2. Run query:

```cypher
MATCH (p:Patent)-[:HAS_STEP]->(s:Step {stage: "identification"})
WHERE s.run_id = "{your_run_id}"
RETURN p.patent_id, s.order_index, s.text_raw
ORDER BY s.order_index
LIMIT 25
```

**Manual validation**: Review ~10-15 steps to check parsing quality. Look for:
- Proper splitting at `->`
- Correct order_index
- Preserved Chinese/English text

---

## Stage 2: Encode Transformations (Copilot-Assisted)

### Open JSONL in VS Code

```bash
# Open Stage 1 output
code .artifacts/$RUN_ID/stage1/steps.jsonl
```

### Use Copilot Chat for Transformation Characterization

**Workflow for EACH step** (or batch of 5-10 similar steps):

1. **Select a step's `text_raw` field** in the JSONL file
2. **Open Copilot Chat** (`Ctrl+Shift+I` or `Cmd+Shift+I`)
3. **Use this prompt template** (saved in `copilot-prompts/stage2-transformation-encoding.md`):

```
Characterize this process step's transformations along three dimensions:

Step: "{paste step text here}"

Return JSON with this structure:
{
  "energy": {
    "from_form": "electrical|thermal|mechanical|chemical_potential|kinetic|other",
    "to_form": "electrical|thermal|mechanical|chemical_potential|kinetic|other",
    "magnitude": 0.0-1.0,
    "rationale": "1-sentence explanation of the transformation"
  },
  "matter": {
    "from_form": "separated_powders|liquid_ambient_temp|solid_crystalline|lithium_ions_dispersed|other",
    "to_form": "homogeneous_mixture|liquid_elevated_temp|liquid_solution|lithium_ions_intercalated|other",
    "magnitude": 0.0-1.0,
    "rationale": "1-sentence explanation of the transformation"
  },
  "information": {
    "from_form": "voltage_measurement|temperature_sensor|optical_signal|none|other",
    "to_form": "battery_state_data|control_feedback|concentration_data|none|other",
    "magnitude": 0.0-1.0,
    "rationale": "1-sentence explanation of the transformation"
  }
}

Magnitude scale (degree of transformation):
- 0.0-0.2: Minimal transformation (form barely changes)
- 0.3-0.5: Partial transformation (moderate conversion)
- 0.6-0.8: Significant transformation (substantial conversion)
- 0.9-1.0: Near-complete transformation (almost full conversion)

Note: If no transformation occurs in a dimension, use same from_form and to_form with magnitude 0.0-0.1
```

4. **Review Copilot's response**
5. **Adjust from_form/to_form and magnitudes if needed** (you're the domain expert!)
6. **Copy JSON output** and add `transformation_tags` field to the step in JSONL

**Example edit**:

```jsonl
{"id": "step-uuid-123", "text_raw": "Heat to 95°C", "transformation_tags": null}
↓
{"id": "step-uuid-123", "text_raw": "Heat to 95°C", "transformation_tags": {"energy": {"from_form": "electrical", "to_form": "thermal", "magnitude": 0.85, "rationale": "Electrical heating element converts electricity to thermal energy"}, "matter": {"from_form": "liquid_ambient_temp", "to_form": "liquid_elevated_temp", "magnitude": 0.7, "rationale": "Liquid temperature increased significantly"}, "information": {"from_form": "none", "to_form": "none", "magnitude": 0.0, "rationale": "No information processing in this step"}}}
```

### Batch Processing with Copilot Edits

For repetitive steps (e.g., 10 similar heating steps):

1. **Open `steps.jsonl`**
2. **Use Copilot Edits** (`Ctrl+Shift+P` → "Copilot: Open Inline Chat")
3. **Prompt**: "Add transformation_tags field to all steps where text_raw contains 'heat' or '加热'"
4. **Review suggestions** - accept/reject/modify each

### Write Back to Neo4j (Optional)

```bash
# After manually editing JSONL, optionally import Stage 2 data to Neo4j
python -m step_graph encode \
  --run-id $RUN_ID \
  --input .artifacts/$RUN_ID/stage1/steps.jsonl \
  --neo4j-uri bolt://localhost:7687

# Expected output:
# Reading edited JSONL...
# ✓ Updating 87 steps with transformation_tags
# ✓ Confirm write to Neo4j? [Y/n]: y
# ✓ Stage 2 complete
```

**Time estimate**: 20-25 min for 10-15 steps (with Copilot assistance).

---

## Stage 3: Atomize Steps (Copilot-Assisted)

### Identify Composite Steps

**Copilot Chat workflow**:

1. **Select 5-10 steps** from JSONL
2. **Prompt**: "Which of these steps are composite (contain multiple actions)? List them."

**Example Copilot response**:
```
Composite steps:
- "Mix powder A and B, then heat to 95°C" → 2 actions
- "Charge at 0.5C, discharge at 1C, repeat 3 cycles" → 3 actions
```

3. **For each composite step**, ask Copilot to split:

**Prompt**:
```
Split this composite step into atomic sub-steps:

Original: "Mix powder A and B, then heat to 95°C"

Return JSON array:
[
  {"text": "Mix powder A and B", "order": 1},
  {"text": "Heat mixture to 95°C", "order": 2}
]
```

4. **Review and adjust** (ensure logical atomicity)
5. **Create new step entries** in JSONL with `COMPOSED_OF` relationship

### Edit JSONL for Atomicization

**Example**:

```jsonl
# Original composite step
{"id": "step-uuid-456", "text_raw": "Mix powder A and B, then heat to 95°C", "stage": "encoding"}

# Add atomic sub-steps
{"id": "step-uuid-456-atom-1", "text_raw": "Mix powder A and B", "stage": "atomicized", "parent_id": "step-uuid-456", "order": 1}
{"id": "step-uuid-456-atom-2", "text_raw": "Heat mixture to 95°C", "stage": "atomicized", "parent_id": "step-uuid-456", "order": 2}
```

### Write Atomicized Steps to Neo4j

```bash
python -m step_graph atomize \
  --run-id $RUN_ID \
  --input .artifacts/$RUN_ID/stage2/atomicized_steps.jsonl \
  --neo4j-uri bolt://localhost:7687

# Expected output:
# ✓ Created 23 atomic steps with COMPOSED_OF relationships
# ✓ Stage 3 complete
```

**Time estimate**: 10-15 min for 10-20 steps.

---

## Stage 4: Cluster Steps (Manual + Copilot)

### Find Candidate Clusters

**Copilot Chat workflow**:

1. **Export all atomicized steps** to a readable format:

```bash
# Query Neo4j to export steps
python -m step_graph export \
  --run-id $RUN_ID \
  --stage atomicized \
  --format txt \
  --output .artifacts/$RUN_ID/stage4/steps_for_clustering.txt
```

2. **Open `steps_for_clustering.txt` in VS Code**
3. **Use Copilot Chat** to identify similar groups:

**Prompt**:
```
Analyze these process steps and identify groups of semantically similar steps:

{paste 20-30 steps here}

Return groups with similarity score (0-1). Only include groups where score > 0.85.

Format:
Group 1 (similarity: 0.92):
- Step ID: step-uuid-123, Text: "Heat to 95°C"
- Step ID: step-uuid-789, Text: "Heat to 100°C"
```

4. **Review Copilot's suggestions**
5. **Manually validate each cluster** (you decide final grouping)

### Create Clusters in JSONL

**Example**:

```jsonl
# Cluster node
{"cluster_id": "cluster-heating-thermal", "run_id": "{run_id}", "cluster_method": "manual-copilot", "member_count": 5}

# Add IN_CLUSTER relationships to step entries
{"id": "step-uuid-123", "cluster_id": "cluster-heating-thermal"}
{"id": "step-uuid-789", "cluster_id": "cluster-heating-thermal"}
```

### Write Clusters to Neo4j

```bash
python -m step_graph cluster \
  --run-id $RUN_ID \
  --input .artifacts/$RUN_ID/stage4/clusters.jsonl \
  --neo4j-uri bolt://localhost:7687

# Expected output:
# ✓ Created 12 clusters with 67 IN_CLUSTER relationships
# ✓ Stage 4 complete
```

**Time estimate**: 5-10 min per cluster group (~10-15 clusters total).

---

## Visualizing Results

### Neo4j Browser Queries

**View full pipeline for one patent**:
```cypher
MATCH path = (p:Patent {patent_id: "CN-100342579-C"})-[*1..4]-(n)
WHERE p.run_id = "{your_run_id}"
RETURN path
LIMIT 50
```

**View all clusters**:
```cypher
MATCH (c:Cluster)<-[:IN_CLUSTER]-(s:Step)
WHERE c.run_id = "{your_run_id}"
RETURN c.cluster_id, collect(s.text_raw) AS steps
ORDER BY c.cluster_id
```

**Find steps with high energy transformation**:
```cypher
MATCH (s:Step)
WHERE s.run_id = "{your_run_id}"
  AND s.transformation_tags.energy.magnitude > 0.8
RETURN s.text_raw, 
       s.transformation_tags.energy.from_form AS from_form,
       s.transformation_tags.energy.to_form AS to_form,
       s.transformation_tags.energy.magnitude AS magnitude
ORDER BY s.transformation_tags.energy.magnitude DESC
```

**Find specific transformation patterns (e.g., electrical→chemical)**:
```cypher
MATCH (s:Step)
WHERE s.run_id = "{your_run_id}"
  AND s.transformation_tags.energy.from_form = "electrical"
  AND s.transformation_tags.energy.to_form = "chemical_potential"
RETURN s.text_raw, s.transformation_tags.energy.magnitude
ORDER BY s.transformation_tags.energy.magnitude DESC
```

### Export for Analysis

```bash
# Export to CSV for Excel/Pandas
python -m step_graph export \
  --run-id $RUN_ID \
  --format csv \
  --output final_results.csv
```

---

## Common Workflows

### Re-Process a Single Patent

```bash
# If you want to redo one patent's encoding
python -m step_graph identify \
  --input pilot-batch/CN-100342579-C.json \
  --output .artifacts/$RUN_ID/ \
  --run-id $RUN_ID \
  --neo4j-uri bolt://localhost:7687 \
  --force  # Overwrite existing
```

### Rollback a Run

```bash
# Delete all data for a run_id (if you made mistakes)
python -m step_graph rollback \
  --run-id $RUN_ID \
  --neo4j-uri bolt://localhost:7687

# WARNING: This deletes everything!
```

### Share Your Prompt Templates

```bash
# Save your effective Copilot prompts
mkdir -p copilot-prompts/
# Document prompts that worked well in markdown files
# See copilot-prompts/stage2-encoding-prompt.md for examples
```

---

## Troubleshooting

### Neo4j Connection Errors

**Error**: `Could not connect to Neo4j at bolt://localhost:7687`

**Solutions**:
1. Check Neo4j Desktop is running (green status)
2. Or check Docker: `docker ps | grep neo4j`
3. Verify `.env` has correct URI and credentials

### Copilot Not Responding Effectively

**Issue**: Copilot gives generic or incorrect transformation characterizations

**Solutions**:
1. **Add few-shot examples** to your prompt (2-3 correct examples)
2. **Narrow context**: Only ask about 1-5 steps at a time
3. **Iterate**: Use follow-up questions to refine Copilot's answers
4. **Remember**: You are the expert - Copilot is a suggestion tool

**Issue**: Copilot suggests `from_form = to_form` with high magnitude

**Solutions**:
1. This is contradictory (no transformation but high magnitude)
2. Either change the forms to show actual transformation, or reduce magnitude to <0.2
3. Example fix: "electrical → electrical, magnitude 0.8" should be "electrical → thermal, magnitude 0.8" for a heating process

### JSONL Editing Mistakes

**Issue**: Accidentally broke JSON syntax while editing

**Solution**:
```bash
# Validate JSONL before importing
python -c "import json; open('.artifacts/$RUN_ID/stage2/steps.jsonl').readlines()"
# If error, fix syntax in VS Code (Copilot can help: "Fix JSON syntax errors in this file")
```

### Too Many Steps to Process

**Issue**: 10 patents = 100+ steps, too tedious

**Solutions**:
1. **Reduce scope**: Start with 5 patents (~50 steps)
2. **Batch similar steps**: Use Copilot Edits for repetitive patterns
3. **Focus on diversity**: Pick 1-2 steps per patent type (ensure coverage, not exhaustiveness)

---

## Performance Tips

1. **Use Copilot Chat history**: Re-use effective prompts by scrolling up in Chat
2. **Batch similar steps**: Group heating steps, mixing steps, etc. for faster Copilot processing
3. **Save prompt templates**: Document working prompts in `copilot-prompts/`
4. **Validate incrementally**: Check Neo4j after each stage before moving forward
5. **Don't aim for perfection**: 85% accuracy with 20 patents is better than 95% with 2 patents

---

## Success Metrics (MVP)

- **Throughput**: 10-20 patents in 2 weeks
- **Accuracy**: >85% correct transformation characterizations (manual validation)
- **Efficiency**: <30 min per patent with Copilot assistance
- **Reproducibility**: Documented prompts enable others to replicate workflow

---

## Next Steps

1. **Process your pilot batch** (5-10 patents)
2. **Refine prompt templates** based on what works
3. **Scale to 20 patents** once workflow is smooth
4. **Analyze results** with Neo4j Cypher queries (or just work with JSONL files)
5. **Iterate**: Adjust transformation taxonomy if needed

---

## Resources

- **Full plan**: [plan.md](./plan.md)
- **Research findings**: [research.md](./research.md)
- **Data schema**: [data-model.md](./data-model.md)
- **Copilot prompts**: `copilot-prompts/` directory
- **Neo4j docs**: https://neo4j.com/docs/
- **Copilot usage tips**: https://docs.github.com/en/copilot

---

**Last Updated**: 2025-10-28  
**Version**: 1.0.0 (Minimal Viable MVP)  
**Philosophy**: Human curator + Copilot assistant = Quality over automation
