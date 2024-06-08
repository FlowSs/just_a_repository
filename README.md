## This is the replication package for the paper "DiffEval: Assessing Difficulty of Code Generation Tasks for Large Language Models" submitted at ASE 2024.

## Note on the replication package

* Some files (especially the raw generated codes) can be larger than the maximum file size allowed on `anonymous.4open.science`. As such, they will not be visible. We plan on moving them to a more permanent solution (Zenodo ...) 

* The code has a lot of redundancy and is split into a lot of smaller scripts to show the different part of the process as part of a replication package. We are working on a more unified structure of the package.


## Organization of the repository

* `data/`: contains all data from the experiments.
  * `humaneval/` and `classeval/` are the data on the original benchmarks
  * `sub_benchmarks/` contains the generated tasks.
   
  Each directory contains the prompts generated for all tasks (`prompts_generated_X.csv`) as well as the code generated by the Code LLMs (`raw/`). It also contains the functional correctness assessment (`post_test/`) and syntactic correctness (`sim/`). The difficulty scores are stored in `scores/`.

  * `prompt_templates/` contains the templates we used to generate the prompts as well as to generate the new tasks.

* `scripts/`: contains all the scripts to run DiffEval process. The usage of each scripts is explained below.

## Setting up the environment

We used Python 3.10 in our experiments in an anaconda environments. The file `environment.yaml` can be used to recreate the environment with anaconda (`conda env create -f environment.yml`).

The scripts for having GPT4 generates the prompts as well as having the LLMs generate codes uses OpenAI and HuggingFace API. As such, both an OpenAI key and HuggingFace token are required. For the scripts to work, one needs to add a `key.json` file in the `script/` directory. This file needs to contain both OpenAI and HuggingFace key/token as follow:

```json
{'OPENAI': YOUR_OPENAI_KEY,
'HF_TOKEN': YOUR_HF_TOKEN}
```

## Generating the level/rephrasing prompts

The script used is `generate_prompts.py`. It generate either the level type prompts or the rephrase type prompts on a given dataset or on the new generated tasks (sub_benchmark).

Usage:

```python
generate_prompts.py [-h] [-d DATASET] [-s SUB_BENCHMARK] [--levels] [--rephrase]

options:
  -h, --help            show this help message and exit
  -d DATASET, --dataset DATASET
  -s SUB_BENCHMARK, --sub_benchmark SUB_BENCHMARK
  --levels
  --rephrase
```

As the prompts are generated first on the levels and then on the rephrasing, one should first run the script with the `--levels` flag and then with the `--rephrase` flag.

The generated output will be a `.json` file named `prompts_generated_{DATASET}.csv` or `prompts_generated_{DATASET}_{SUB_BENCHMARK}.csv`. In the provided script, the files are saved at the root of the `script/` folder. The actual data are in `data/`. 

## Running on the code LLMs

The scripts used are `run_llm.py` and `run_llm_greedy.py`. The scripts let a model generates codes for based on the prompts of a given dataset/sub_benchmark. 

Usage:

```python
run_llm.py [-h] [-m MODEL] [-d DATASET] [-s SUB_BENCHMARK]

options:
  -h, --help            show this help message and exit
  -m MODEL, --model MODEL
  -d DATASET, --dataset DATASET
  -s SUB_BENCHMARK, --sub_benchmark SUB_BENCHMARK
```

The generated output will be a `.json` file named `results_{DATASET}_{MODEL}.json` or `results_{DATASET}_{SUB_BENCHMARK}_{MODEL}.json` for the script `run_llm.py`. For `run_greedy_llm.py` will have the same output with a `_greedy.json` at the end. In the provided scripts, the files are saved at the root of the `script/` folder. The actual data are in `data/`. 

The outputs of this script should be stored in the `raw/` sub-folder of each data folder. For instance, `results_humanevalplus_gpt.json` should be placed in `data/humanevalplus/raw/`

## Checking the functional correctness

To assess the code generated, we do the following:

* For HumanEval+, we make use of the test pipeline provided by the official implementation [EvalPlus](https://github.com/evalplus/evalplus). The final output should be a `.json` file structured with key related to HumanEval task_id. Inside each key, a nested dictionary is contained with the three levels which themselves contain a list of tuple (code, code_result).

```json
{
'0': {'level 1': [(SOME_CODE, TRUE/FALSE), ..., (SOME_CODE, TRUE/FALSE)],
      'level 2': [(SOME_CODE, TRUE/FALSE), ..., (SOME_CODE, TRUE/FALSE)],
      'level 3': [(SOME_CODE, TRUE/FALSE), ..., (SOME_CODE, TRUE/FALSE)]},
'1': { ...}
}
```
This script is used both for greedy and normal approach. 

* For ClassEval, we will use a modified version of their pipeline. To execute it, run the `evaluation.py` in `script/test_pipelines/`. The only parameter to modify is the source model (for instance `-so gpt`).

```python
evaluation.py [-h] [-so SOURCE_MODEL] [-e EVAL_DATA] [-st SAMPLED_TASKS]

options:
  -h, --help            show this help message and exit
  -so SOURCE_MODEL, --source_model SOURCE_MODEL
                        name of the model to evaluate
  -e EVAL_DATA, --eval_data EVAL_DATA
                        ClassEval data
  -st SAMPLED_TASKS, --sampled_tasks SAMPLED_TASKS
                        sampled_tasks
```
This script is used both for greedy and normal approach. For the greedy, one should add `_greedy` to the model name, e.g., `-so gpt_greedy`.


* For the the sub_benchmark, we will use our own test pipeline. You can the script `test_pipeline_humaneval_hard.py` or `test_pipeline_classeval_hard.py` in `script/test_pipelines/`.

```python
test_pipeline_humaneval_hard.py [-h] [-m MODEL] [-s SUB_BENCHMARK]

options:
  -h, --help            show this help message and exit
  -m MODEL, --model MODEL
  -s SUB_BENCHMARK, --sub_benchmark SUB_BENCHMARK
```

*Note*: Some requirements are needed to run the tasks of the benchmarks. One should install the requirements using the `rq.txt` file in each of the `data` repository. We recommend to use a Docker installation to run those pipelines.

The results obtained are `.json` files named `results_{DATASET}_{MODEL}_eval.json` or `results_{DATASET}_{SUB_BENCHMARK}_{MODEL}_eval.json`. They are all formatted similarly to the example in the HumanEval+ pipeline. All the results obtain from those scripts will be saved in a `post_test/` sub-directory in each of the data sub-folders. For instance, `results_humanevalplus_gpt_eval.json` will be in `data/humanevalplus/post_test/`

## Getting the similarity

The next step is to compute the similarity using the CodeBLEU metric. To do so, depending on the benchmark / sub_benchmark to assess, one should use the correct script from the `script/sim` folder. They all follow the same approach, with the only different that the `_hard.py` scripts require an argument, that is the sub_benchmark to process using `-s SUB_BENCHMARK`.

```python
get_sim_humanevalplus_hard.py [-h] [-s SUB_BENCHMARK]

options:
  -h, --help            show this help message and exit
  -s SUB_BENCHMARK, --sub_benchmark SUB_BENCHMARK
```

The results obtained are `.json` files named `results_{DATASET}_{MODEL}_sim.json` or `results_{DATASET}_{SUB_BENCHMARK}_{MODEL}_sim.json`. All the results obtain from those scripts will be saved in a `sim/` sub-directory in each of the data sub-folders. For instance, `results_humanevalplus_gpt_sim.json` will be in `data/humanevalplus/sim/`

## Getting the difficulty score

Finally, one can obtain the difficulty score by running the relevant script in the sub-folder `diff_score/`. For the benchmarks HumanEval+ and ClassEval, this will also return the Pass@1 score obtained on the level prompts as well as the cumulative distribution shown in the paper (see Fig 2). This returns the hard task per LLM as well as the hard tasks Overall LLMs. In that last case, we also get the difficulty score per hard task as well as the score per level.

*Note*: The difficulty score is computed such as the higher the more difficult while the scores on the individual level (Eq 4 in the paper) are computed such as lower is more difficult. Thus, it's normal for the difficulty scores and the scores per level to have opposing trend in the output.