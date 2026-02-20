2026-02-18

- unsupervised learning
	- masking loss = omit data and force model to predict
	- contrastive loss = use out of distribution data to condition model on in-distribution data
- memory requirements
	- 1 Bn params
	- weights and gradients in FP16
	- Adam optimizer stores 12 bytes per weight in FP16
	- 1 Bn x (2B + 2B + 12B) = 14.9 GB
- time requirements
	- 1.7 Bn params = 6 weeks on 1 x DGX A100
	- 1 Trillion params = 69 years on 1 x DGX A100
- execution time
	- data
	- tools (GPU utilization, memory footprint)
	- inference
	- systems (compute, communication, storage, operations)
	- algorithms
- Megatron: https://github.com/NVIDIA/Megatron-LM/
	- memory constraints
		- parallelism
		- activation checkpointing
		- tensor offloading
		- automatic mixed precision
- NeMo framework = NVIDIA pipeline for AI model training
	- NeMo curator: https://github.com/NVIDIA-NeMo/Curator
	- NeMo RL
	- NeMo evaluator
	- NeMo retriever
	- NeMo guardrails
- NeMo features
	- end-to-end pipeline
	- accelerated performance
	- production ready
	- run anywhere
	- increased ROI

Lab 1
- SLURM class environment
- multi-GPU and multi-node GPT pre-training with NeMo
- profile with pytorch_profiler and optimize GPT model pre-training

```python
!lscpu # display CPU info
!grp 'cpu cores' /proc/cpuinfo | uniq # number of CPU cores
!nvidia-smi # display GPU utilization

# multi-GPU config needs fast and scalable interconnect
# NVLink = direct GPU-to-GPU interconnect
!nvidia-smi topo --matrix # check interconnect toplogy
!nvidia-smi nvlink --status # check NVLink status

# p2pBandwidthLatencyTest = demo CUDA peer-to-pee data transfer b/w pairs of GPUs
# compute bandwidth + latency w/ or w/o NVLinks
# Tests on GPU pairs using P2P and without P2P 
git clone --depth 1 --branch v11.2 https://github.com/NVIDIA/cuda-samples.git`
!/dli/cuda-samples/bin/x86_64/linux/release/p2pBandwidthLatencyTest

# compute nodes managed by cluster manager (e.g. SLURM workload manager)
# open source cluster management + job scheduler system for large and small Linux clusters
!sinfo # check slurm config

# submit jobs w/ srun
!srun -N 1 nvidia-smi
# srun = run parallel jobs
# -N = specify nodes allocated
# -G = specify GPUs within node to allocate

# submit jobs w/ sbatch
!sbatch /dli/code/test_sbatch.sbatch
# transfer execution to SLURM manager after auto-populating the args for execution
JOBID=!squeue -u root | grep dli | awk '{print $1}' # get Job ID
slurm_job_output='/dli/nemo/logs/'+JOBID[0]+'.out' # get output
!squeue # check SLURM scheduling queue
```

`test_sbatch.sbatch`
```bash
#!/bin/bash

#SBATCH -N 1
#SBATCH --job-name=dli_firstSlurmJob
#SBATCH -o /dli/nemo/logs/%j.out
#SBATCH -e /dli/nemo/logs/%j.err

