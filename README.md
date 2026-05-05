LLAMA FACTORY INSTALLATION AND TRAINING GUIDE

This guide will walk you through setting up LLaMA Factory on a new AWS instance
and running CPT (Continued Pre-Training) and SFT (Supervised Fine-Tuning) 
training for both LLaMA and Gemma models.

PART 1: INSTALLING MINICONDA AND SETTING UP ENVIRONMENT

1. INSTALLING MINICONDA ON LINUX (AWS EC2 INSTANCE)
   -------------------------------------------------

   a) Download Miniconda installer:
      wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

   b) Make the installer executable:
      chmod +x Miniconda3-latest-Linux-x86_64.sh

   c) Run the installer (follow prompts, accept license, choose installation location):
      ./Miniconda3-latest-Linux-x86_64.sh

   d) After installation, initialize conda for your shell:
      source ~/.bashrc
      (or restart your terminal)

   e) Verify installation:
      conda --version


2. CREATING AND ACTIVATING CONDA ENVIRONMENT
   ------------------------------------------

   a) Create a new conda environment named "nust" with Python 3.10:
      conda create -n nust python=3.10 -y

   b) Activate the environment:
      conda activate nust

   c) Verify Python version:
      python --version
      (Should show Python 3.10.x)

   Note: Always activate this environment before working with LLaMA Factory:
      conda activate nust

   IMPORTANT: Using a virtual environment (conda environment) is crucial to avoid
   conflicts and issues with global system packages. It ensures isolated dependencies
   and prevents breaking system-wide installations.


PART 2: HUGGING FACE SETUP

1. INSTALLING HUGGING FACE HUB
   ----------------------------

   Some datasets and models require access to Hugging Face Hub. Install the 
   required version:

   pip install "huggingface_hub<1.0.0"


2. LOGGING INTO HUGGING FACE
   ---------------------------

   a) Login to Hugging Face CLI:
      huggingface-cli login

   b) Enter your Hugging Face token when prompted
      (You can get your token from: https://huggingface.co/settings/tokens)

   c) IMPORTANT: Ensure your token has access to the models you plan to use.
      Some models may require you to accept their license terms on Hugging Face
      website before you can download them.


PART 3: INSTALLING LLAMA FACTORY

1. CLONING LLAMA FACTORY REPOSITORY
   ---------------------------------

   Make sure you are in your desired directory and have activated the conda 
   environment:

   conda activate nust

   a) Clone the LLaMA Factory repository:
      git clone --depth 1 https://github.com/hiyouga/LlamaFactory.git

   b) Navigate to the cloned directory:
      cd LlamaFactory

   c) Install LLaMA Factory in editable mode:
      pip install -e .

   d) Install metrics dependencies:
      pip install -r requirements/metrics.txt

   e) (Optional) Install DeepSpeed dependencies if needed:
      pip install -r requirements/deepspeed.txt


2. RESOLVING PACKAGE VERSION CONFLICTS
   ------------------------------------

   IMPORTANT: There can be conflicts between different packages like LLaMA Factory,
   DeepSpeed, PyTorch, transformers, etc. The following versions have been tested
   and found to work well together to resolve common conflicts:

   Recommended versions:
   - torch: 2.9.1+cu128
   - trl: 0.24.0
   - deepspeed: 0.16.9
   - transformers: 4.57.1
   - accelerate: 1.11.0

   To install these specific versions:

   pip install torch==2.9.1+cu128 --index-url https://download.pytorch.org/whl/cu128
   pip install trl==0.24.0
   pip install deepspeed==0.16.9
   pip install transformers==4.57.1
   pip install accelerate==1.11.0

   Note: Adjust the CUDA version in torch installation based on your system's
   CUDA version. The example above uses CUDA 12.8.

   To verify installations, you can run:
   python -c "import torch, trl, deepspeed, transformers, accelerate; print('torch:', torch.__version__); print('trl:', trl.__version__); print('deepspeed:', deepspeed.__version__); print('transformers:', transformers.__version__); print('accelerate:', accelerate.__version__)"


