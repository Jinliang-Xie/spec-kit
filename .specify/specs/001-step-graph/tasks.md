# Implementation Tasks: Patent Step Graph Extraction

**Feature**: 001-step-graph  
**Generated**: 2025-10-29  
**Based on**: spec.md v0.3.0, plan.md v1.0.0, research.md v0.2.0

---

## Task Organization

Tasks are organized by **Epic** (matching spec.md), with clear dependencies and parallel execution opportunities. Each task includes:
- **ID**: Unique task identifier
- **Dependencies**: Prerequisites (use task IDs)
- **Parallel**: `[P]` if task can run in parallel with siblings
- **Files**: Target files to create/modify
- **Verification**: How to validate completion

**Execution Strategy**: Complete Infrastructure tasks first, then proceed Epic-by-Epic with checkpoints.

---

## Epic 0: Infrastructure Foundation

**Goal**: Set up project structure, core utilities, and Neo4j client (optional for visualization).

**Checkpoint**: Basic project structure exists; CLI framework works; Neo4j connection optional but testable.

---

### TASK-INF-001: Initialize Project Structure
**Priority**: Critical | **Effort**: 1 hour | **Dependencies**: None | **[P]**

**Description**: Create Python package structure and configuration files.

**Files to create**:
```
step_graph/
├── __init__.py
├── __main__.py
├── parsers/
│   └── __init__.py
├── neo4j/
│   └── __init__.py
├── cli/
│   └── __init__.py
└── utils/
    └── __init__.py

pyproject.toml
.env.example
README.md
.gitignore
```

**Actions**:
1. Create directory structure as shown above
2. Initialize `pyproject.toml` with minimal dependencies:
   ```toml
   [project]
   name = "step-graph"
   version = "0.1.0"
   requires-python = ">=3.11"
   dependencies = [
       "click>=8.0",
   ]
   
   [project.optional-dependencies]
   neo4j = ["neo4j>=5.14"]
   
   [project.scripts]
   step-graph = "step_graph.__main__:cli"
   ```
3. Create `.env.example` with Neo4j configuration template
4. Add basic `.gitignore` (Python, `.env`, `.artifacts/`)

**Verification**:
```bash
python -m step_graph --help  # Should show CLI help (even if empty)
```

---

### TASK-INF-002: Implement Core Utilities
**Priority**: High | **Effort**: 2 hours | **Dependencies**: INF-001 | **[P]**

**Description**: Create shared utility functions for UUID generation, config management, and JSONL operations.

**Files to create**:
- `step_graph/utils/ids.py`
- `step_graph/utils/config.py`
- `step_graph/utils/artifacts.py`

**Implementation**:

**`utils/ids.py`**:
```python
import uuid

def generate_step_id() -> str:
    """Generate UUID v4 for Step entities."""
    return str(uuid.uuid4())

def generate_run_id() -> str:
    """Generate UUID v4 for pipeline runs."""
    return str(uuid.uuid4())
```

**`utils/config.py`**:
```python
import os
from pathlib import Path
from typing import Optional

def get_neo4j_uri() -> Optional[str]:
    """Get Neo4j URI from environment (optional)."""
    return os.getenv("NEO4J_URI")

def get_neo4j_auth() -> Optional[tuple]:
    """Get Neo4j credentials from environment (optional)."""
    user = os.getenv("NEO4J_USER", "neo4j")
    password = os.getenv("NEO4J_PASSWORD")
    return (user, password) if password else None
```

**`utils/artifacts.py`**:
```python
import json
from pathlib import Path
from typing import List, Dict, Any

def write_jsonl(data: List[Dict[Any, Any]], filepath: Path) -> None:
    """Write list of dicts to JSONL file."""
    filepath.parent.mkdir(parents=True, exist_ok=True)
    with open(filepath, 'w', encoding='utf-8') as f:
        for item in data:
            f.write(json.dumps(item, ensure_ascii=False) + '\n')

def read_jsonl(filepath: Path) -> List[Dict[Any, Any]]:
    """Read JSONL file into list of dicts."""
    data = []
    with open(filepath, 'r', encoding='utf-8') as f:
        for line in f:
            if line.strip():
                data.append(json.loads(line))
    return data

def create_artifact_dir(run_id: str, stage: str, base_dir: Path = Path(".artifacts")) -> Path:
    """Create and return artifact directory for a stage."""
    artifact_dir = base_dir / run_id / stage
    artifact_dir.mkdir(parents=True, exist_ok=True)
    return artifact_dir
```

**Verification**:
```python
# Test in Python REPL
from step_graph.utils.ids import generate_step_id, generate_run_id
from step_graph.utils.artifacts import write_jsonl, read_jsonl
from pathlib import Path

# UUID generation
assert len(generate_step_id()) == 36
assert len(generate_run_id()) == 36

# JSONL I/O
test_data = [{"id": "1", "text": "test"}]
write_jsonl(test_data, Path("/tmp/test.jsonl"))
assert read_jsonl(Path("/tmp/test.jsonl")) == test_data
```

---

### TASK-INF-003: Setup CLI Framework
**Priority**: High | **Effort**: 2 hours | **Dependencies**: INF-001, INF-002

**Description**: Implement Click-based CLI entry point with subcommand structure.

**Files to create/modify**:
- `step_graph/__main__.py`
- `step_graph/cli/__init__.py`

**Implementation**:

**`__main__.py`**:
```python
import click
from step_graph.utils.config import get_neo4j_uri

@click.group()
@click.version_option(version="0.1.0")
def cli():
    """Patent Step Graph Extraction Pipeline"""
    pass

@cli.command()
def check():
    """Check environment and dependencies."""
    click.echo("✓ Python 3.11+ detected")
    
    neo4j_uri = get_neo4j_uri()
    if neo4j_uri:
        click.echo(f"✓ Neo4j URI configured: {neo4j_uri}")
    else:
        click.echo("⚠ Neo4j not configured (optional for visualization)")
    
    click.echo("\n✅ Basic setup OK")

if __name__ == "__main__":
    cli()
```

**Verification**:
```bash
python -m step_graph --version
python -m step_graph check
```

---

### TASK-INF-004: Implement Neo4j Client (Optional)
**Priority**: Medium | **Effort**: 3 hours | **Dependencies**: INF-001, INF-002 | **[P]**

