# TRIPWEAVER : Planning via LLM-Guided SMT-Optimization

A hybrid travel itinerary planner that uses natural language constraints, LLM-based code generation, and Z3 optimization to solve multi-city travel planning problems.

## 🔎 Project Overview

- **Input**: user query (text constraints + persona + destination/origin/dates)
- **Pipeline**:
  1. Normalize with LLM prompt (role: turn query into JSON)
  2. Convert constraints → planning steps via LLM (e.g., destination, departure, transportation, budget)
  3. Convert steps → Python code templates via LLM prompts
  4. Append hard-coded solve routine (`prompts/solve_{3,5,7}.txt`)
  5. Execute generated code with Z3 solver (`z3` Optimize/Solver)
  6. If satisfiable, generate an initial travel plan
  7. Run POI scheduling with Z3 solver and optimizer
  8. Produce the final travel plan with POI scheduling

<img src="assets/methodology.jpg" alt="Methodology diagram" height="700" />

## 📁 Repo Structure

- `Test_TravelPlanner.py`: main workflow and pipeline implementation.
- `z3_code_execution.py`: execute generated code in parallel across query batch outputs.
- `z3_temporal_scheduler.py`: temporal POI scheduling and itinerary optimization.
- `z3_temporal_scheduler_with_relaxation.py`: temporal POI scheduling with relaxation using soft constraints for more flexible schedule generation.
- `tools/`: API wrapper modules for external data fetching.
  - `cities/apis.py`
  - `flights/apis.py`
  - `accommodations/apis.py`
  - `attractions/apisv3.py`
  - `googleDistanceMatrix/apis.py`
  - `restaurants/apis.py`
- `prompts/`: prompt templates for LLM stages.
- `utils/`: helper logic (budget, selection, etc.).
- `openai_func.py` / `open_source_models.py`: LLM integration utilities.
- `requirements.txt`: Python dependencies.
- `output/`: generated run outputs (plans, codes, logs).

## ⚙️ Installation

1. Clone the repo:
   ```bash
   git clone <repo-url>
   cd TripWeaver
   ```
   After cloning the repo put the tripcraft database in the root folder. For database refer to the drive link in the Email.
   TripCraft_database folder should be in root folder.
   tripcraft_3day.csv, tripcraft_5day.csv, tripcraft_7day.csv should be in root folder.

   ---
2. Create and activate a Python environment:

- Using `conda` (env name `tripweaver`):
   ```bash
   conda create -n tripweaver python=3.11 -y
   conda activate tripweaver
   ```

3. Install deps:
   ```bash
   pip install -r requirements.txt
   ```

4. API keys (if you want real external API behavior):
   - `HUGGING_FACE_TOKEN`

## ▶️ Usage

### 1. Running the code generation and planner workflow

```bash
python Test_TravelPlanner.py --set_type 3d --model_name phi
```

**Arguments:**
- `--set_type`: Dataset type to use
  - `3d`: Use tripcraft_3day.csv dataset
  - `5d`: Use tripcraft_5day.csv dataset
  - `7d`: Use tripcraft_7day.csv dataset
  - Default: `3d`
- `--model_name`: LLM model to use for code generation
  - `gpt`: OpenAI GPT models
  - `qwen`: Qwen model
  - `phi`: Phi model
  - `llama`: Llama model
  - `mistral`: Mistral model
  - Default: `gpt`

### 2. Running parallel Z3 code execution

```bash
python z3_code_execution.py --days 3d --model_name phi
```

This script reads generated `query.json` and `codes.txt` from `output/<set_type>/<model_name>_nl/<index>/`, executes the generated Python code in parallel using `ProcessPoolExecutor`, and writes per-job run results.

### 3. Running temporal scheduling

```bash
python z3_temporal_scheduler.py
```

or with relaxed scheduling:

```bash
python z3_temporal_scheduler_with_relaxation.py
```

These scripts process plan outputs and build a scheduled itinerary with POIs, meals, accommodations, and transportation timing.

### 4. Output location
- `output/<set_type>/<model_name>_nl/<index>/plans/`
- `output/<set_type>/<model_name>_nl/<index>/codes/`


## ⚡ Evaluation

### 📊 Feasibility Metrics (Discrete)

```sh
# set_type: 3d/5d/7d
cd evaluation
python eval.py --set_type <SET_TYPE> --evaluation_file_path <EVALUATION_FILE_PATH>
```

### ♾️ Qualitative Metrics (Continuous)

```sh
cd evaluation
python qualitative_metrics.py --gen_file <generated_output_jsonl_file> --anno_file <annotation_jsonl_file>
```
