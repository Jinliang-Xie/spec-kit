# Codex Implementation Prompt: Patent Step Graph Extraction System

## Mission

Implement a complete, production-ready Python CLI system that extracts, characterizes, atomizes, and clusters technical "Steps" from patent JSON data, with optional Neo4j visualization. Use this prompt to guide full implementation from scratch to deployment-ready code.

---

## Context: What You're Building

You are building a **human-in-the-loop patent analysis pipeline** with four progressive stages:

1. **Stage 1 (Identification)**: Parse patent JSON → extract Steps with parameters → write JSONL + optional Neo4j
2. **Stage 2 (Encoding)**: Manually characterize Steps along transformation dimensions (energy/matter/information form conversions) using Copilot Chat assistance
3. **Stage 3 (Atomicization)**: Split composite Steps into atomic units with Copilot suggestions → track provenance
4. **Stage 4 (Clustering)**: Aggregate similar Steps across patents using manual curation + Copilot analysis

**Key Design Principles**:
- **JSONL-First Architecture**: All stage outputs write to JSONL files as source of truth; Neo4j is optional for visualization
- **No LLM API Calls**: Use GitHub Copilot/Codex for manual curation in Stages 2-4 (human-in-the-loop workflow)
- **Modular CLI**: Each stage is an independent Click command with `--run-id` versioning
- **Quality over Speed**: Manual review ensures >85% accuracy; target 30-40 min per patent for full pipeline

---

## Technical Stack