srun -l /dli/code/test.sh 
```
SLURM docs: https://slurm.schedmd.com

```python
# interactive sessions with --pty
!srun -N 1 --pty /bin/bash
# session closed when exit node or cancel job
!scancel <JOBID> # cancel job
!scancel -u $USER # cancel all user's jobs
```

- memory consumption in large model training
	- model states
		- model params
		- gradients
		- optimize states (momentum + variance in Adam)
		- mixed precision keeps two copies of params
		- gradients stored in FP16
		- optimizer states stored in FP32
		- total memory = 16 x (# of params) bytes
	- residual states
		- activations
		- temporary buffers
		- unusable fragmented memory (i.e. residual states)

resources:
- https://lilianweng.github.io/posts/2021-09-25-train-large/
- https://www.deepspeed.ai/tutorials/megatron/

- 4 types of parallelism
	- data = process more in data in same time
		- standard
			- each GPU has copy of model
			- optimizer states and random seeds are the same
			- each GPU gets different batch of data
			- data sampling handled by distributed sampler
			- gradients need to be synced
			- sync done with bucketed ring-allreduce algorithm
		- distributed
			- overlaps gradient computation w/ communication so GPU is always utilized
			- replication memory explosion
		- sharded
			- each GPU gets shard of model
			- each GPU gets shard of data
			- local forward pass: gather weights from others
			- local backward pass: gather again weights from others
			- local weights shard update: sync gradients
			- discard full weights after each layer's forward pass, saving memory for subsequent layers
	- pipeline = (interlayer) split contiguous sets of layers; maximize GPU utilization in single-node
		- sequentially processed as groups of continuous model layers
		- underutilization
		- split into micro-batches
		- during backward pass, load next micro-batch
		- `p` = number of pipline stages
		- `m` = number of micro-batches
		- `t_forward` = forward step time
		- `t_backward` = backward step time
		- total time = `(m + p - 1) x (t_forward + t_backward)`
		- ideal time = `m x (t_forward + t_backward)`
		- bubble time = `(p - 1) x (t_forward + t_backward)`
		- each GPU has groups of layers that are not continuous across groups, reducing bubbles
	- tensor = (intra-layer) split individual layers across multiple GPUs; minimize latency in single-node
		- reduce memory proportional to number of workers (model dependent)
		- high compute bandwitch to overcome communication overhead
		- easier to load balance (reduce latency for inference)
		- less restrictive on batch-size; dropout = row parallel
		- good for large matrices, split column-wise
		- does not scale well beyond node boundary (always inside one node / one GPU)
		- https://github.com/facebookresearch/fairscale
		- https://github.com/tensorflow/mesh
	- sequence = train w/ longer sequences
		- distribute sequence across GPUs
		- devices share same params
		- challenge: calculating attention scores across devices as Q, K , V matrices split across GPUs
		- higher batch size and throughput during training by increasing parallel size and same communication as tensor parallelism
- activation = any tensor that is created in forward pass and necessary for gradient computation in backward pass
	- save activations that take less space and are expensive to recompute
	- recompute activations that take lots of space and are cheap to recompute
- scale out = use aggregate memory of multiple GPUs
- scale up = trade compute for memory from activations leveraging checkpoints
- mixture of experts
	- train single multilayer network to perform subtasks is different
	- interaction effects slow learning, poor generalization
	- prior knowledge of task separation is evidence for expert system
	- system composed of series of experts with router (gating network)
	- weights of experts decoupled
	- choice of objective function = how to combine experts
- flash attention
- hardware
	- mixed precision: sign + exponent + mantissa (more storage for exponent or mantissa options)

Lab 2
- NNs trained on SGD split dataset into batches to process sequentially
	- forward step: feature maps computed
	- backward pass: gradients computed and averaged to determine param updates
	- next batch processed
- data parallelism mode
	- data split across multiple machines
	- each processed by a copy of same NN hosted by each machine
	- param updates averaged from all machines
	- model updates reflected on all copies
	- implementing exchange of gradients
		- centralized method = server machine responsible for distributing data chunks, accumulating gradients, updating model params
		- decentralized method = each worker sends and collects gradients from others to aggregate and update locally
			- workers deliver computation at different speeds
			- model param updated based on worker sync points
		- relaxed method = workers operation w/ outdated params (inconsistency risk)
	- Horovod = data parallelism implementation, compatible w/ TensorFlow, Keras, PyTorch, Apache MXNet https://github.com/horovod/horovod
	- NVIDIA Apex = PyTorch extension library w/ utils to streamline distributed training + mixed precision https://nvidia.github.io/apex/
- model parallelism = split model params across multiple machines
	- train bigger models that do not fit on 1 GPU
	- cost of additional communications due to feature maps exchange
	- pipeline parallelism = cut model sequentially into pieces and assign to specific worker
		- GPipe = micro-batch parallelism https://arxiv.org/pdf/1811.06965
	- tensor parallelism = divide matrix operations across workers
		- NeMo = multi-node pre-training of transformer-based models https://github.com/NVIDIA-NeMo/NeMo

`megatron_gpt_pretraining.py`
```python
# experiment manager / logging
exp_manager.explicit_log_dir = <PATH-TO-CHECKPOINTS>
exp_manager.name=experiment_name

