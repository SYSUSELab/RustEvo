# RustEvo²
**Repo Under Construction ...**

This repository contains code and datasets for the paper [&#34;RustEvo²: An Evolving Benchmark for API Evolution in LLM-based Rust Code Generation&#34;](https://arxiv.org/abs/)



## Dataset Overview
Our work can be divided into two phases: 
Phase I: API Evolution Data Collection - We collect API changes from multiple sources including official Rust repositories and third-party crates. We analyze changelogs, documentation, and implementation changes to identify and categorize API evolutions into Stabilizations, Signature Changes, Behavioral Changes, and Deprecations.

Phase II: RustEvo² Construction - We transform the collected API evolution data into natural programming tasks using an LLM-based generation pipeline. This process creates programming queries, code solutions, and test programs that implicitly require the use of specific API versions.

The following figure illustrates our two-phase framework:
<div align="center">
  <img src="Imgs/overview.pdf" width="50%"/>
</div>


### Dataset Format
RustEvo² consists of 588 API changes (380 from Rust standard libraries, 208 from 15 third-party crates) spanning versions 1.71.0 to 1.84.0. These changes are categorized into four types: Stabilizations (31.3%), Signature Changes (31.5%), Behavioral Changes (33.2%), and Deprecations (4.1%), reflecting their actual distribution in the Rust ecosystem.

Each task in RustEvo² consists of <API change information, programming query, function signature, reference solution, test program>. The API change information includes name, module path, version details, documentation, and source code. Programming queries describe real-world scenarios without explicitly mentioning the API. Function signatures guide implementation without revealing API specifics. Test programs verify correct API usage and functional behavior.

One task example:
```json
    {
        "task_idx": 39,
        "query": "In a performance-critical application, you need to efficiently update a large collection of objects by cloning their state from another collection. The objects implement a custom `Clone` trait, but you want to avoid unnecessary trait bounds that could complicate the implementation. Design a function to handle this cloning operation efficiently.",
        "function_signature": "fn update_collection<T: Clone>(target: &mut Vec<T>, source: &Vec<T>)",
        "code": "fn update_collection<T: Clone>(target: &mut Vec<T>, source: &Vec<T>) {\n    target.truncate(source.len());\n    for (t, s) in target.iter_mut().zip(source.iter()) {\n        t.clone_from(s);\n    }\n    if target.len() < source.len() {\n        target.extend(source[target.len()..].iter().cloned());\n    }\n}",
        "test_program": "..."
    },
```


## Usage

### Setup
1. Environment Setup:

```bash
conda create -n RustEvo python=3.8
conda activate RustEvo
pip install -r requirements.txt
```

2. Install Rust toolchain
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup toolchain install 1.71.0 1.72.0 1.73.0 1.74.0 1.75.0 1.76.0 1.77.0 1.78.0 1.79.0 1.80.0 1.81.0 1.82.0 1.83.0 1.84.0
```

### Construct your own evolving dataset
If you don't want to construct a new dataset, you can directly use the existing dataset in the `data` folder.

1. Phase I: API Evolution Collection
```bash
python scripts/rust_api_analyzer.py --repo ./rust-repo --output ./reports --start 1.72.0 --end 1.84.0
python scripts/crate_analyzer.py --crates_num 15 --start_date 2024-01-01 --end_date 2025-02-21
```

2. Phase II: Task Generation
```bash
python scripts/generate_query.py --input ./reports/rust_api_changes.json --output ./data/queries/queries_rust.json
python scripts/generate_code.py --input ./data/queries/queries_rust.json --output ./data/codes/codes_rust.json
python scripts/generate_test.py --input_file ./data/codes/codes_rust.json --output_file ./data/test_programs/test_programs_rust.json
```

### Evaluate
1. Replace the target LLM in the evaluate/generation.py
2. Run the evaluation script
```bash
cd evaluate
./run.sh eval_models.py --model_name
```


## Results
Some important results of our experiments:
### Performance by Model
| Model | Pass@1 (%) | API Usage Accuracy (%) | Coverage (%) |
|-------|------------|---------|--------------|
| Claude-3.7-Sonnet | 65.3 | 78.2 | 83.6 |
| o1-mini | 57.5 | 70.4 | 85.2 |
| GPT-4o | 55.4 | 68.4 | 77.2 |
| Gemini-1.5-Pro | 55.3 | 62.6 | 60.9 |
| DeepSeek-v3 | 54.8 | 69.7 | 71.0 |
| Gemini-2.0-Flash | 52.6 | 73.5 | 72.5 |
| Llama-3.1-70B | 51.0 | 65.3 | 69.0 |
| Qwen-2.5-72B | 50.9 | 66.7 | 64.7 |
| Claude-3.5-Sonnet | 48.1 | 68.7 | 80.3 |
| Grok-3 | 40.5 | 67.2 | 70.4 |

### Performance by API Change Type
| Change Type | Average Pass@1 (%) |
|-------------|-------------------|
| Stabilizations | 65.8 |
| Signature Changes | 58.2 |
| Behavioral Changes | 38.0 |
| Deprecations | 40.4 |

Complete evaluation results and error analysis are [here](Results).
