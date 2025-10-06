# RM-01 Developer Guide

**Read Before Use:**

RM-01 consists of an **Inference Module**, an **Application Module**, and an **Encryption and Management Chip** (hereinafter referred to as the Management Module), interconnected via an **onboard Ethernet switch chip**, forming an internal LAN subnet. When a user connects to a host (e.g., PC, smartphone, iPad) via the **USB Type-C** interface, RM-01 will virtualize an Ethernet interface for the host through USB Ethernet functionality. The host will then obtain an IP address and automatically join the subnet for data interaction.

After the device is powered on and connected to the host via the **USB Type-C** interface, the system will automatically configure the local network subnet. The **user host** will be assigned a static IP address `10.10.99.100`. The **Inference Module** (IP: `10.10.99.98`) and the **Application Module** (IP: `10.10.99.99`)—both deploy independent SSH services, allowing users to access them directly via standard SSH clients (e.g., OpenSSH, PuTTY). The Management Module, however, requires access via a serial port tool.

---

## How to Provide Internet Access to RM-01 from the Host (Using macOS as an Example)

After connecting the **user host** via USB Type-C, RM-01 will appear in the network interface list as:  
- **`AX88179A`** (Developer Version)  
- **`RMinte RM-01`** (Commercial Release Version)

Please follow these steps to configure internet sharing:

1. Open **System Settings**  
2. Go to **Network** → **Sharing**  
3. Enable **Internet Sharing**  
4. Click the **"i" icon** next to the sharing settings to enter the configuration interface:  
   - Set **"Share your connection from"** to: **Wi-Fi**  
   - In **"To computers using"**, select: **AX88179A** or **RMinte RM-01** (depending on the device model)  
5. Click **Done**

Then, return to the **Network** settings page and manually configure the RM-01 network interface:  
- **IP Address**: `10.10.99.100`  
- **Subnet Mask**: `255.255.255.0`  
- **Router**: `10.10.99.100` (i.e., the host’s own IP)

> **Note**:  
> This configuration sets the host as a gateway, providing NAT network access for RM-01. The default gateway and DNS for RM-01 are automatically assigned by the host via DHCP. Manually setting the IP ensures that it remains within the `10.10.99.0/24` subnet, consistent with the device’s internal service communication.

---

## About the **CFexpress Type-B** Storage Card

The **CFexpress Type-B** storage card is one of the core components of the RM-01 device, responsible for system boot, deployment of the inference framework, and key functions such as ISV/SV software distribution and authorization authentication.

The storage card is divided into three independent partitions, namely:

`rm01sys` `rm01app` `rm01models`

`rm01sys` — System Partition  
The operating system and core runtime environment of the **Inference Module** are installed in this partition. Users or developers are strictly prohibited from accessing, modifying, or deleting the contents of this partition. Any unauthorized changes may cause the Inference Module to fail to boot or render inference functions inoperable, and any resulting hardware or software damage is not covered by any warranty services.

`rm01app` — Application Partition  
This partition is used to temporarily store `Docker` image files submitted by users or developers. After the image is written to `rm01app`, the RM-01 system will automatically migrate it to the host’s built-in **NVMe SSD** storage and complete containerized deployment. Do not directly run or modify application files in this partition.

`rm01models` — Model Partition  
This partition is dedicated to storing large-scale AI models (e.g., LLMs, multimodal models, etc.) loaded by users or developers. For details on model formats, size limitations, loading procedures, and compatibility requirements, refer to the “About Models” section below.

---

## About the Application Module

IP Address: `10.10.99.99`  
Port Range: `59000-59299`

Application Module Hardware Specifications:

```
Processor: Intel Core i3-N305 (8 cores, 8 threads, base frequency 1.8 GHz, max turbo frequency 3.8 GHz)  
Memory: 16 GB / 24 GB LPDDR5-4800MT/s (onboard, non-expandable)  
Storage: 512 GB / 1 TB / 2 TB (optional) NVMe SSD  
```

Application Module SSH Access Credentials:

```
Default Username: rm01  
Default Password: rm01 (factory preset, for initial login only)  
```