**Language**: Python 3.12+ (current environment: Python 3.12.10)  
**CLI Framework**: Click 8.3.x (latest stable: [8.3.0](https://click.palletsprojects.com/en/stable/))  
**Database**: Neo4j 5.x (optional - for visualization only, docs: [neo4j.com/docs](https://neo4j.com/docs/))  
**Testing**: pytest with >60% coverage for parsers/utils  
**Development**: VS Code + GitHub Copilot  
**Dependencies**:
```toml
[tool.poetry.dependencies]
python = "^3.12"
click = "^8.3.0"
neo4j = {version = "^5.0.0", optional = true}
pydantic = {version = "^2.0.0", optional = true}

[tool.poetry.dev-dependencies]
pytest = "^7.4.0"
pytest-cov = "^4.1.0"
black = "^23.0.0"
ruff = "^0.1.0"
```

---

## Project Structure

Create this exact directory hierarchy:

```
step_graph/
├── __init__.py                 # Package metadata + version
├── __main__.py                 # Entry point: python -m step_graph
├── parsers/
│   ├── __init__.py
│   ├── lane_a.py              # Parse technical_process (-> delimiter)
│   ├── lane_b.py              # Fallback: parse raw text (future)
│   └── normalizer.py          # Handle 0/empty/"null" → unknown
├── neo4j/
│   ├── __init__.py
│   ├── client.py              # Neo4j driver wrapper (optional)
│   ├── schema.py              # Constraints + indexes
│   └── queries.py             # Cypher templates by stage
├── cli/
│   ├── __init__.py
│   ├── identify.py            # Stage 1 command
│   ├── encode.py              # Stage 2 helper (manual workflow)
│   ├── atomize.py             # Stage 3 helper (manual workflow)
│   ├── cluster.py             # Stage 4 helper (manual workflow)
│   └── export.py              # JSONL → Neo4j import utility
└── utils/
    ├── __init__.py
    ├── ids.py                 # UUID v4 generation
    ├── config.py              # Environment variables
    └── artifacts.py           # JSONL read/write with UTF-8

tests/
├── test_parsers/
│   ├── test_lane_a.py
│   └── test_normalizer.py
├── test_neo4j/
│   └── test_queries.py
└── fixtures/
    └── sample_patents.json     # 3-5 minimal test patents

.artifacts/
└── <run_id>/
    ├── stage1_identification/
    │   ├── steps.jsonl
    │   └── parameters.jsonl
    ├── stage2_encoding/
    │   └── encoded_steps.jsonl
    ├── stage3_atomicization/
    │   └── atomic_steps.jsonl
    └── stage4_clustering/
        └── clusters.jsonl

.specify/specs/001-step-graph/
├── spec.md                     # Feature specification (READ THIS FIRST)
├── plan.md                     # Implementation plan (READ THIS SECOND)
├── tasks.md                    # Task breakdown (USE AS CHECKLIST)
├── quickstart.md               # User-facing workflow guide
└── copilot-prompts/            # Templates for Stages 2-4
    ├── stage2-transformation-encoding.md
    ├── stage3-atomicization.md
    └── stage4-clustering.md
```

---

## Implementation Roadmap

### Phase 0: Setup (1 hour)

**Files to Create**:
1. `pyproject.toml` with dependencies above
2. `step_graph/__init__.py` with `__version__ = "0.1.0"`
3. `step_graph/__main__.py`:
```python
import click
from step_graph.cli import identify, encode, atomize, cluster, export

@click.group()
@click.version_option()
def cli():
    """Patent Step Graph Extraction Pipeline"""
    pass

cli.add_command(identify.identify)
cli.add_command(encode.encode)
cli.add_command(atomize.atomize)
cli.add_command(cluster.cluster)
cli.add_command(export.export_to_neo4j)

if __name__ == "__main__":
    cli()
```

4. `.env.example`:
```bash
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=password
```

5. `.gitignore`:
```
.env
.artifacts/
__pycache__/
*.pyc
.pytest_cache/
.coverage
htmlcov/
dist/
build/
*.egg-info/
```

**Verification**: Run `python -m step_graph --help` → should show CLI help

---

### Phase 1: Core Utilities (2 hours)

**File: `step_graph/utils/ids.py`**
```python
import uuid
from datetime import datetime

def generate_step_id() -> str:
    """Generate UUID v4 for Step entities"""
    return str(uuid.uuid4())

def generate_run_id() -> str:
    """Generate timestamped run ID: run_YYYYMMDD_HHMMSS_UUID"""
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    short_uuid = str(uuid.uuid4())[:8]
    return f"run_{timestamp}_{short_uuid}"
```

**File: `step_graph/utils/config.py`**
```python
import os
from pathlib import Path
from dotenv import load_dotenv

load_dotenv()

NEO4J_URI = os.getenv("NEO4J_URI", "bolt://localhost:7687")
NEO4J_USER = os.getenv("NEO4J_USER", "neo4j")
NEO4J_PASSWORD = os.getenv("NEO4J_PASSWORD", "password")
ARTIFACTS_DIR = Path(os.getenv("ARTIFACTS_DIR", ".artifacts"))

def ensure_artifacts_dir(run_id: str, stage: str) -> Path:
    """Create artifacts directory for run_id/stage"""
    path = ARTIFACTS_DIR / run_id / stage
    path.mkdir(parents=True, exist_ok=True)
    return path
```

**File: `step_graph/utils/artifacts.py`**
```python
import json
from pathlib import Path
from typing import List, Dict, Any

def write_jsonl(data: List[Dict[str, Any]], filepath: Path) -> None:
    """Write list of dicts to JSONL with UTF-8 encoding"""
    with open(filepath, 'w', encoding='utf-8') as f:
        for item in data:
            f.write(json.dumps(item, ensure_ascii=False) + '\n')

def read_jsonl(filepath: Path) -> List[Dict[str, Any]]:
    """Read JSONL file, return list of dicts"""
    with open(filepath, 'r', encoding='utf-8') as f:
        return [json.loads(line) for line in f if line.strip()]

def read_json(filepath: Path) -> Dict[str, Any]:
    """Read single JSON file"""
    with open(filepath, 'r', encoding='utf-8') as f:
        return json.load(f)
```

**Tests**: Create `tests/test_utils.py` with tests for UUID generation, JSONL I/O

---

### Phase 2: Stage 1 Implementation (6 hours)

**File: `step_graph/parsers/lane_a.py`**
```python
from typing import List, Dict, Any
from step_graph.utils.ids import generate_step_id

def parse_technical_process(text: str, patent_id: str, run_id: str) -> List[Dict[str, Any]]:
    """
    Parse technical_process field with -> delimiter.
    
    Handle edge cases:
    - Empty/null/"null" → return []
    - Multiple -> in one line → split by ->
    - Whitespace normalization
    """
    if not text or text.strip() in ["", "null", "0"]:
        return []
    
    # Split by -> delimiter, clean whitespace
    raw_steps = [s.strip() for s in text.split('->')]
    
    steps = []
    for i, text_raw in enumerate(raw_steps):
        if text_raw:  # Skip empty strings
            step = {
                "id": generate_step_id(),
                "patent_id": patent_id,
                "run_id": run_id,
                "order_index": i + 1,
                "text_raw": text_raw,
                "stage": "identification",
                "source_field": "technical_process"
            }
            steps.append(step)
    
    return steps

def parse_technical_parameters(
    params_obj: Dict[str, Any], 
    patent_id: str, 
    run_id: str
) -> List[Dict[str, Any]]:
    """
    Extract parameters from technical_parameters object.
    
    Expected structure:
    {
      "nominal_capacity": {"unit": "Ah", "value": "1", "actions": "..."},
      "charge_rate": {"value": "0", ...}  # 0 = not specified
    }
    
    Rules:
    - value="1" → has_value=True (parameter is specified)
    - value="0" → has_value=False (parameter not specified)
    - unit="" or "null" → raw_unit="unknown"
    """
    from step_graph.parsers.normalizer import normalize_parameter
    
    parameters = []
    for param_name, param_data in params_obj.items():
        if isinstance(param_data, dict):
            normalized = normalize_parameter(param_name, param_data)
            normalized["patent_id"] = patent_id
            normalized["run_id"] = run_id
            normalized["stage"] = "identification"
            parameters.append(normalized)
    
    return parameters
```

**File: `step_graph/parsers/normalizer.py`**
```python
from typing import Dict, Any
from step_graph.utils.ids import generate_step_id

def normalize_parameter(param_name: str, param_data: Dict[str, Any]) -> Dict[str, Any]:
    """
    Normalize parameter data handling 0/empty/"null" as unknown.
    
    Constitution requirement: 0/empty/"null" → unknown (not mathematical zero)
    """
    raw_value = param_data.get("value", "0")
    raw_unit = param_data.get("unit", "")
    actions = param_data.get("actions", "")
    
    # Handle has_value flag
    has_value = raw_value == "1"
    
    # Normalize unit
    if not raw_unit or raw_unit.strip() in ["", "null", "0"]:
        raw_unit = "unknown"
    
    return {
        "id": generate_step_id(),
        "param_name": param_name,
        "raw_value": raw_value,
        "raw_unit": raw_unit,
        "has_value": has_value,
        "actions": actions,
        "source_field": "technical_parameters"
    }
```

**File: `step_graph/cli/identify.py`**
```python
import click
from pathlib import Path
import json
from step_graph.parsers.lane_a import parse_technical_process, parse_technical_parameters
from step_graph.utils.ids import generate_run_id
from step_graph.utils.config import ensure_artifacts_dir
from step_graph.utils.artifacts import write_jsonl, read_json

@click.command()
@click.option('--input', '-i', type=click.Path(exists=True), required=True,
              help='Directory containing patent JSON files')
@click.option('--output', '-o', type=click.Path(), default='.artifacts',
              help='Output directory for JSONL artifacts')
@click.option('--run-id', type=str, default=None,
              help='Run ID (auto-generated if not provided)')
@click.option('--neo4j/--no-neo4j', default=False,
              help='Import to Neo4j after processing (requires --neo4j-uri)')
@click.option('--neo4j-uri', type=str, envvar='NEO4J_URI',
              help='Neo4j URI (default: from .env)')
def identify(input, output, run_id, neo4j, neo4j_uri):
    """Stage 1: Identify Steps and Parameters from patent JSON"""
    
    # Generate run ID if not provided
    if not run_id:
        run_id = generate_run_id()
        click.echo(f"Generated run_id: {run_id}")
    
    # Setup output directory
    output_dir = ensure_artifacts_dir(run_id, "stage1_identification")
    
    # Process all JSON files in input directory
    input_path = Path(input)
    json_files = list(input_path.glob("*.json"))
    
    if not json_files:
        click.echo(f"No JSON files found in {input_path}", err=True)
        return
    
    all_steps = []
    all_params = []
    
    with click.progressbar(json_files, label='Processing patents') as files:
        for json_file in files:
            patent_data = read_json(json_file)
            patent_id = json_file.stem  # Use filename as patent_id
            
            # Extract steps
            technical_process = patent_data.get("technical_process", "")
            steps = parse_technical_process(technical_process, patent_id, run_id)
            all_steps.extend(steps)
            
            # Extract parameters
            params_obj = patent_data.get("technical_parameters", {})
            params = parse_technical_parameters(params_obj, patent_id, run_id)
            all_params.extend(params)
    
    # Write JSONL outputs
    steps_file = output_dir / "steps.jsonl"
    params_file = output_dir / "parameters.jsonl"
    
    write_jsonl(all_steps, steps_file)
    write_jsonl(all_params, params_file)
    
    click.echo(f"\n✅ Stage 1 complete:")
    click.echo(f"   Steps: {len(all_steps)} → {steps_file}")
    click.echo(f"   Parameters: {len(all_params)} → {params_file}")
    
    # Optional Neo4j import
    if neo4j:
        if not neo4j_uri:
            click.echo("⚠️  --neo4j flag set but no Neo4j URI configured", err=True)
            return
        
        from step_graph.cli.export import import_stage1_to_neo4j
        import_stage1_to_neo4j(run_id, neo4j_uri)
        click.echo("✅ Imported to Neo4j")
```

**Tests**: Create `tests/test_parsers/test_lane_a.py`:
```python
import pytest
from step_graph.parsers.lane_a import parse_technical_process, parse_technical_parameters

def test_parse_simple_process():
    text = "步骤A -> 步骤B -> 步骤C"
    steps = parse_technical_process(text, "TEST-001", "run_test")
    assert len(steps) == 3
    assert steps[0]["order_index"] == 1
    assert steps[0]["text_raw"] == "步骤A"
    assert steps[0]["stage"] == "identification"

def test_parse_empty_process():
    for text in ["", "null", "0", None]:
        steps = parse_technical_process(text, "TEST-002", "run_test")
        assert steps == []

def test_parse_parameters():
    params_obj = {
        "nominal_capacity": {"unit": "Ah", "value": "1", "actions": "充电至100%"},
        "charge_rate": {"unit": "C", "value": "0", "actions": ""}
    }
    params = parse_technical_parameters(params_obj, "TEST-001", "run_test")
    assert len(params) == 2
    assert params[0]["param_name"] == "nominal_capacity"
    assert params[0]["has_value"] is True
    assert params[1]["has_value"] is False
```

---

### Phase 3: Neo4j Integration (Optional, 4 hours)

**File: `step_graph/neo4j/client.py`**
```python
from neo4j import GraphDatabase
from typing import Optional
import click

class Neo4jClient:
    def __init__(self, uri: str, user: str, password: str):
        self.driver = GraphDatabase.driver(uri, auth=(user, password))
    
    def close(self):
        self.driver.close()
    
    def verify_connectivity(self) -> bool:
        """Test Neo4j connection"""
        try:
            with self.driver.session() as session:
                result = session.run("RETURN 1 AS test")
                return result.single()["test"] == 1
        except Exception as e:
            click.echo(f"Neo4j connection failed: {e}", err=True)
            return False
    
    def execute_query(self, query: str, parameters: Optional[dict] = None):
        """Execute Cypher query with parameters"""
        with self.driver.session() as session:
            return session.run(query, parameters or {})
    
    def import_steps(self, steps: list, run_id: str):
        """Import Step nodes with HAS_STEP and NEXT relationships"""
        from step_graph.neo4j.queries import CREATE_STEP_BATCH
        
        with self.driver.session() as session:
            session.run(CREATE_STEP_BATCH, {"steps": steps, "run_id": run_id})
    
    def import_parameters(self, parameters: list, run_id: str):
        """Import Parameter nodes with HAS_PARAM relationships"""
        from step_graph.neo4j.queries import CREATE_PARAM_BATCH
        
        with self.driver.session() as session:
            session.run(CREATE_PARAM_BATCH, {"parameters": parameters, "run_id": run_id})
```

**File: `step_graph/neo4j/queries.py`**
```python
# Stage 1: Identification
CREATE_STEP_BATCH = """
UNWIND $steps AS step
MERGE (p:Patent {id: step.patent_id})
CREATE (s:Step {
    id: step.id,
    patent_id: step.patent_id,
    run_id: $run_id,
    order_index: step.order_index,
    text_raw: step.text_raw,
    stage: 'identification',
    source_field: step.source_field
})
MERGE (p)-[:HAS_STEP]->(s)

// Create NEXT relationships
WITH step
MATCH (s:Step {id: step.id})
MATCH (prev:Step {
    patent_id: step.patent_id,
    order_index: step.order_index - 1,
    run_id: $run_id
})
MERGE (prev)-[:NEXT]->(s)
"""

CREATE_PARAM_BATCH = """
UNWIND $parameters AS param
MATCH (p:Patent {id: param.patent_id})
CREATE (par:Parameter {
    id: param.id,
    param_name: param.param_name,
    raw_value: param.raw_value,
    raw_unit: param.raw_unit,
    has_value: param.has_value,
    actions: param.actions,
    run_id: $run_id,
    stage: 'identification'
})
MERGE (p)-[:HAS_PARAM]->(par)
"""

# Visualization queries
VISUALIZE_STAGE1 = """
MATCH (p:Patent)-[:HAS_STEP]->(s:Step)
WHERE s.run_id = $run_id AND s.stage = 'identification'
OPTIONAL MATCH (s)-[:NEXT]->(next:Step)
OPTIONAL MATCH (p)-[:HAS_PARAM]->(param:Parameter)
RETURN p, s, next, param
LIMIT 100
"""
```

**File: `step_graph/cli/export.py`**
```python
import click
from step_graph.neo4j.client import Neo4jClient
from step_graph.utils.config import NEO4J_URI, NEO4J_USER, NEO4J_PASSWORD, ARTIFACTS_DIR
from step_graph.utils.artifacts import read_jsonl

@click.command()
@click.option('--run-id', required=True, help='Run ID to export')
@click.option('--stage', type=click.Choice(['1', '2', '3', '4']), default='1',
              help='Stage to export (1=identification, 2=encoding, etc.)')
@click.option('--uri', default=NEO4J_URI, help='Neo4j URI')
def export_to_neo4j(run_id, stage, uri):
    """Export JSONL artifacts to Neo4j"""
    
    stage_map = {
        '1': 'stage1_identification',
        '2': 'stage2_encoding',
        '3': 'stage3_atomicization',
        '4': 'stage4_clustering'
    }
    
    stage_dir = ARTIFACTS_DIR / run_id / stage_map[stage]
    
    if not stage_dir.exists():
        click.echo(f"Stage directory not found: {stage_dir}", err=True)
        return
    
    # Connect to Neo4j
    client = Neo4jClient(uri, NEO4J_USER, NEO4J_PASSWORD)
    
    if not client.verify_connectivity():
        click.echo("Failed to connect to Neo4j", err=True)
        return
    
    # Import Stage 1
    if stage == '1':
        steps = read_jsonl(stage_dir / "steps.jsonl")
        params = read_jsonl(stage_dir / "parameters.jsonl")
        
        client.import_steps(steps, run_id)
        client.import_parameters(params, run_id)
        
        click.echo(f"✅ Imported {len(steps)} steps and {len(params)} parameters")
    
    client.close()

def import_stage1_to_neo4j(run_id: str, uri: str):
    """Helper function called by identify command"""
    # (Same logic as above, extracted for reuse)
    pass
```

---

### Phase 4: Stages 2-4 Workflow Helpers (3 hours)

**File: `.specify/specs/001-step-graph/copilot-prompts/stage2-transformation-encoding.md`**
```markdown
# Stage 2: Transformation Encoding - Copilot Prompt

## Task
Characterize this Step along three transformation dimensions:

1. **Energy Transformation**: What energy form conversion occurs? (e.g., electrical→thermal, mechanical→electrical)
2. **Matter Transformation**: What material form conversion occurs? (e.g., solid→liquid, ambient_temp→elevated_temp)
3. **Information Transformation**: What information form conversion occurs? (e.g., sensor_reading→state_data, raw_signal→processed_metric)

## Input
Step text: `{STEP_TEXT}`

## Output Format (JSON)
```json
{
  "step_id": "{STEP_ID}",
  "transformation_tags": {
    "energy": {
      "from_form": "electrical|thermal|mechanical|chemical|...",
      "to_form": "electrical|thermal|mechanical|chemical|...",
      "magnitude": 0.85,
      "rationale": "Explanation of transformation"
    },
    "matter": {
      "from_form": "liquid_ambient_temp|solid|gas|...",
      "to_form": "liquid_elevated_temp|solid|gas|...",
      "magnitude": 0.7,
      "rationale": "Explanation"
    },
    "information": {
      "from_form": "voltage_measurement|...",
      "to_form": "battery_state_data|...",
      "magnitude": 0.5,
      "rationale": "Explanation"
    }
  }
}
```

## Magnitude Scale
- 0.0-0.3: Minor transformation
- 0.4-0.6: Moderate transformation
- 0.7-1.0: Major transformation

## Examples
[Include 3-5 domain-specific examples from battery patents]

## Copilot Usage
1. Load `.artifacts/{run_id}/stage1_identification/steps.jsonl`
2. For each step, use Copilot Chat with this prompt template
3. Save encoded results to `.artifacts/{run_id}/stage2_encoding/encoded_steps.jsonl`
4. Estimated time: 5-10 min per step (for 10-20 steps = 1-3 hours per patent)
```

**File: `step_graph/cli/encode.py`**
```python
import click
from pathlib import Path

@click.command()
@click.option('--run-id', required=True, help='Run ID from Stage 1')
def encode(run_id):
    """Stage 2: Encode transformations (human-in-the-loop with Copilot)"""
    
    prompt_file = Path(".specify/specs/001-step-graph/copilot-prompts/stage2-transformation-encoding.md")
    
    if not prompt_file.exists():
        click.echo(f"Prompt template not found: {prompt_file}", err=True)
        return
    
    click.echo("📋 Stage 2: Manual Transformation Encoding")
    click.echo("\n1. Open .artifacts/{run_id}/stage1_identification/steps.jsonl")
    click.echo("2. Use Copilot Chat with the prompt template:")
    click.echo(f"   {prompt_file}")
    click.echo("3. Save results to .artifacts/{run_id}/stage2_encoding/encoded_steps.jsonl")
    click.echo("\n⏱️  Estimated time: 5-10 min per step")
    click.echo("\nSee quickstart.md for detailed workflow.")
```

*(Similar structure for `atomize.py` and `cluster.py` commands)*

---

### Phase 5: Testing & Documentation (4 hours)

**File: `tests/test_integration/test_e2e.py`**
```python
import pytest
from pathlib import Path
from click.testing import CliRunner
from step_graph.cli.identify import identify

@pytest.fixture
def sample_patent_dir(tmp_path):
    """Create sample patent JSON for testing"""
    patent = {
        "technical_process": "准备电极材料 -> 涂布 -> 干燥 -> 压延",
        "technical_parameters": {
            "nominal_capacity": {"unit": "Ah", "value": "1", "actions": "充电至100%"}
        }
    }
    
    patent_file = tmp_path / "TEST-001.json"
    import json
    with open(patent_file, 'w', encoding='utf-8') as f:
        json.dump(patent, f, ensure_ascii=False)
    
    return tmp_path

def test_stage1_end_to_end(sample_patent_dir, tmp_path):
    """Test Stage 1 CLI command with sample data"""
    runner = CliRunner()
    
    result = runner.invoke(identify, [
        '--input', str(sample_patent_dir),
        '--output', str(tmp_path / '.artifacts'),
        '--run-id', 'test_run_001'
    ])
    
    assert result.exit_code == 0
    assert "✅ Stage 1 complete" in result.output
    
    # Verify JSONL outputs
    steps_file = tmp_path / '.artifacts/test_run_001/stage1_identification/steps.jsonl'
    assert steps_file.exists()
    
    from step_graph.utils.artifacts import read_jsonl
    steps = read_jsonl(steps_file)
    assert len(steps) == 4
    assert steps[0]["text_raw"] == "准备电极材料"
```

**File: `README.md`** (Update with installation and usage):
```markdown
# Patent Step Graph Extraction System

Extract, characterize, atomize, and cluster technical steps from patent JSON data.

## Quick Start

### Installation
```bash
# Clone repository
git clone <repo_url>
cd step_graph

# Install with Poetry (recommended)
poetry install

# Or with pip
pip3 install -e .

# Setup Neo4j (optional) - using latest 5.x
docker run -p 7474:7474 -p 7687:7687 -e NEO4J_AUTH=neo4j/password neo4j:5-community

# Configure
cp .env.example .env
# Edit .env with your Neo4j credentials
```

### Usage

**Stage 1: Identify Steps**
```bash
python -m step_graph identify \
  --input IG_start/data/raw_patent_data/ \
  --run-id run_20251028_001 \
  --neo4j
```

**Stage 2-4: Manual Workflow**
See `.specify/specs/001-step-graph/quickstart.md` for detailed human-in-the-loop workflow with Copilot.

## Architecture

- **JSONL-First**: All outputs write to `.artifacts/{run_id}/{stage}/`
- **Neo4j-Optional**: Graph visualization when available
- **Copilot-Driven**: Stages 2-4 use manual curation for quality

## Documentation

- **Specification**: `.specify/specs/001-step-graph/spec.md`
- **Implementation Plan**: `.specify/specs/001-step-graph/plan.md`
- **Quickstart Guide**: `.specify/specs/001-step-graph/quickstart.md`
- **Tasks Breakdown**: `.specify/specs/001-step-graph/tasks.md`
```

---

## Verification Checklist

After implementing each phase, verify:

### Phase 0 (Setup)
- [ ] `python -m step_graph --help` shows CLI commands
- [ ] `.env` file configured with Neo4j credentials (or skipped)
- [ ] Project structure matches specification

### Phase 1 (Utilities)
- [ ] `pytest tests/test_utils.py` passes
- [ ] UUID generation works: `python -c "from step_graph.utils.ids import generate_step_id; print(generate_step_id())"`
- [ ] JSONL I/O works with UTF-8 encoding (test with Chinese text)

### Phase 2 (Stage 1)
- [ ] `python -m step_graph identify --input <dir> --run-id test_001` runs successfully
- [ ] `.artifacts/test_001/stage1_identification/steps.jsonl` created
- [ ] `.artifacts/test_001/stage1_identification/parameters.jsonl` created
- [ ] Handles edge cases: empty technical_process, "null" values, 0 parameters

### Phase 3 (Neo4j - Optional)
- [ ] `python -m step_graph export --run-id test_001 --stage 1` imports to Neo4j
- [ ] Neo4j Browser shows Patent → Step → Parameter graph
- [ ] Cypher query works: `MATCH (s:Step {run_id: 'test_001'}) RETURN count(s)`

### Phase 4 (Stages 2-4 Helpers)
- [ ] `python -m step_graph encode --run-id test_001` shows workflow instructions
- [ ] Copilot prompt templates exist in `.specify/specs/001-step-graph/copilot-prompts/`
- [ ] `quickstart.md` explains manual workflow step-by-step

### Phase 5 (Testing)
- [ ] `pytest tests/` passes with >60% coverage
- [ ] E2E test processes sample patent end-to-end
- [ ] `README.md` includes installation and quick start guide

---

## Critical Implementation Notes

### 1. Handle 0/Empty/"null" as Unknown (Constitution Requirement)

**Wrong**:
```python
if value == 0:  # Treats 0 as mathematical zero
    return None
```

**Correct**:
```python
if value in ["0", "", "null", None]:  # Treats as "not specified"
    return "unknown"
```

### 2. JSONL UTF-8 Encoding (Chinese Text)

**Wrong**:
```python
with open(file, 'w') as f:  # Uses ASCII by default
    json.dump(data, f)
```

**Correct**:
```python
with open(file, 'w', encoding='utf-8') as f:
    json.dump(data, f, ensure_ascii=False)
```

### 3. Neo4j Parameterized Queries (Security)

**Wrong**:
```python
query = f"MATCH (s:Step {{run_id: '{run_id}'}}) RETURN s"  # SQL injection risk
```

**Correct**:
```python
query = "MATCH (s:Step {run_id: $run_id}) RETURN s"
session.run(query, {"run_id": run_id})
```

### 4. Click CLI Optional Dependencies

**Wrong**:
```python
from step_graph.neo4j.client import Neo4jClient  # Crashes if neo4j not installed
```

**Correct**:
```python
try:
    from step_graph.neo4j.client import Neo4jClient
    neo4j_available = True
except ImportError:
    neo4j_available = False
    click.echo("Neo4j support not installed (pip install step_graph[neo4j])")
```

### 5. Run ID Versioning

**Pattern**: Always tag outputs with `run_id` to support multiple versions:
```python
# JSONL path includes run_id
output_path = f".artifacts/{run_id}/stage1_identification/steps.jsonl"

# Neo4j nodes include run_id property
CREATE (s:Step {id: $id, run_id: $run_id, ...})

# Queries filter by run_id
MATCH (s:Step {run_id: $run_id}) RETURN s
```

---

## Expected Deliverables

After full implementation, you should have:

1. **Functional CLI**: `python -m step_graph {identify|encode|atomize|cluster|export}`
2. **Stage 1 Automation**: Processes 10-50 patents → JSONL outputs
3. **Stage 2-4 Workflow**: Documented Copilot prompts and manual workflow
4. **Optional Neo4j**: Visualization of all 4 stages (when Neo4j available)
5. **Tests**: >60% coverage for parsers/utils
6. **Documentation**: README, quickstart.md, copilot-prompts/

---

## Performance Targets

- **Stage 1**: Process 10 patents in <5 minutes (fully automated)
- **Stage 2-4**: 30-40 minutes per patent (manual with Copilot assistance)
- **JSONL I/O**: <1 second per stage write
- **Neo4j Import**: 100-1000 nodes per batch in <10 seconds
- **Accuracy**: >85% transformation characterizations correct (domain expert review)

---

## Next Steps After Implementation

1. **Process Initial Dataset**: Run on 10-20 battery patents from `IG_start/data/raw_patent_data/`
2. **Validate Quality**: Domain expert review of Stage 2-4 outputs
3. **Iterate on Prompts**: Refine Copilot prompts based on accuracy results
4. **Expand Taxonomy**: Add new transformation forms based on observed patterns
5. **Optimize Workflow**: Reduce manual curation time from 30-40 min to 15-20 min per patent

---

## Questions to Resolve During Implementation

If you encounter ambiguities, refer to:

1. **Constitution**: `memory/constitution.md` (Chinese) or `memory/constitution.en.md` (English)
2. **Specification**: `.specify/specs/001-step-graph/spec.md` (detailed requirements)
3. **Implementation Plan**: `.specify/specs/001-step-graph/plan.md` (architecture decisions)
4. **Task Breakdown**: `.specify/specs/001-step-graph/tasks.md` (step-by-step checklist)

If still unclear, ask:
- **For Stage 1**: How should Lane B (raw text) parser differ from Lane A (semi-structured JSON)?
- **For Stage 2-4**: What edge cases in transformation encoding need special handling?
- **For Neo4j**: Should we create indexes on `run_id` + `stage` for query performance?

---

## Success Criteria

Implementation is complete when:

- ✅ All 28 tasks in `tasks.md` are completed
- ✅ `pytest tests/` passes with >60% coverage
- ✅ 5-10 sample patents processed end-to-end successfully
- ✅ Average time per patent <50 minutes (including manual Stages 2-4)
- ✅ >85% transformation characterizations accurate (domain expert validation)
- ✅ No blocking bugs in identified issues
- ✅ README and quickstart.md guide new users from zero to first patent processed

---

**Now begin implementation starting with Phase 0 (Setup). Follow the roadmap sequentially, running verification steps after each phase. Good luck!**