**Description**: Create Neo4j driver wrapper with connection management and basic operations.

**Files to create**:
- `step_graph/neo4j/client.py`
- `step_graph/neo4j/models.py`

**Implementation**:

**`neo4j/client.py`**:
```python
from neo4j import GraphDatabase
from typing import Optional
import logging

logger = logging.getLogger(__name__)

class Neo4jClient:
    def __init__(self, uri: str, auth: tuple):
        self.driver = GraphDatabase.driver(uri, auth=auth)
    
    def close(self):
        self.driver.close()
    
    def test_connection(self) -> bool:
        """Test Neo4j connection."""
        try:
            with self.driver.session() as session:
                result = session.run("RETURN 1 AS num")
                return result.single()["num"] == 1
        except Exception as e:
            logger.error(f"Connection failed: {e}")
            return False
    
    def create_constraints(self):
        """Create uniqueness constraints and indexes."""
        with self.driver.session() as session:
            # Constraints
            session.run("""
                CREATE CONSTRAINT step_id_unique IF NOT EXISTS
                FOR (s:Step) REQUIRE s.id IS UNIQUE
            """)
            session.run("""
                CREATE CONSTRAINT patent_id_unique IF NOT EXISTS
                FOR (p:Patent) REQUIRE p.patent_id IS UNIQUE
            """)
            
            # Indexes
            session.run("""
                CREATE INDEX step_run_id_idx IF NOT EXISTS
                FOR (s:Step) ON (s.run_id)
            """)
            session.run("""
                CREATE INDEX step_stage_idx IF NOT EXISTS
                FOR (s:Step) ON (s.stage)
            """)
        
        logger.info("✓ Constraints and indexes created")
```

**`neo4j/models.py`**:
```python
from typing import Dict, Any, List

def create_patent_node(tx, patent_id: str, run_id: str, **properties):
    """Create or update Patent node."""
    query = """
    MERGE (p:Patent {patent_id: $patent_id})
    SET p.run_id = $run_id,
        p += $properties
    RETURN p
    """
    tx.run(query, patent_id=patent_id, run_id=run_id, properties=properties)

def create_step_node(tx, step_id: str, run_id: str, stage: str, **properties):
    """Create or update Step node."""
    query = """
    MERGE (s:Step {id: $step_id})
    SET s.run_id = $run_id,
        s.stage = $stage,
        s += $properties
    RETURN s
    """
    tx.run(query, step_id=step_id, run_id=run_id, stage=stage, properties=properties)

def create_has_step_relationship(tx, patent_id: str, step_id: str, order: int):
    """Link Patent to Step with order."""
    query = """
    MATCH (p:Patent {patent_id: $patent_id})
    MATCH (s:Step {id: $step_id})
    MERGE (p)-[r:HAS_STEP]->(s)
    SET r.order = $order
    """
    tx.run(query, patent_id=patent_id, step_id=step_id, order=order)
```

**Verification**:
```python
from step_graph.neo4j.client import Neo4jClient
from step_graph.utils.config import get_neo4j_uri, get_neo4j_auth

# Only test if Neo4j is configured
uri = get_neo4j_uri()
auth = get_neo4j_auth()

if uri and auth:
    client = Neo4jClient(uri, auth)
    assert client.test_connection() == True
    client.create_constraints()
    client.close()
    print("✓ Neo4j client working")
else:
    print("⚠ Neo4j not configured - skipping test")
```

---

### TASK-INF-005: Create Test Infrastructure
**Priority**: Medium | **Effort**: 2 hours | **Dependencies**: INF-001, INF-002 | **[P]**

**Description**: Setup pytest configuration and basic test fixtures.

**Files to create**:
```
tests/
├── conftest.py
├── unit/
│   └── __init__.py
└── integration/
    └── __init__.py
```

**Implementation**:

**`tests/conftest.py`**:
```python
import pytest
from pathlib import Path
import tempfile
import shutil

@pytest.fixture
def temp_dir():
    """Provide temporary directory for tests."""
    tmp = tempfile.mkdtemp()
    yield Path(tmp)
    shutil.rmtree(tmp)

@pytest.fixture
def sample_patent_data():
    """Provide sample patent JSON for testing."""
    return {
        "patent_id": "CN-100342579-C",
        "technical_process": "混合粉末 -> 加热至95°C -> 保持30分钟",
        "technical_parameters": {
            "temperature": {"value": "95°C", "unit": "celsius"},
            "time": {"value": "30", "unit": "minutes"}
        }
    }

@pytest.fixture
def run_id():
    """Provide consistent run_id for tests."""
    return "test-run-12345"
```

**Verification**:
```bash
pytest tests/ -v  # Should pass (or skip if no tests yet)
```

---

**Epic 0 Checkpoint**:
- ✅ Project structure exists
- ✅ Core utilities (IDs, config, JSONL) work
- ✅ CLI framework responds to `--help` and `check`
- ✅ Neo4j client optional but functional (if configured)
- ✅ Test infrastructure ready

---

## Epic 1: Stage 1 — Step Identification (Automated)

**Goal**: Implement Lane A parsing to extract Steps and Parameters from patent JSON, write to JSONL (and optionally to Neo4j).

**Reference**: spec.md Epic 1 (US-1.1, US-1.2, US-1.3)

**Checkpoint**: Can run `python -m step_graph identify` on sample patents; produces valid JSONL; Neo4j import works if enabled.

---

### TASK-S1-001: Implement Lane A Parser
**Priority**: Critical | **Effort**: 4 hours | **Dependencies**: INF-001, INF-002

**Description**: Parse `technical_process` field to extract ordered Steps.

**Files to create**:
- `step_graph/parsers/lane_a.py`

**Implementation**:

**Reference**: research.md Section 3 (edge cases), data-model.md (JSONL schema)