# strategy for distributed training params
model.tensor_model_parallel_size = 1 
model.pipeline_model_parallel_size = 1

# GPT model architecture
model.num_layers = 24
model.hidden_size = 1024
model.num_attention_heads = 16
model.encoder_seq_length = 1024
model.max_position_embeddings = 1024

# optimizer params
model.micro_batch_size = 4
model.global_batch_size = 8
model.optim.lr = 0.00015
model.optim.sched.min_lr = 0.000015
model.optim.sched.name = CosineAnnealing
model.optim.sched.max_steps = 800
model.optim.sched.warmup_steps = 80
model.optim.weight_decay = 1e-1

# trainer params
trainer.gradient_clip_val = 1.0
trainer.precision = 16
trainer.devices = 2
trainer.num_nodes = 1
trainer.max_steps = 1000
trainer.val_check_interval = 200

# tokenizer
model.tokenizer.vocab_file = $VOCAB_FILE
model.tokenizer.merge_file = $MERGE_FILE

# dataset paths and sampling probabilities
model.data.data_prefix = [concat_prob, <PATH-TO-DATA>]
```

`pretrain_gpt.sh`
```bash
# Distributed training args
NNODES=1
GPUS_PER_NODE=1
TP_SIZE=1
PP_SIZE=1

# Distributed training 
MICRO_BATCH_SIZE=16
GLOBAL_BATCH_SIZE=16 # in data parellelism only GLOBAL_BATCH_SIZE = NNODES x GPUS_PER_NODE x MICRO_BATCH_SIZE

# Model architecture 
NLAYERS=12
NHIDDEN=768
NHEADS=32
SEQ_LEN=1024

# Data Paths
VOCAB_FILE=/dli/data/GPT-2_assets/gpt2-vocab.json
MERGE_FILE=/dli/data/GPT-2_assets/gpt2-merges.txt
DATA_PATH=[1.0,/dli/data/GPT-2_assets/my-gpt2_text_document]

OUTPUT_PATH=/dli/nemo
LOGS_PATH=/dli/nemo/logs
NAME="X-NODE-X-GPU" # change accordingly


OPTIMIZER_ARGS=" \
            model.optim.name=fused_adam \
            model.optim.betas=[0.9,0.95] \
            model.optim.lr=6e-5 \
            model.optim.sched.min_lr=6e-6 \
            model.optim.sched.name=CosineAnnealing \
            +model.optim.sched.max_steps=800 \
            model.optim.sched.warmup_steps=80 \
            model.optim.weight_decay=1e-1 \
        "
        
TRAINER_ARGS=" \
            trainer.gradient_clip_val=1.0 \
            trainer.precision=32 \
            trainer.devices=$GPUS_PER_NODE \
            trainer.num_nodes=$NNODES \
            trainer.max_steps=100 \
            trainer.enable_model_summary=true \
            trainer.log_every_n_steps=10 \
            trainer.val_check_interval=20 \
            trainer.limit_val_batches=10 \
        "

GPT_ARGS=" \
            model.num_layers=$NLAYERS \
            model.hidden_size=$NHIDDEN \
            model.num_attention_heads=$NHEADS \
            model.encoder_seq_length=$SEQ_LEN \
            model.data.seq_length=$SEQ_LEN \
            model.max_position_embeddings=$SEQ_LEN \
            model.micro_batch_size=$MICRO_BATCH_SIZE \
            model.global_batch_size=$GLOBAL_BATCH_SIZE \
            model.tokenizer.vocab_file=$VOCAB_FILE \
            model.tokenizer.merge_file=$MERGE_FILE \
            model.init_method_std=0.006 \
            $OPTIMIZER_ARGS \
        "