**Security Notice:**  
To ensure system security, immediately use the `passwd` command to change the default password after the first SSH login. The default password is only for initial configuration and must not be used in production or deployment environments.

---

## About the Inference Module

IP Address: `10.10.99.98`  
Service Port Range: `58000–58999`

The **Inference Module** is the core computing unit of RM-01, supporting various high-performance AI inference configurations. Users can select the appropriate model based on model scale and performance requirements. The four factory-preset hardware configurations are as follows:

| Memory | Memory Bandwidth | Compute Power        | Tensor Core Count |
|--------|------------------|:--------------------|:------------------|
| 32 GB  | 204.8 GB/s       | 200 TOPS (INT8)     | 56                |
| 64 GB  | 204.8 GB/s       | 275 TOPS (INT8)     | 64                |
| 64 GB  | 273 GB/s         | 1,200 TFLOPS (FP4)  | 64                |
| 128 GB | 273 GB/s         | 2,070 TFLOPS (FP4)  | 96                |

RM-01 comes pre-installed with the following two inference frameworks on the **CFexpress Type-B** storage card, both running on the Inference Module:

`vLLM`: Automatically starts, listens on port **58000** by default, provides OpenAI-compatible API interfaces, and supports standard requests such as POST /v1/chat/completions.  
`TEI` (Text Embedding Inference): Requires manual startup, used for text embedding services.

API Access Method:  
After successfully loading a model, the **vLLM** inference service can be accessed via the following address:  
`http://10.10.99.98:58000/v1/chat/completions`  
It supports direct calls using standard OpenAI clients (e.g., openai-python, curl, Postman).

**Security Notice:**  
To ensure system security and stability, the Inference Module does not provide SSH access permissions. Users and developers cannot directly log in or interactively operate the underlying operating system of this module. Any attempts to bypass security policies or directly access the Inference Module may result in system anomalies, data corruption, or service interruptions, which are not covered by warranty services.

---

## About Models

RM-01 supports inference for various AI models, including but not limited to: **Large Language Models (LLM)**, **Multimodal Models (MLM)**, **Vision-Language Models (VLM)**, **Text Embedding Models (Embedding)**, and **Reranker Models (Reranker)**. All model files must be stored on the device’s built-in **CFexpress Type-B** storage card, and users need to use a compatible **CFexpress Type-B** card reader to upload, manage, and update models on the host side.

When the **CFexpress Type-B** storage card is connected to RM-01, the system mounts it as a read-only data volume named `models` at the path `/home/rm01/models`. Its standard file structure is as follows:

```bash
models/
├── auto/                  # Directory for automatic model loading (production-grade deployment)
│   ├── embedding/         # Embedding models (not automatically loaded, see below)
│   ├── llm/               # Large language models (weight files stored directly, see below)
│   └── reranker/          # Reranker models (not automatically loaded, see below)
└── dev/                   # Developer-defined model directory (higher priority)
    ├── embedding/
    ├── llm/
    └── reranker/
```

> **Note**:  
> - The `auto/` directory is used for **lightweight, standardized model deployment**, automatically recognized by the system.  
> - The `dev/` directory is used for **fine-grained control of model behavior by developers**, with higher priority than `auto/`. The system will ignore models in `auto/` if `dev/` is used.

---

### 1. Automatic Mode (auto)

#### Usage:  
Place the **complete weight files** of the model (e.g., `.safetensors`, `.bin`, `.pt`, `.awq`, etc.) **directly in the `auto/llm/` directory**, **without nesting in subfolders**.

> Incorrect Example:  
> `auto/llm/Qwen3-30B-A3B-Instruct-2507-AWQ/model.safetensors`  
>  
> Correct Example:  
> `auto/llm/model-001-of-006.safetensors`  
> `auto/llm/config.json`  
> `auto/llm/tokenizer.json`  
> `auto/llm/vocab.json`