```python
from typing import List, Dict, Any
from step_graph.utils.ids import generate_step_id

def parse_technical_process(text: str, patent_id: str, run_id: str) -> List[Dict[str, Any]]:
    """
    Parse technical_process field into ordered Steps.
    
    Handles edge cases:
    - Multiple delimiters: "Step A -> Step B -> Step C"
    - Mixed language: "加热 heat to 95°C"
    - Empty/null: returns empty list
    """
    if not text or text.strip() in ["", "null", "0"]:
        return []
    
    # Split by '->' delimiter
    raw_steps = [s.strip() for s in text.split('->')]
    
    steps = []
    for i, text_raw in enumerate(raw_steps):
        if text_raw:  # Skip empty steps
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

def detect_language(text: str) -> str:
    """Simple heuristic: detect if text contains Chinese characters."""
    for char in text:
        if '\u4e00' <= char <= '\u9fff':
            return "zh"
    return "en"
```

**Test** (`tests/unit/test_lane_a_parser.py`):
```python
from step_graph.parsers.lane_a import parse_technical_process

def test_parse_simple_process():
    steps = parse_technical_process(
        "Mix powder -> Heat to 95°C -> Cool",
        patent_id="TEST-001",
        run_id="run-1"
    )
    assert len(steps) == 3
    assert steps[0]["text_raw"] == "Mix powder"
    assert steps[0]["order_index"] == 1
    assert steps[2]["order_index"] == 3

def test_parse_empty_process():
    assert parse_technical_process("", "TEST-001", "run-1") == []
    assert parse_technical_process("null", "TEST-001", "run-1") == []

def test_parse_mixed_language():
    steps = parse_technical_process(
        "混合粉末 -> 加热至95°C",
        patent_id="TEST-001",
        run_id="run-1"
    )
    assert len(steps) == 2
    assert "混合" in steps[0]["text_raw"]
```

**Verification**:
```bash
pytest tests/unit/test_lane_a_parser.py -v
```

---

### TASK-S1-002: Implement Parameter Normalizer
**Priority**: High | **Effort**: 3 hours | **Dependencies**: INF-001, INF-002

**Description**: Parse and normalize `technical_parameters` field with dual-track (raw + normalized).

**Files to create**:
- `step_graph/parsers/normalizers.py`

**Implementation**:

**Reference**: research.md Section 3 (common parameters), spec.md FR-1.3

```python
import re
from typing import Dict, Any, Optional

def normalize_parameter(param_name: str, raw_value: str, raw_unit: str) -> Dict[str, Any]:
    """
    Normalize parameter value with dual-track approach.
    
    Handles:
    - 0/empty/"null" -> unknown with has_value=False
    - Unit conversion (basic cases only for MVP)
    - Value extraction from formatted strings
    """
    # Handle missing/unknown
    if raw_value in ["", "0", "null", None]:
        return {
            "param_name": param_name,
            "raw_value": raw_value or "unknown",
            "raw_unit": raw_unit or "",
            "normalized_value": None,
            "unit_std": "",
            "has_value": False,
            "confidence": 0.0
        }
    
    # Try to extract numeric value
    normalized_value = None
    unit_std = raw_unit
    confidence = 1.0
    
    # Handle common formats
    if param_name == "temperature":
        match = re.search(r'([\d.]+)\s*°?([CF])', str(raw_value))
        if match:
            value, unit = float(match.group(1)), match.group(2)
            if unit == 'F':
                # Convert F to C
                normalized_value = (value - 32) * 5/9
                unit_std = "celsius"
            else:
                normalized_value = value
                unit_std = "celsius"
    
    elif param_name in ["time", "duration"]:
        # Extract numeric part (assume minutes for MVP)
        match = re.search(r'([\d.]+)', str(raw_value))
        if match:
            normalized_value = float(match.group(1))
            unit_std = "minutes"  # Simplified assumption
    
    else:
        # Generic numeric extraction
        match = re.search(r'([\d.]+)', str(raw_value))
        if match:
            normalized_value = float(match.group(1))
    
    return {
        "param_name": param_name,
        "raw_value": str(raw_value),
        "raw_unit": raw_unit,
        "normalized_value": normalized_value,
        "unit_std": unit_std,
        "has_value": normalized_value is not None,
        "confidence": confidence
    }

def parse_parameters(params_dict: Dict[str, Any]) -> List[Dict[str, Any]]:
    """Parse technical_parameters dict into normalized list."""
    if not params_dict:
        return []
    
    normalized = []
    for param_name, param_data in params_dict.items():
        if isinstance(param_data, dict):
            raw_value = param_data.get("value", "")
            raw_unit = param_data.get("unit", "")
        else:
            raw_value = str(param_data)
            raw_unit = ""
        
        normalized.append(normalize_parameter(param_name, raw_value, raw_unit))
    
    return normalized
```

**Test** (`tests/unit/test_normalizers.py`):
```python
from step_graph.parsers.normalizers import normalize_parameter, parse_parameters

def test_temperature_normalization():
    result = normalize_parameter("temperature", "95°C", "celsius")
    assert result["normalized_value"] == 95.0
    assert result["has_value"] is True

def test_missing_value():
    result = normalize_parameter("pressure", "0", "")
    assert result["has_value"] is False
    assert result["normalized_value"] is None

def test_fahrenheit_conversion():
    result = normalize_parameter("temperature", "212°F", "fahrenheit")
    assert abs(result["normalized_value"] - 100.0) < 0.1  # Boiling point
    assert result["unit_std"] == "celsius"
```

**Verification**:
```bash
pytest tests/unit/test_normalizers.py -v
```

---

### TASK-S1-003: Implement Stage 1 CLI Command
**Priority**: Critical | **Effort**: 3 hours | **Dependencies**: S1-001, S1-002, INF-003

**Description**: Create `identify` subcommand to run Stage 1 pipeline.

**Files to create**:
- `step_graph/cli/identify.py`

**Implementation**:

**Reference**: spec.md FR-8 (CLI workflow), plan.md (Copilot workflow)