PART 4: S3 ACCESS SETUP FOR DATASETS

1. INSTALLING S3 ACCESS LIBRARIES
   --------------------------------

   To access datasets stored in S3 buckets, you need to install the following
   packages:

   pip install s3fs
   pip install boto3


2. CONFIGURING AWS CREDENTIALS FOR S3 ACCESS
   -------------------------------------------

   IMPORTANT: Before accessing datasets from S3, ensure EC2 instance has
   proper access to the S3 bucket you plan to use.

3. CONFIGURING DATASETS IN DATASET_INFO.JSON
   ------------------------------------------

   All datasets that need to be accessed from S3 must be configured in the
   dataset_info.json file located at:

   LlamaFactory/data/dataset_info.json

   Format for S3 datasets:

   For CPT (Continued Pre-Training) datasets:

```json
{
  "dataset_name": {
    "cloud_file_name": "s3://bucket-name/path/to/dataset.jsonl",
    "columns": {
      "prompt": "text"
    }
  }
}
```

   For SFT (Supervised Fine-Tuning) datasets:

```json
{
  "dataset_name": {
    "cloud_file_name": "s3://bucket-name/path/to/dataset.jsonl",
    "formatting": "sharegpt",
    "columns": {
      "messages": "conversations"
    }
  }
}
```

   REAL EXAMPLES FROM MY DATASET_INFO.JSON:

   CPT Dataset Examples:

```json
{
  "cpt_jazz_pretrain": {
    "cloud_file_name": "s3://local-llm-data-1/NUST/CPT_Final_Data/merged_final_dataset.jsonl",
    "columns": {
      "prompt": "text"
    }
  },
  "cpt_jazz_v3": {
    "cloud_file_name": "s3://local-llm-data-1/NUST/CPT_Final_Data/v3_Jazz_Data_merged_dataset.jsonl",
    "columns": {
      "prompt": "text"
    }
  },
  "cpt_local_v3": {
    "cloud_file_name": "s3://local-llm-data-1/NUST/CPT_Final_Data/v3_Local_merged_dataset.jsonl",
    "columns": {
      "prompt": "text"
    }
  },
  "cpt_opensource_v3": {
    "cloud_file_name": "s3://local-llm-data-1/NUST/CPT_Final_Data/v3_Open_Source_merged_dataset.jsonl",
    "columns": {
      "prompt": "text"
    }
  }
}
```

   SFT Dataset Examples:

```json
{
  "sft_sharegpt": {
    "cloud_file_name": "s3://local-llm-data-1/NUST/SFT_Final_Dataset/combined_conversations_100k.jsonl",
    "formatting": "sharegpt",
    "columns": {
      "messages": "conversations"
    }
  },
  "ramzan_short_question_answer": {
    "cloud_file_name": "s3://local-llm-data-1/NUST/sharegpt_processed_data_by_ramzan/ShortQuestionAnswer_sharegpt_final.jsonl",
    "formatting": "sharegpt",
    "columns": {
      "messages": "conversations"
    }
  },
  "ramzan_roman_urdu": {
    "cloud_file_name": "s3://local-llm-data-1/NUST/sharegpt_processed_data_by_ramzan/RomanUrdu10kWithInstruction_sharegpt_final.jsonl",
    "formatting": "sharegpt",
    "columns": {
      "messages": "conversations"
    }
  },
  "ramzan_32k_sharegpt": {
    "cloud_file_name": "s3://local-llm-data-1/NUST/sharegpt_processed_data_by_ramzan/32k_sharegpt_final.jsonl",
    "formatting": "sharegpt",
    "columns": {
      "messages": "conversations"
    }
  },
  "ramzan_openhermes": {
    "cloud_file_name": "s3://local-llm-data-1/NUST/sharegpt_processed_data_by_ramzan/openhermes_6k_raw_sharegpt_final.jsonl",
    "formatting": "sharegpt",
    "columns": {
      "messages": "conversations"
    }
  },
  "ramzan_metamath": {
    "cloud_file_name": "s3://local-llm-data-1/NUST/sharegpt_processed_data_by_ramzan/metamath_4k_raw_sharegpt_final.jsonl",
    "formatting": "sharegpt",
    "columns": {
      "messages": "conversations"
    }
  },
  "ramzan_aya_urdu": {
    "cloud_file_name": "s3://local-llm-data-1/NUST/sharegpt_processed_data_by_ramzan/aya_urdu_2k_raw_sharegpt_final.jsonl",
    "formatting": "sharegpt",
    "columns": {
      "messages": "conversations"
    }
  }
}
```

   Note: You can add multiple datasets in the same JSON file. Make sure the
   JSON syntax is valid (proper commas, brackets, etc.). The dataset names
   (like "cpt_jazz_v3", "ramzan_openhermes") will be referenced in your
   YAML training configuration files.