#### System Behavior:  
- Upon device startup, the system scans the `auto/llm/` directory and automatically loads models in compatible formats.  
- **Automatic loading is not supported for embedding or reranker models**, only LLMs.  
- After loading, the model enables **basic inference capabilities by default** and **does not enable the following advanced features**:  
  - Speculative Decoding  
  - Prefix Caching  
  - Chunked Prefill  
  - The maximum context length (`max_model_len`) is restricted to the system’s safe threshold (typically ≤ 8192 tokens).  
- **Limited performance optimization**: To ensure system stability and multitasking concurrency, models in automatic mode use a conservative memory allocation strategy (`gpu_memory_utilization` ≤ 0.8).

> **Important Note**:  
> Automatic mode is suitable for **quickly verifying model compatibility** or **standardized deployment scenarios**, but it is **not suitable for high-performance inference in production environments**. For full performance, use **manual mode (dev)**.

---

### 2. Manual Mode (dev) — Developer Mode

> **Manual mode has higher priority than automatic mode**. When any `.yaml` configuration file exists in the `embedding`, `llm`, or `reranker` directories under `dev/`, the system will **completely ignore all models in `auto/`**.

#### Configuration File Structure:  
The `embedding`, `llm`, and `reranker` directories under `dev/` must contain the following three YAML configuration files to start the corresponding model services:

| File Name            | Purpose                              |
|----------------------|--------------------------------------|
| `embedding_run.yaml` | Start text embedding model service   |
| `llm_run.yaml`       | Start large language model service    |
| `reranker_run.yaml`  | Start reranker model service          |

> All files must be located in the corresponding folders under `dev/`, and **file names must match exactly**, case-sensitive.

#### Example Configuration File (`llm_run.yaml`):

```yaml
model: /home/rm01/models/dev/llm/Qwen3-30B-A3B-Instruct-2507-AWQ
port: 58000
gpu_memory_utilization: 0.85
max_model_len: 24576
served_model_name: RM-01 LLM
enable_prefix_caching: true
enable_chunked_prefill: true
max_num_batched_tokens: 512
block_size: 16
tensor_parallel_size: 1
dtype: auto
```

> **Note**:  
> - The `model` path must be an **absolute path** pointing to the model file directory (not a compressed file).  
> - The `port` must be within the `58000–58999` range and must not conflict with other services.  
> - The `gpu_memory_utilization` is recommended to be set between 0.7–0.9 to maximize throughput.  
> - All parameters follow the **vLLM official documentation** `https://docs.vllm.ai/en/latest/index.html`. Ensure compatibility with the vLLM version currently used on the **CFexpress Type-B** storage card.

#### Startup Process:  
1. Copy the model files (complete directory) to `dev/llm/` (or `embedding/`, `reranker/`).  
2. Create and place the corresponding `.yaml` configuration file.  
3. Insert the **CFexpress Type-B** storage card and restart RM-01.  
4. The system will automatically load the configuration in `dev/` and start the corresponding service.  
5. Once the service is started, it can be accessed via interfaces like `http://10.10.99.98:58000/v1/chat/completions`.

> **Recommendation**: For initial deployment, use the `--load-format auto` and `--dtype auto` options provided by vLLM to automatically adapt to the model format.

---

### Security and Maintenance Notes

- **Prohibited SSH Login to Inference Module**: All model management must be done via the **CFexpress Type-B** storage card.  
- **Model Files Must Be Raw Weights**: Do not use compressed files (.zip/.tar.gz), encrypted packages, or non-standard formats.  
- **File Permissions**: All model files must be readable (`chmod 644`), and directories must be executable (`chmod 755`).  
- **Version Control**: It is recommended to use Git or file naming conventions (e.g., `Qwen3-30B-A3B-Instruct-v1.2-20250930`) to manage model versions.  
- **Backup Recommendation**: Back up the `dev/` and `auto/` directories before updating models to avoid configuration loss.

---

### Mode Selection Recommendations

| Scenario                        | Recommended Mode                     |
|--------------------------------|--------------------------------------|
| Quickly verify model compatibility | Automatic Mode (auto)               |
| High-performance inference in production | Manual Mode (dev) + fine-tuned configuration |
| Multi-model parallel deployment | Manual Mode (dev) + multiple `.yaml` files |
| Development debugging, prototype validation | Manual Mode (dev)                  |