```python
import click
import json
from pathlib import Path
from step_graph.parsers.lane_a import parse_technical_process
from step_graph.parsers.normalizers import parse_parameters
from step_graph.utils.artifacts import write_jsonl, create_artifact_dir
from step_graph.utils.ids import generate_run_id

@click.command()
@click.option('--input', type=click.Path(exists=True), required=True,
              help='Directory containing patent JSON files')
@click.option('--output', type=click.Path(), default='.artifacts',
              help='Output directory for artifacts')
@click.option('--run-id', default=None,
              help='Run ID (auto-generated if not provided)')
def identify(input, output, run_id):
    """Stage 1: Identify and extract Steps from patent JSON files."""
    
    input_dir = Path(input)
    output_dir = Path(output)
    
    if not run_id:
        run_id = generate_run_id()
    
    click.echo(f"🔍 Stage 1: Step Identification")
    click.echo(f"   Run ID: {run_id}")
    click.echo(f"   Input: {input_dir}")
    
    # Collect patent JSON files
    patent_files = list(input_dir.glob('*.json'))
    click.echo(f"   Found {len(patent_files)} patent files")
    
    all_steps = []
    all_parameters = []
    
    # Process each patent
    for patent_file in patent_files:
        with open(patent_file, 'r', encoding='utf-8') as f:
            patent_data = json.load(f)
        
        patent_id = patent_data.get('patent_id', patent_file.stem)
        
        # Parse steps
        technical_process = patent_data.get('technical_process', '')
        steps = parse_technical_process(technical_process, patent_id, run_id)
        all_steps.extend(steps)
        
        # Parse parameters
        technical_params = patent_data.get('technical_parameters', {})
        params = parse_parameters(technical_params)
        
        # Link parameters to steps (simplified: assign to all steps)
        for param in params:
            param['run_id'] = run_id
            param['patent_id'] = patent_id
        
        all_parameters.extend(params)
        
        click.echo(f"   ✓ {patent_id}: {len(steps)} steps, {len(params)} parameters")
    
    # Write JSONL artifacts
    stage_dir = create_artifact_dir(run_id, 'stage1', output_dir)
    
    write_jsonl(all_steps, stage_dir / 'steps.jsonl')
    write_jsonl(all_parameters, stage_dir / 'parameters.jsonl')
    
    click.echo(f"\n✅ Stage 1 complete")
    click.echo(f"   Total: {len(all_steps)} steps, {len(all_parameters)} parameters")
    click.echo(f"   Output: {stage_dir}")
    click.echo(f"\n📝 Next: Review {stage_dir}/steps.jsonl and proceed to Stage 2")
```

**Register in `__main__.py`**:
```python
from step_graph.cli.identify import identify

cli.add_command(identify)
```

**Verification**:
```bash
# Prepare test data
mkdir -p test-patents
echo '{"patent_id": "TEST-001", "technical_process": "Mix -> Heat -> Cool", "technical_parameters": {"temperature": {"value": "95°C"}}}' > test-patents/TEST-001.json

# Run command
python -m step_graph identify --input test-patents --run-id test-run-1

# Check output
cat .artifacts/test-run-1/stage1/steps.jsonl
```

---

### TASK-S1-004: Implement Neo4j Import for Stage 1 (Optional)
**Priority**: Medium | **Effort**: 3 hours | **Dependencies**: S1-003, INF-004 | **[P]**

**Description**: Create `import-neo4j` command to load JSONL artifacts into Neo4j for visualization.

**Files to create**:
- `step_graph/cli/import_neo4j.py`
- `step_graph/neo4j/importers.py`

**Implementation**:

**`neo4j/importers.py`**:
```python
from typing import List, Dict, Any
from step_graph.neo4j.models import (
    create_patent_node,
    create_step_node,
    create_has_step_relationship
)

def import_stage1_steps(session, steps: List[Dict[str, Any]]):
    """Import Stage 1 steps into Neo4j."""
    for step in steps:
        session.execute_write(
            create_step_node,
            step_id=step['id'],
            run_id=step['run_id'],
            stage=step['stage'],
            text_raw=step['text_raw'],
            order_index=step['order_index'],
            source_field=step['source_field']
        )
        
        session.execute_write(
            create_has_step_relationship,
            patent_id=step['patent_id'],
            step_id=step['id'],
            order=step['order_index']
        )
```

**`cli/import_neo4j.py`**:
```python
import click
from pathlib import Path
from step_graph.neo4j.client import Neo4jClient
from step_graph.neo4j.importers import import_stage1_steps
from step_graph.utils.artifacts import read_jsonl
from step_graph.utils.config import get_neo4j_uri, get_neo4j_auth

@click.command()
@click.option('--jsonl', type=click.Path(exists=True), required=True,
              help='Path to stage JSONL directory (e.g., .artifacts/run-id/stage1/)')
@click.option('--neo4j-uri', default=None,
              help='Neo4j URI (default: from NEO4J_URI env)')
def import_neo4j(jsonl, neo4j_uri):
    """Import JSONL artifacts to Neo4j for visualization."""
    
    uri = neo4j_uri or get_neo4j_uri()
    auth = get_neo4j_auth()
    
    if not uri or not auth:
        click.echo("❌ Neo4j not configured. Set NEO4J_URI and NEO4J_PASSWORD")
        return
    
    jsonl_dir = Path(jsonl)
    steps_file = jsonl_dir / 'steps.jsonl'
    
    if not steps_file.exists():
        click.echo(f"❌ Steps file not found: {steps_file}")
        return
    
    click.echo(f"📊 Importing to Neo4j: {uri}")
    
    steps = read_jsonl(steps_file)
    click.echo(f"   Found {len(steps)} steps")
    
    client = Neo4jClient(uri, auth)
    
    with client.driver.session() as session:
        import_stage1_steps(session, steps)
    
    client.close()
    
    click.echo(f"✅ Import complete. Open Neo4j Browser to visualize")
```

**Register in `__main__.py`**:
```python
from step_graph.cli.import_neo4j import import_neo4j
cli.add_command(import_neo4j)
```

**Verification** (only if Neo4j configured):
```bash
python -m step_graph import-neo4j --jsonl .artifacts/test-run-1/stage1/

# Then in Neo4j Browser:
# MATCH (s:Step {run_id: 'test-run-1'}) RETURN s LIMIT 25
```

---

**Epic 1 Checkpoint**:
- ✅ Lane A parser handles common edge cases
- ✅ Parameter normalization works for top 5 parameters
- ✅ `identify` command produces valid JSONL
- ✅ Neo4j import works (if configured)
- ✅ Unit tests pass for parsers and normalizers

---

## Epic 2: Stage 2 — Transformation Encoding (Copilot-Assisted)