OUTPUT_ARGS=" \
            exp_manager.explicit_log_dir=$OUTPUT_PATH/$NAME \
            exp_manager.resume_if_exists=false \
            exp_manager.name=$NAME \
        "
        
PARALLEL_ARGS=" \
            model.tensor_model_parallel_size=$TP_SIZE \
            model.pipeline_model_parallel_size=$PP_SIZE \
        "
        

export CMD=" \
            python /dli/code/NeMo/examples/nlp/language_modeling/megatron_gpt_pretraining.py \
            --config-path=/dli/code/NeMo/examples/nlp/language_modeling/conf/ \
            --config-name=megatron_gpt_config.yaml \
            $TRAINER_ARGS \
            $PARALLEL_ARGS \
            $GPT_ARGS \
            $OUTPUT_ARGS \
            model.data.data_prefix=$DATA_PATH \
            model.data.data_impl=mmap \
            model.data.splits_string=\"949,50,1\" \
        "

bash -c '$LAUNCHER $CMD' 2>&1 | tee -a $LOGS_PATH/$NAME.txt
```
```python
!srun -N 1 --pty /bin/bash
!bash ./code/pretrain_gpt.sh
# world size = number of GPUs = data_parallel size x tensor_model_parallel_size x pipeline_model_parellel_size
```

Lab 3
- for multi-node jobs, rely on SLURM scheduler (`sbatch`)
- default: NVIDIA Collective Communications Library (NCCL) distributed backend of PyTorch distributed launcher
- multi-node + data parallelism
```bash
#!/bin/bash
#SBATCH --job-name=dli_2nodes
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=2       
#SBATCH --cpus-per-task=32 ### Number of threads per task (OMP threads)
#SBATCH -o /dli/nemo/logs/%j.out
#SBATCH -e /dli/nemo/logs/%j.err

set -x -e

export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

# Distributed training args
NNODES=2
GPUS_PER_NODE=2
TP_SIZE=1
PP_SIZE=1 

# Distributed training 
MICRO_BATCH_SIZE=16
GLOBAL_BATCH_SIZE=64 # in data parallelism, GLOBAL_BATCH_SIZE = NNODES x GPUS_PER_NODE x MICRO_BATCH_SIZE

# Model architecture 
NLAYERS=12
NHIDDEN=768
NHEADS=32
SEQ_LEN=1024

# Data Paths
VOCAB_FILE=/dli/data/GPT-2_assets/gpt2-vocab.json
MERGE_FILE=/dli/data/GPT-2_assets/gpt2-merges.txt
DATA_PATH=[1.0,/dli/data/GPT-2_assets/my-gpt2_text_document]

OUTPUT_PATH=/dli/nemo
LOGS_PATH=/dli/nemo/logs
NAME="X-NODES-X-GPUS" # change accordingly


OPTIMIZER_ARGS=" \
            model.optim.name=fused_adam \
            model.optim.betas=[0.9,0.95] \
            model.optim.lr=6e-5 \
            model.optim.sched.min_lr=6e-6 \
            model.optim.sched.name=CosineAnnealing \
            +model.optim.sched.max_steps=800 \
            model.optim.sched.warmup_steps=80 \
            model.optim.weight_decay=1e-1 \
        "

# ADDED PROFILER BELOW
TRAINER_ARGS=" \
            trainer.gradient_clip_val=1.0 \
            trainer.precision=32 \
            trainer.devices=$GPUS_PER_NODE \
            trainer.num_nodes=$NNODES \
            trainer.max_steps=100 \
            trainer.enable_model_summary=true \
            trainer.log_every_n_steps=10 \
            trainer.val_check_interval=20 \
            trainer.limit_val_batches=10 \
            +trainer.use_profiler=true \
        "