PART 5: TRAINING CONFIGURATION FILES

1. YAML CONFIGURATION FILES LOCATION
   ----------------------------------

   Training configuration files (YAML files) for CPT and SFT training should be
   created in the following directory:

   LlamaFactory/examples/train_full/ or /train_lora

   This directory contains example YAML files for full model training. You can:
   - Use existing example files as templates
   - Create new YAML files for your specific training runs
   - Reference datasets configured in dataset_info.json by their dataset name

   Example structure:
   LlamaFactory/
   └── examples/
       └── train_full/
           ├── llama_cpt.yaml
           ├── llama_sft.yaml
           ├── gemma_cpt.yaml
           └── gemma_sft.yaml


PART 6: RUNNING TRAINING WITH LLAMA FACTORY

1. BASIC TRAINING COMMAND
   -----------------------

   The basic command to run training using LLaMA Factory is:

   llamafactory-cli train <path_to_yaml_file>

   Example:
   llamafactory-cli train examples/train_full/llama3_8b_full_sft.yaml


2. MULTI-GPU TRAINING (SINGLE NODE)
   ----------------------------------

   For multi-GPU training on a single machine, use FORCE_TORCHRUN:

   FORCE_TORCHRUN=1 llamafactory-cli train examples/train_full/llama3_8b_full_sft.yaml

   This will automatically use all available GPUs on the machine.


3. SPECIFYING SPECIFIC GPUS
   -------------------------

   To use specific GPUs (e.g., GPU 0 and GPU 1):

   CUDA_VISIBLE_DEVICES=0,1 llamafactory-cli train examples/train_full/llama3_8b_full_sft.yaml

   Or combine with FORCE_TORCHRUN for multi-GPU:

   CUDA_VISIBLE_DEVICES=0,1 FORCE_TORCHRUN=1 llamafactory-cli train examples/train_full/llama3_8b_full_sft.yaml



4. EXAMPLES FOR CPT AND SFT TRAINING
   -----------------------------------

   CPT (Continued Pre-Training) Example:
   FORCE_TORCHRUN=1 llamafactory-cli train examples/train_full/llama3_8b_full_pretrain.yaml

   SFT (Supervised Fine-Tuning) Example:
   FORCE_TORCHRUN=1 llamafactory-cli train examples/train_full/llama3_8b_full_sft.yaml


5. NOTES
   ------

   - Always make sure you are in the LlamaFactory directory when running commands
   - Ensure your conda environment is activated: conda activate nust
   - The YAML file path is relative to the LlamaFactory directory
   - Training logs and checkpoints will be saved to the output directory specified
     in your YAML configuration file
   - Use `nvidia-smi` to check GPU availability and usage
   - Monitor training progress through the logs or tensorboard (if configured)