**Goal**: Use GitHub Copilot to characterize Steps along energy/matter/information transformations. Manual curation workflow with JSONL editing.

**Reference**: spec.md Epic 2 (US-2.1, US-2.2), research.md Section 2 & 4 (transformation taxonomy)

**Note**: Stage 2 is **human-in-the-loop** with Copilot assistance. No automated CLI command; manual JSONL editing guided by Copilot prompts.

**Checkpoint**: Documentation exists; prompt templates validated; sample JSONL with transformation_tags.

---

### TASK-S2-001: Document Transformation Schema
**Priority**: High | **Effort**: 2 hours | **Dependencies**: None | **[P]**

**Description**: Ensure `contracts/stage2-output-format.md` documents transformation_tags structure.

**File to verify/update**:
- `.specify/specs/001-step-graph/contracts/stage2-output-format.md`

**Required Content**:
- JSON schema for transformation_tags (from_form, to_form, magnitude, rationale)
- 3-4 complete examples (battery charging, heating, mixing, etc.)
- Magnitude scoring guide (0-1 scale for transformation degree)
- Common transformation patterns table (from research.md Section 4)

**Verification**:
- File exists and includes all required content
- Examples use actual from_form→to_form patterns from research

---

### TASK-S2-002: Create Copilot Prompt Template for Transformations
**Priority**: Critical | **Effort**: 3 hours | **Dependencies**: S2-001

**Description**: Create/validate `copilot-prompts/stage2-transformation-encoding.md` with effective prompts.

**File to verify/update**:
- `.specify/specs/001-step-graph/copilot-prompts/stage2-transformation-encoding.md`

**Required Content** (from research.md Section 2):

```markdown
# Stage 2: Transformation Encoding Prompt

## Prompt Template

Given this process step: "{step_text}"

Characterize transformations along three dimensions. Return JSON:

{
  "energy": {
    "from_form": "electrical|thermal|mechanical|chemical_potential|kinetic|other",
    "to_form": "thermal|chemical_potential|electrical|mechanical|other",
    "magnitude": 0.0-1.0,
    "rationale": "Explain the energy conversion"
  },
  "matter": {
    "from_form": "separated_powders|liquid_ambient_temp|solid_crystalline|...",
    "to_form": "homogeneous_mixture|liquid_elevated_temp|liquid_solution|...",
    "magnitude": 0.0-1.0,
    "rationale": "Explain the matter transformation"
  },
  "information": {
    "from_form": "voltage_measurement|temperature_sensor|optical_signal|none|...",
    "to_form": "battery_state_data|control_feedback|concentration_data|none|...",
    "magnitude": 0.0-1.0,
    "rationale": "Explain the information transformation"
  }
}

**Magnitude scale** (degree of transformation):
- 0.0-0.2: Minimal transformation (form barely changes)
- 0.3-0.5: Partial transformation (incomplete conversion)
- 0.6-0.8: Significant transformation (substantial conversion)
- 0.9-1.0: Near-complete transformation (nearly full conversion)

## Few-Shot Examples

[Include 3-4 examples from research.md Section 4]

Example 1: "Charge battery at 0.5C to 4.2V"
- Energy: electrical → chemical_potential (magnitude 0.9)
- Matter: lithium_ions_dispersed → lithium_ions_intercalated (magnitude 0.7)
- Information: voltage_measurement → battery_state_data (magnitude 0.5)

[Add more examples...]

## Common Transformation Patterns

[Copy tables from research.md Section 4]
```

**Verification**:
- Test prompt in Copilot Chat with 3-5 sample steps
- Achieve >70% accurate transformation characterizations
- Document any prompt refinements needed

---

### TASK-S2-003: Update Quickstart Guide for Stage 2 Workflow
**Priority**: High | **Effort**: 2 hours | **Dependencies**: S2-001, S2-002

**Description**: Ensure `quickstart.md` has clear Stage 2 human-in-the-loop workflow instructions.

**File to verify/update**:
- `.specify/specs/001-step-graph/quickstart.md`

**Required Sections** (verify already exists per recent updates):
- Stage 2: Encode Transformations (Copilot-Assisted)
- Workflow: Open JSONL → Copilot Chat → Edit transformation_tags → Save
- Example Copilot Chat session
- Batch processing tips with Copilot Edits
- Neo4j import (optional)

**Verification**:
- Follow quickstart instructions manually with 1-2 sample patents
- Confirm workflow is clear and efficient

---

### TASK-S2-004: Create Sample Transformation-Encoded JSONL
**Priority**: Medium | **Effort**: 2 hours | **Dependencies**: S2-002 | **[P]**

**Description**: Manually create 5-10 sample steps with complete transformation_tags using Copilot assistance.

**File to create**:
- `tests/fixtures/sample-stage2-steps.jsonl`

**Process**:
1. Take 5-10 steps from Stage 1 output
2. Use Copilot Chat with prompt template to characterize each
3. Manually review and adjust magnitude scores
4. Save as reference fixture

**Verification**:
- All samples have valid transformation_tags structure
- Magnitude scores align with research.md guidelines
- At least 3 different transformation patterns represented

---

**Epic 2 Checkpoint**:
- ✅ Transformation schema documented in `contracts/`
- ✅ Copilot prompt template tested and refined
- ✅ Quickstart guide includes Stage 2 workflow
- ✅ Sample transformation-encoded JSONL exists
- ⚠️ **No automated CLI** - This is human-in-the-loop by design

---

## Epic 3: Stage 3 — Step Atomicization (Copilot-Assisted)

**Goal**: Use Copilot to identify and split composite steps into atomic sub-steps. Manual JSONL editing with provenance tracking.

**Reference**: spec.md Epic 3 (US-3.1, US-3.2), research.md Section 2 (atomicization)

**Checkpoint**: Prompt template validated; sample atomicized JSONL with parent_id tracking.

---

### TASK-S3-001: Create Copilot Prompt Template for Atomicization
**Priority**: High | **Effort**: 3 hours | **Dependencies**: S2-002

**Description**: Create/validate `copilot-prompts/stage3-atomization.md` with composite step detection prompts.

**File to verify/update**:
- `.specify/specs/001-step-graph/copilot-prompts/stage3-atomization.md`

**Required Content**:

```markdown
# Stage 3: Atomicization Prompt

## Decision Criteria

A step is composite if:
1. **Multiple transformation types**: Involves significant transformations in 2+ dimensions
2. **Multiple explicit actions**: Contains multiple verbs (e.g., "mix AND heat")
3. **Sequential sub-operations**: Implies temporal sequence within the step

## Prompt Template

Given this step and its transformation characterization:

Step: "{step_text}"
Transformations: {paste transformation_tags JSON}

Questions:
1. Is this a composite step that should be split?
2. If yes, what are the atomic sub-steps?
3. For each atomic sub-step, re-characterize transformations

Return:
- is_composite: true/false
- split_reason: "explanation if composite"
- atomic_steps: [
    {
      "text": "atomic sub-step 1",
      "transformation_tags": {...}
    },
    ...
  ]

## Examples

[Include 2-3 examples of composite → atomic splits]

Example: "Mix powders and heat to 95°C"
- is_composite: true
- split_reason: "Contains sequential mechanical (mixing) and thermal (heating) operations"
- atomic_steps:
  1. "Mix powders" (matter: separated→homogeneous)
  2. "Heat to 95°C" (energy: ambient→elevated)

## Critical: Re-encode Transformations

After splitting, MUST re-run Stage 2 transformation encoding on each atomic sub-step.
```

**Verification**:
- Test with 5 composite steps from research
- Copilot correctly identifies composite vs. atomic
- Split suggestions are reasonable

---

### TASK-S3-002: Document Stage 3 Output Format
**Priority**: Medium | **Effort**: 1 hour | **Dependencies**: None | **[P]**

**Description**: Ensure `contracts/stage3-output-format.md` shows atomicized JSONL structure.

**File to verify/update**:
- `.specify/specs/001-step-graph/contracts/stage3-output-format.md`

**Required Content**:
- JSONL schema with `parent_id` field (for provenance)
- `split_reason` field
- Example showing composite step + its atomic children
- Note about re-encoding transformation_tags for atomic steps

**Verification**:
- Schema matches spec.md FR-4.2 requirements
- Example is clear and complete

---

### TASK-S3-003: Create Sample Atomicized JSONL
**Priority**: Medium | **Effort**: 2 hours | **Dependencies**: S3-001 | **[P]**

**Description**: Manually create 3-5 examples of composite→atomic splits using Copilot.

**File to create**:
- `tests/fixtures/sample-stage3-atomicized.jsonl`

**Process**:
1. Identify 3-5 composite steps from Stage 2 output
2. Use Copilot prompt to split
3. Re-encode transformations for each atomic sub-step
4. Document parent_id and split_reason

**Verification**:
- All atomic steps have valid transformation_tags
- Provenance chain is clear (parent_id links)
- split_reason explains rationale

---

**Epic 3 Checkpoint**:
- ✅ Atomicization prompt template tested
- ✅ Stage 3 output format documented
- ✅ Sample atomicized JSONL with provenance
- ⚠️ **No automated CLI** - Human-in-the-loop by design

---

## Epic 4: Stage 4 — Step Clustering (Manual with Copilot Analysis)

**Goal**: Use Copilot to analyze similarity between steps and suggest clusters. Manual cluster creation in JSONL.

**Reference**: spec.md Epic 4 (US-4.1, US-4.2), research.md Section 2 (similarity comparison)

**Checkpoint**: Prompt template for similarity; sample clusters.jsonl.

---

### TASK-S4-001: Create Copilot Prompt Template for Clustering
**Priority**: Medium | **Effort**: 2 hours | **Dependencies**: S3-001

**Description**: Create `copilot-prompts/stage4-clustering.md` with similarity analysis prompts.

**File to create/verify**:
- `.specify/specs/001-step-graph/copilot-prompts/stage4-clustering.md`

**Required Content**:

```markdown
# Stage 4: Clustering Prompt

## Similarity Criteria

Two steps are similar if:
1. **Transformation alignment**: Similar from_form→to_form patterns in 2+ dimensions
2. **Magnitude proximity**: Magnitude scores differ by <0.2
3. **Semantic equivalence**: Describe same process despite wording differences

## Prompt Template

Compare these two steps:

Step A: "{step_text_a}"
Transformations: {transformation_tags_a}

Step B: "{step_text_b}"
Transformations: {transformation_tags_b}

Questions:
1. Are these steps semantically similar?
2. What is the similarity score (0-1)?
3. What are the key transformation alignments or differences?

Return:
- similarity_score: 0.0-1.0
- rationale: "explanation"
- should_cluster: true/false (threshold: 0.85)

## Examples

[Include 2-3 similarity comparisons]
```

**Verification**:
- Test with 5-10 step pairs (similar and dissimilar)
- Similarity scores are reasonable
- Threshold of 0.85 works well

---

### TASK-S4-002: Document Stage 4 Output Format
**Priority**: Medium | **Effort**: 1 hour | **Dependencies**: None | **[P]**

**Description**: Ensure `contracts/stage4-output-format.md` shows cluster JSONL structure.

**File to verify/update**:
- `.specify/specs/001-step-graph/contracts/stage4-output-format.md`

**Required Content**:
- Cluster JSONL schema with `cluster_id`, `member_step_ids`, `cluster_method`
- Example showing 3-5 steps clustered together
- Similarity scores matrix (optional)

**Verification**:
- Schema matches spec.md FR-5.2
- Example is clear

---

### TASK-S4-003: Create Sample Clusters JSONL
**Priority**: Low | **Effort**: 2 hours | **Dependencies**: S4-001 | **[P]**

**Description**: Manually create 2-3 sample clusters using Copilot similarity analysis.

**File to create**:
- `tests/fixtures/sample-stage4-clusters.jsonl`

**Process**:
1. Take 10-15 atomicized steps from Stage 3
2. Use Copilot to compare pairs and find similar groups
3. Create cluster objects with member_step_ids
4. Document similarity rationale

**Verification**:
- At least 2 clusters with 3+ members each
- Similarity scores recorded
- Clusters are semantically coherent

---

**Epic 4 Checkpoint**:
- ✅ Clustering prompt template tested
- ✅ Stage 4 output format documented
- ✅ Sample clusters JSONL exists
- ⚠️ **No automated CLI** - Human-in-the-loop by design