GPT_ARGS=" \
            model.num_layers=$NLAYERS \
            model.hidden_size=$NHIDDEN \
            model.num_attention_heads=$NHEADS \
            model.encoder_seq_length=$SEQ_LEN \
            model.data.seq_length=$SEQ_LEN \
            model.max_position_embeddings=$SEQ_LEN \
            model.micro_batch_size=$MICRO_BATCH_SIZE \
            model.global_batch_size=$GLOBAL_BATCH_SIZE \
            model.tokenizer.vocab_file=$VOCAB_FILE \
            model.tokenizer.merge_file=$MERGE_FILE \
            model.init_method_std=0.006 \
            $OPTIMIZER_ARGS \
        "

OUTPUT_ARGS=" \
            exp_manager.explicit_log_dir=$OUTPUT_PATH/$NAME \
            exp_manager.resume_if_exists=false \
            exp_manager.name=$NAME \
        "

PARALLEL_ARGS=" \
            model.tensor_model_parallel_size=$TP_SIZE \
            model.pipeline_model_parallel_size=$PP_SIZE \
        "

export CMD=" \
            python /dli/code/NeMo/examples/nlp/language_modeling/megatron_gpt_pretraining.py \
            --config-path=/dli/code/NeMo/examples/nlp/language_modeling/conf/ \
            --config-name=megatron_gpt_config.yaml \
            $TRAINER_ARGS \
            $PARALLEL_ARGS \
            $GPT_ARGS \
            $OUTPUT_ARGS \
            model.data.data_prefix=$DATA_PATH \
            model.data.data_impl=mmap \
            model.data.splits_string=\"949,50,1\" \
        "

clear; srun --jobid $SLURM_JOBID bash -c 'NCCL_DEBUG=INFO $CMD' 2>&1 | tee -a $LOGS_PATH/$NAME.txt
```

```python
!grep Channel /dli/nemo/logs/<OUTPUT-FILENAME>.txt | grep slurmnode1 | head # check type of networking b/w GPUs and nodes during training
# internode communication = NET/Socket/0
# direct GPU-to-GPU communications = P2P/IPC
```
- add profiler to PyTorch Lightning trainer to profile memory usage
- used internally by NeMo framework
- add to `megatron_gpt_pretraining.py`:
```python
if cfg.trainer.get('use_profiler', False):
        schedule = torch.profiler.schedule(wait=50, warmup=1, active=2, repeat=1)
        profiler = PyTorchProfiler(activities=[
                torch.profiler.ProfilerActivity.CPU,
                torch.profiler.ProfilerActivity.CUDA,
            ], 
            schedule=schedule, record_shapes=True, profile_memory=True, with_stack=True, with_flops=True, with_modules=True)
        del cfg.trainer.use_profiler
        trainer = Trainer(plugins=plugins, strategy=strategy, profiler=profiler, **cfg.trainer)
    else:
        trainer = Trainer(plugins=plugins, strategy=strategy, **cfg.trainer)
```
- or add profiler via default option `trainer.profile=pytorch` or `simple` or `advanced` but this will output less information
```python
%%js
const href = window.location.hostname +'/tensorboard/';
let a = document.createElement('a');
let link = document.createTextNode('Open Tensorboard!');
a.appendChild(link);
a.href = "http://" + href;
a.style.color = "navy"
a.target = "_blank"
element.append(a);