---

## Epic 5: Visualization and Utilities

**Goal**: Provide optional Neo4j visualization commands, export utilities, and rollback functionality.

**Reference**: spec.md Epic 5 (US-5.1, US-5.2)

**Checkpoint**: Neo4j queries documented; export/rollback commands work.

---

### TASK-VIZ-001: Document Neo4j Query Templates
**Priority**: Medium | **Effort**: 2 hours | **Dependencies**: None | **[P]**

**Description**: Create/verify Neo4j query templates in `data-model.md` or separate doc.

**File to verify/update**:
- `.specify/specs/001-step-graph/data-model.md` (append section)

**Required Queries**:
1. View all stages for a run_id
2. Find steps with high energy transformation magnitude
3. Find specific transformation patterns (e.g., electrical→chemical)
4. Visualize atomicization provenance (parent→children)
5. View clusters and members

**Verification**:
- All queries work in Neo4j Browser (if configured)
- Queries are parameterized (use `$runId`, etc.)

---

### TASK-VIZ-002: Implement Export Command
**Priority**: Low | **Effort**: 2 hours | **Dependencies**: INF-003 | **[P]**

**Description**: Create `export` command to convert JSONL to CSV or plain text.

**File to create**:
- `step_graph/cli/export.py`

**Implementation**:

```python
import click
import csv
from pathlib import Path
from step_graph.utils.artifacts import read_jsonl

@click.command()
@click.option('--jsonl', type=click.Path(exists=True), required=True,
              help='JSONL file to export')
@click.option('--format', type=click.Choice(['csv', 'txt']), default='csv',
              help='Output format')
@click.option('--output', type=click.Path(), required=True,
              help='Output file path')
def export(jsonl, format, output):
    """Export JSONL to CSV or text format."""
    
    data = read_jsonl(Path(jsonl))
    output_path = Path(output)
    
    if format == 'csv':
        # Extract flat fields (skip nested transformation_tags for now)
        fieldnames = ['id', 'text_raw', 'order_index', 'stage', 'run_id']
        
        with open(output_path, 'w', newline='', encoding='utf-8') as f:
            writer = csv.DictWriter(f, fieldnames=fieldnames, extrasaction='ignore')
            writer.writeheader()
            writer.writerows(data)
    
    elif format == 'txt':
        with open(output_path, 'w', encoding='utf-8') as f:
            for item in data:
                f.write(f"{item.get('text_raw', '')}\n")
    
    click.echo(f"✅ Exported {len(data)} items to {output_path}")
```

**Verification**:
```bash
python -m step_graph export --jsonl .artifacts/test-run-1/stage1/steps.jsonl --format csv --output steps.csv
```

---

### TASK-VIZ-003: Implement Rollback Command
**Priority**: Medium | **Effort**: 2 hours | **Dependencies**: INF-004 | **[P]**

**Description**: Create `rollback` command to delete all data for a run_id from Neo4j.

**File to create**:
- `step_graph/cli/rollback.py`

**Implementation**:

```python
import click
from step_graph.neo4j.client import Neo4jClient
from step_graph.utils.config import get_neo4j_uri, get_neo4j_auth

@click.command()
@click.option('--run-id', required=True, help='Run ID to rollback')
@click.option('--neo4j-uri', default=None, help='Neo4j URI')
@click.option('--yes', is_flag=True, help='Skip confirmation')
def rollback(run_id, neo4j_uri, yes):
    """Delete all Neo4j data for a specific run_id."""
    
    uri = neo4j_uri or get_neo4j_uri()
    auth = get_neo4j_auth()
    
    if not uri or not auth:
        click.echo("❌ Neo4j not configured")
        return
    
    if not yes:
        click.confirm(f"⚠️  Delete all data for run_id '{run_id}'?", abort=True)
    
    client = Neo4jClient(uri, auth)
    
    with client.driver.session() as session:
        result = session.run("""
            MATCH (n {run_id: $run_id})
            DETACH DELETE n
            RETURN count(n) AS deleted_count
        """, run_id=run_id)
        
        deleted_count = result.single()["deleted_count"]
    
    client.close()
    
    click.echo(f"✅ Deleted {deleted_count} nodes for run_id '{run_id}'")
```

**Verification** (if Neo4j configured):
```bash
python -m step_graph rollback --run-id test-run-1 --yes
```

---

**Epic 5 Checkpoint**:
- ✅ Neo4j query templates documented
- ✅ Export command works for CSV/TXT
- ✅ Rollback command works (if Neo4j configured)

---

## Epic 6: Integration and End-to-End Validation

**Goal**: Validate full pipeline with real patents; ensure documentation is complete.

**Checkpoint**: Can process 5-10 patents end-to-end; quickstart guide validated.

---

### TASK-INT-001: End-to-End Pipeline Test
**Priority**: Critical | **Effort**: 4 hours | **Dependencies**: All previous tasks

**Description**: Process 5-10 real patents through all 4 stages manually following quickstart.md.

**Files to create**:
- `tests/integration/test_end_to_end.py` (optional automated test)
- `.artifacts/integration-test-{date}/` (manual run artifacts)

**Process**:
1. Select 5-10 patents from `IG_start/data/raw_patent_data/`
2. Run Stage 1: `python -m step_graph identify`
3. Manually perform Stage 2 with Copilot (transformation encoding)
4. Manually perform Stage 3 with Copilot (atomicization)
5. Manually perform Stage 4 with Copilot (clustering)
6. Import to Neo4j (if configured)
7. Document time taken and issues encountered

**Success Criteria**:
- All 5-10 patents processed successfully
- Total time <50 min per patent (on average)
- >85% transformation characterizations accurate (domain expert review)
- No blocking bugs

---

### TASK-INT-002: Validate Quickstart Guide
**Priority**: High | **Effort**: 2 hours | **Dependencies**: INT-001

**Description**: Have a fresh user (or simulate fresh start) follow quickstart.md step-by-step.

**Process**:
1. Clear .artifacts/ directory
2. Follow quickstart.md from "Prerequisites" to "Stage 4"
3. Note any unclear instructions or missing steps
4. Update quickstart.md with clarifications

**Verification**:
- Quickstart is followable without prior context
- All commands work as documented
- Expected outputs match actual outputs

---

### TASK-INT-003: Update README with Getting Started
**Priority**: Medium | **Effort**: 1 hour | **Dependencies**: INT-002

**Description**: Update repository `README.md` with quick start instructions.

**File to update**:
- `README.md` (repository root)

**Required Sections**:
- Project overview
- Installation instructions
- Quick start (5-minute example)
- Link to full quickstart guide
- Link to spec and plan

**Verification**:
- README is clear and concise
- Installation works on fresh Python 3.11+ environment

---

**Epic 6 Checkpoint**:
- ✅ End-to-end pipeline validated with real data
- ✅ Quickstart guide accurate and complete
- ✅ README provides clear getting started path

---

## Task Summary

### Total Task Count: 28 tasks

**By Epic**:
- Epic 0 (Infrastructure): 5 tasks
- Epic 1 (Stage 1): 4 tasks
- Epic 2 (Stage 2): 4 tasks
- Epic 3 (Stage 3): 3 tasks
- Epic 4 (Stage 4): 3 tasks
- Epic 5 (Visualization): 3 tasks
- Epic 6 (Integration): 3 tasks

**By Priority**:
- Critical: 5 tasks
- High: 7 tasks
- Medium: 13 tasks
- Low: 3 tasks

**Parallelization Opportunities**:
- 12 tasks marked `[P]` can run in parallel with dependencies satisfied
- Estimated 30-40% time savings with 2-3 developers

**Estimated Timeline**:
- Epic 0: 1 week (infrastructure setup)
- Epic 1: 1 week (Stage 1 implementation)
- Epic 2-4: 1 week (Copilot prompt refinement + samples)
- Epic 5-6: 1 week (utilities + validation)

**Total**: ~4 weeks for MVP (1 developer) or ~2.5 weeks (2-3 developers with parallelization)

---

## Dependency Graph (Critical Path)

```
INF-001 (Project Structure)
   ├─> INF-002 (Core Utilities) [P]
   │      ├─> INF-003 (CLI Framework)
   │      │      ├─> S1-003 (Stage 1 CLI)
   │      │      │      ├─> INT-001 (E2E Test)
   │      │      │      │      ├─> INT-002 (Validate Quickstart)
   │      │      │      │      │      └─> INT-003 (README)
   │      │      │      └─> S1-004 (Neo4j Import) [P]
   │      │      │             └─> VIZ-003 (Rollback) [P]
   │      │      └─> VIZ-002 (Export) [P]
   │      ├─> S1-001 (Lane A Parser)
   │      │      ├─> S1-002 (Normalizer)
   │      │      │      └─> S1-003 (Stage 1 CLI)
   │      │      └─> S1-003 (Stage 1 CLI)
   │      └─> INF-005 (Test Infrastructure) [P]
   └─> INF-004 (Neo4j Client) [P]
          └─> S1-004 (Neo4j Import) [P]

S2-001 (Document Schema) [P]
   └─> S2-002 (Copilot Prompts)
          ├─> S2-003 (Quickstart Update)
          │      └─> INT-002 (Validate Quickstart)
          ├─> S2-004 (Sample JSONL) [P]
          └─> S3-001 (Atomicization Prompts)
                 ├─> S3-002 (Stage 3 Format) [P]
                 ├─> S3-003 (Sample JSONL) [P]
                 └─> S4-001 (Clustering Prompts)
                        ├─> S4-002 (Stage 4 Format) [P]
                        └─> S4-003 (Sample JSONL) [P]

VIZ-001 (Neo4j Queries) [P]
```

**Critical Path** (longest sequential chain):
INF-001 → INF-002 → INF-003 → S1-003 → INT-001 → INT-002 → INT-003

**Critical Path Duration**: ~3 weeks (assuming 1 developer)

---

## Implementation Notes

### Testing Strategy

**Unit Tests** (>60% coverage target):
- `test_lane_a_parser.py`: Parse technical_process edge cases
- `test_normalizers.py`: Parameter normalization rules
- `test_ids.py`: UUID generation
- `test_artifacts.py`: JSONL read/write

**Integration Tests**:
- `test_stage1_identify.py`: Full Stage 1 pipeline with sample patent
- `test_neo4j_import.py`: JSONL → Neo4j import (if Neo4j configured)
- `test_end_to_end.py`: Manual workflow validation

**Manual Validation**:
- Stage 2-4: Human review of Copilot-generated transformations
- Domain expert validation of sample characterizations

### Quality Gates

**Before merging to main**:
- ✅ All unit tests pass
- ✅ Integration test with 5+ patents succeeds
- ✅ Quickstart guide validated by fresh user
- ✅ Code formatted (black/ruff)
- ✅ Type hints on public functions
- ✅ Docstrings on modules and key functions

### Risk Mitigation

**Risk**: Copilot accuracy variability
- **Mitigation**: Document effective prompts; mandatory human review

**Risk**: Manual curation throughput bottleneck
- **Mitigation**: Limit MVP to 10-15 patents; batch similar steps

**Risk**: Neo4j setup barrier
- **Mitigation**: JSONL-first architecture; Neo4j optional

---

## Appendix: Command Cheat Sheet

**Stage 1 (Automated)**:
```bash
python -m step_graph identify --input patents/ --output .artifacts/ --run-id $(uuidgen)
```

**Stage 2-4 (Manual with Copilot)**:
```bash
# Open JSONL in VS Code
code .artifacts/{run-id}/stage1/steps.jsonl

# Use Copilot Chat with prompt templates from copilot-prompts/
# Edit JSONL manually with Copilot suggestions
# Save updated JSONL
```

**Import to Neo4j (Optional)**:
```bash
python -m step_graph import-neo4j --jsonl .artifacts/{run-id}/stage1/
```

**Export to CSV**:
```bash
python -m step_graph export --jsonl .artifacts/{run-id}/stage1/steps.jsonl --format csv --output steps.csv
```

**Rollback**:
```bash
python -m step_graph rollback --run-id {run-id} --yes
```

---

**End of Tasks Document**

*Generated on 2025-10-29 based on spec.md v0.3.0, plan.md v1.0.0, and research.md v0.2.0*