# create a link to Tensorboard for browser
```
- tensor + pipeline parallelism
	- 2 nodes, 2 GPUs each
	- 2 GPUs for tensor parallelism (`TP_SIZE=2`)
	- 2 GPUs for pipeline parallelism (`PP_SIZE=2`)
	- no more resources remain for data parallelism
	- `GLOBAL_BATCH_SIZE` = `MODEL_BATCH_SIZE`

Lab 4
- automatic mixed precision (AMP) = use different numerical precisions when running math operations
	- perform some operations in half-precision format, reducing memory required
	- keep single-precision in critical parts of network
	- update `trainer.precision=16` in `TRAINER_ARGS`

- model inference
	- perplexity = how confused the model is by the input
	- larger sequence at training = higher training cost
- common practice:
	- train small model --> stop training when converged --> lightly compress
- optimal practice:
	- train large model --> stop training early --> heavily compress
- autoregressive models = next token prediction
- inference process
	- tokenizer
		- compute attention mechanism over entire prompt
		- create KV cache
		- longer prompt = bigger prefill (longer time to first token)
		- very fast b/c of GPU parallelization
	- autoregression
		- generate 1 token at a time
		- slowest phase of inference
		- memory bound
	- stop when `stop_token` or generated or `max_token_limit` reached
- improvements
	- KV caching (in $(Q*K^T)*V$ computation)
	- quantization: FP32 --> INT8 --> FP32
		- INT8 has 16x math throughput and 4x bandwidth reduction
	- post-training quantization (PTQ)
		- calibration data --> pre-trained model --> gather layer statistics --> compute q-params --> quantize models
		- start with pre-trained model and evaluate on calibration dataset
		- calibration data used to calibrate model (can be subset of training data)
		- calculate dynamic ranges of weights and activations in the network to compute quantization params
		- quantize network using q-params and run inference
	- quantization-aware training (QAT)
		- pre-trained model --> add QAT ops --> finetine w/ QAT ops --> store q-params --> quantize model for inference
		- start with pre-trained model and introduce quantization at various layers
		- finetune for small number of epochs
		- simulate quantization process that occurs during inference
		- learn the q-params which can help reduce accuracy drop b/w quantized and pre-trained model
	- model pruning = reduce complexity of NN by removing unnecessary connections
		- reduce memory bandwidth
		- reduce memory footprint
		- accelerate compute
		- challenge: maintain accuracy
	- structured sparsity support = accelerate inference w/ sparse tensor core
		- maximize throughput at low latency w/ sparsity
		- automatic sparsity = easy-to-use workflow to induce sparsity
		- TensorRT = accelerate inference using sparse kernels
	- distillation
```python
from apex.contrib.sparsity import ASP

# in training phase, augment model and optimizer
ASP.prune_trained_model(model, optimizer)

# in inference phase, set kSPARSE_WEIGHTS flag in IBuilderConfig
```
- distributed inferencing
	- pipeline parallelism = (inter-layer) split sets of layers across multiple devices
		- maximize GPU utilization and throughput
		- used easily with TRITON
	- tensor parallelism = (intra-layer) split individual layers across multiple devices
		- minimizes latency
- NVIDIA TensorRT
	- SDK for high performance, DL inference
	- maximize throughout for latency-critical apps with compiler and runtime
	- general purpose
	- features
		- reduced mixed precision
		- layer and tensor fusion (optimize use of GU memory bandwidth)
		- kernel auto-tuning (select best algorithm on target GPU)
		- dynamic tensor memory
		- multi-stream execution
		- time fusion (optimize RNN over time steps)
	- TensorRT-LLM = TensorRT ++
		- KV caching + custom MHA kernels
		- inflight batching, paged KV cache (attention)
		- only for LLMs
- bigger model = more sample efficient

resources:
- https://jalammar.github.io/illustrated-transformer/
- https://huggingface.co/blog/how-to-generate

Lab 8
```python
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM

# download from HuggingFace
model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-2-13b-hf")
tokenizer = AutoTokenizer.from_pretrained("meta-llama/Llama-2-13b-hf)

# local path to weights
model = AutoModelForCausalLM.from_pretrained(<LOCAL-PATH-TO-WEIGHTS>)
tokenizer = AutoTokenizer.from_pretrained(<LOCAL-PATH-TO-WEIGHTS>)

assert torch.cuda.is_available()
device = torch.device("cuda:0")
model.half().to(device)
model = model.eval() # switch to eval mode for inference

# generate random sentence
with torch.no_grad(): # turn off gradient computation
	output = model.generate(
		input_ids=None,
		max_length=128,
		num_return_sequences=1,
		pad_token_id=tokenizer.eos_token_id)
# decode sentence
for sentence in output:
    sentence = sentence.tolist()
    text = tokenizer.decode(sentence, clean_up_tokenization_spaces=True)
    print(text)

# few-show learning
input_ids = tokenizer.encode(
"Question: What is the typical color of the sky ? Answer: Blue. " \
"Question: At what temperature does water boil? Answer: Hundred degrees celcius.  " \
"Question: What is the name of the first man on the moon ? Answer: Neil Amrstrong. " \
"Question: What is the capital city of France? Answer: ",return_tensors="pt").cuda(0)

# generate 20 new tokens
greedy_output = model.generate(input_ids=input_ids, max_new_tokens=20)
# greedy decoding = take token w/ max. probability
# alternatives: Beam Search, Top-K, Top-P
# hyperparams: Temperature of logits, Repetition penalty

print("Output:\n" + 100 * '-')
print(tokenizer.decode(greedy_output[0], skip_special_tokens=True))

output = model.generate(
	input_ids=input_ids,
	max_length=80,
	num_return_sequences=1,
	num_beams=5,
	temperature=0.7,
	repetition_penalty=3.0,
    pad_token_id=tokenizer.eos_token_id)

sentence = output[0].tolist()
text = tokenizer.decode(sentence, clean_up_tokenization_spaces=True)
print(text)

# inference latency
import time

execution_time = 0
num_iterations = 10
with torch.no_grad():
    for _ in range(num_iterations):
        start = time.time()
        output = model.generate(
	        input_ids=None,
	        max_length=128,
            num_return_sequences=1,
            pad_token_id=tokenizer.eos_token_id,
            eos_token_id=50256)
        end = time.time()
        execution_time += end - start
        
print("Average inference time of 128 tokens is:",
     1000 * (execution_tiame/float(num_iterations)), "ms")
```

Lab 9

`launch_triton.py`
```python
import argparse
import subprocess
from pathlib import Path


def parse_arguments():
    parser = argparse.ArgumentParser()
    parser.add_argument('--world_size',
                        type=int,
                        default=1,
                        help='world size, only support tensor parallelism now')
    parser.add_argument('--tritonserver',
                        type=str,
                        default='/opt/tritonserver/bin/tritonserver')
    path = str(Path(__file__).parent.absolute()) + '/../all_models/gpt'
    parser.add_argument('--model_repo', type=str, default=path)
    return parser.parse_args()


def get_cmd(world_size, tritonserver, model_repo):
    cmd = 'mpirun --allow-run-as-root '
    for i in range(world_size):
        cmd += ' -n 1 {} --model-repository={} --disable-auto-complete-config --backend-config=python,shm-region-prefix-name=prefix{}_ : '.format(
            tritonserver, model_repo, i)
    cmd += '&'
    return cmd


if __name__ == '__main__':
    args = parse_arguments()
    cmd = get_cmd(int(args.world_size), args.tritonserver, args.model_repo)
    subprocess.call(cmd, shell=True)
```

---
data pruning of common crawl data
- URL filtering
- text extraction
- language identification
- deduplication
- document-wise filtering
- line-wise correction
- fuzzy deduplication
- exact deduplication

model-based filtering = remove artifacts

NeMo framework = NVIDIA pipeline for AI model training
- NeMo curator: https://github.com/NVIDIA-NeMo/Curator
	- text
		- download
		- pre-process
		- synthetic data generation*
		- quality filtering*
		- deduplication*
		- blending / shuffling
	- image
		- download
		- pre-process
		- quality filtering*
		- demand deduplication*
		- captioning*
	- video
		- splitting and transcoding
		- quality filtering*
		- annotation
		- semantic deduplication*
		- dataset creation
	- classifier models: https://developer.nvidia.com/blog/enhance-your-training-data-with-new-nvidia-nemo-curator-classifier-models/
		- style
			- domain
			- quality
		- type of task
			- task + complexity
			- instruction data guard
			- multilingual
			- content type classifier
- NeMo RL
- NeMo evaluator
- NeMo retriever
- NeMo guardrails