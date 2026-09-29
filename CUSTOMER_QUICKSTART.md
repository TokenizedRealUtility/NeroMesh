# NEROMESH AI Work Quick Start

NEROMESH lets you submit AI work to the **TRU / NEROMESH Axon Compute Network**.

Instead of running the AI workload yourself, you submit a job through the NEROMESH AI Console. The network places the job into the queue, assigns it to an available opted-in worker, processes the request, and returns the result to you.

This guide explains how to access the AI Console, submit work, monitor a job, and retrieve the completed result.

> **Private Alpha**
>
> NEROMESH is currently operating as a private alpha.
>
> Customer access requires a NEROMESH customer access token.
>
> Billing, automated customer funding, and production settlement are not yet enabled.

---

# Step 1 — Open NEROMESH

Go to:

**https://tru.neromesh.space**

From the main NEROMESH page, click:

**LAUNCH AI CONSOLE**

![Open NEROMESH AI Console](images/customer-01-open-ai-console.png)

---

# Step 2 — Enter Your Customer Access Token

The NEROMESH AI Console requires a customer access token.

Customer tokens begin with:

```text
nmc_
```

Example:

```text
nmc_...
```

Enter your customer access token into the AI Console.

![Enter NEROMESH Customer Token](images/customer-02-access-token.png)

> Keep your customer token private.
>
> Your `nmc_...` token identifies your customer account and authorizes job submission.
>
> Do not publish it in screenshots, GitHub repositories, chat rooms, or public documentation.

---

# Step 3 — Choose the Type of Work

NEROMESH currently supports approved AI workload types.

The primary job types are:

```text
TEXT / GENERATE
```

and:

```text
TRU EVOLVE / PREVIEW
```

![Choose NEROMESH Job Type](images/customer-03-job-type.png)

---

# TEXT / GENERATE

Use **TEXT / GENERATE** for normal AI requests.

Examples include:

- Writing
- Summarization
- Brainstorming
- Classification
- Data interpretation
- Question answering
- Draft generation
- General AI inference

Enter your request into the prompt field.

Example:

```text
Explain proof-of-work blockchain mining in simple terms.
```

NEROMESH will send the approved workload to an available worker.

---

# TRU EVOLVE / PREVIEW

Use **TRU EVOLVE / PREVIEW** for TRU-native AI evolution workflows.

Depending on the evolution request, the console may ask for information such as:

```text
Token ID
Token Type
Milestone
Evolution Instructions
```

The worker performs the AI inference and returns an evolution preview.

> A preview does not automatically modify TRU Core or commit anything to the blockchain.
>
> Core evolution commits remain disabled during the current private alpha.

---

# Step 4 — Enter Your Prompt

Enter the work you want the network to perform.

![Enter AI Work](images/customer-04-enter-prompt.png)

For example:

```text
Write a short technical summary explaining how a distributed GPU
worker network can process AI inference without giving the worker
access to a customer's wallet.
```

Be as specific as possible.

A clear prompt normally produces a better result than a very short or ambiguous request.

---

# Step 5 — Set the Output Limit

Choose the maximum output size for the job.

The console may display this as:

```text
MAX OUTPUT TOKENS
```

![Set Output Tokens](images/customer-05-output-limit.png)

This limits the maximum amount of text the worker may return.

A larger value permits a longer response, while a smaller value limits the result.

---

# Step 6 — Approve Community Processing

NEROMESH uses independently operated worker machines.

Before submitting a job, you must acknowledge that your request may be processed by a participating community worker.

![Community Processing Consent](images/customer-06-community-processing.png)

Only submit information that is appropriate for processing by the NEROMESH network.

Do not place passwords, wallet seeds, private keys, API secrets, or other sensitive credentials inside an AI prompt.

---

# Step 7 — Submit the Job

When your request is ready, click:

**SUBMIT JOB**

![Submit NEROMESH Job](images/customer-07-submit-job.png)

NEROMESH creates a unique job and places it into the network queue.

The basic flow is:

```text
Customer
   │
   ▼
AI Console
   │
   ▼
NEROMESH Coordinator
   │
   ▼
Job Queue
   │
   ▼
Available Worker
   │
   ▼
AI Inference
   │
   ▼
Result
   │
   ▼
Customer
```

---

# Step 8 — Watch the Job Status

After submission, the job appears under:

**YOUR JOBS**

![NEROMESH Job Queue](images/customer-08-job-queue.png)

A job normally progresses through these states:

```text
QUEUED
   ↓
LEASED
   ↓
COMPLETED
```

## QUEUED

```text
QUEUED
```

means NEROMESH has accepted your request and is waiting for an appropriate worker.

If no worker is currently available, the job remains queued.

## LEASED

```text
LEASED
```

means a NEROMESH worker has accepted the job and is processing it.

At this point the job is actively assigned to a worker.

## COMPLETED

```text
COMPLETED
```

means the worker successfully processed the workload and returned the result to NEROMESH.

The result can now be viewed from the AI Console.

## FAILED

A job may also show:

```text
FAILED
```

if processing could not be completed.

A failed job does not indicate that your NEROMESH customer account has failed.

The job itself can be inspected for additional information.

---

# Step 9 — View Your Result

Once a job shows:

```text
COMPLETED
```

click:

**VIEW RESULT / DETAILS**

![View NEROMESH Result](images/customer-09-view-result.png)

The returned AI output will be displayed with the job.

The job details may also include information about the workload and its processing state.

---

# Step 10 — Cancel a Queued Job

A job that is still:

```text
QUEUED
```

may be cancelled before a worker accepts it.

Use the available:

**CANCEL**

control for the queued job.

![Cancel NEROMESH Job](images/customer-10-cancel-job.png)

Once the job has been leased to a worker, it can no longer be cancelled through the normal queued-job cancellation process.

---

# Understanding the Job Lifecycle

The NEROMESH workload lifecycle is:

```text
CREATE
  │
  ▼
QUEUED
  │
  │ Worker becomes available
  ▼
LEASED
  │
  │ Worker performs inference
  ▼
COMPLETED
  │
  ▼
RESULT
```

If processing fails:

```text
LEASED
  │
  ▼
FAILED
```

NEROMESH keeps the customer request, worker assignment and returned result separated so that the coordinator can track the lifecycle of each job.

---

# What Happens Behind the Scenes?

When you submit a job, your computer does not directly connect to somebody else's GPU.

Instead:

```text
Your Browser
     │
     │ HTTPS
     ▼
tru.neromesh.space
     │
     ▼
Cloudflare
     │
     ▼
NEROMESH Coordinator
     │
     ▼
Scheduler
     │
     ▼
Available AXON Worker
     │
     ▼
Approved Local AI Model
     │
     ▼
Result Returned
     │
     ▼
NEROMESH
     │
     ▼
Your AI Console
```

Workers remain connected to NEROMESH and request approved work from the coordinator.

Customers do not need to know the worker's IP address or directly communicate with the worker machine.

---

# What Is AXON?

NEROMESH calls its distributed compute layer the:

**Axon Compute Network**

An axon is the part of a biological neuron that carries signals between cells.

NEROMESH uses the concept as a model for distributed compute:

```text
Worker Machine     = Neuron
Network Connection = Axon
AI Job             = Signal
NEROMESH           = Coordinating Mesh
```

AXON connects independent compute into coordinated intelligence.

---

# Current Approved Model

The current private-alpha NEROMESH worker uses the approved model:

```text
Qwen2.5-0.5B-Instruct
```

with the approved GGUF configuration:

```text
qwen2.5-0.5b-instruct-q4_k_m
```

The model runs locally on participating NEROMESH worker hardware.

NEROMESH verifies the approved model package before a worker can use it.

---

# Customer Credentials

Customer access is separate from worker enrollment.

NEROMESH currently uses different credentials for different roles:

| Credential | Prefix | Purpose |
|---|---|---|
| Customer Access Token | `nmc_...` | Submit and view customer jobs |
| Worker Install Invitation | `nmi_...` | Enroll a worker machine |
| Worker Account Key | `nmp_...` | Access a worker's private account |
| TRU Address | `TT...` | Public TRU receiving address |

If you are only submitting AI work, you use:

```text
nmc_...
```

You do not need a worker installation key.

You do not need a worker account key.

You do not need to provide a TRU wallet private key.

---

# Important Security Information

Never place any of the following inside a NEROMESH prompt:

```text
Wallet seed phrases
Private keys
Passwords
Private API keys
Authentication secrets
Recovery phrases
Sensitive account credentials
```

NEROMESH jobs may be processed on independently operated machines participating in the Axon Compute Network.

Only submit information appropriate for distributed processing.

---

# Customer Access During Private Alpha

NEROMESH is currently in:

```text
PRIVATE ALPHA
```

Customer access is presently issued by the NEROMESH operator.

During the private alpha:

```text
Community workers           ENABLED
Durable jobs                ENABLED
AI inference                ENABLED

Automated customer billing  NOT YET ENABLED
Automated TRU settlement    NOT YET ENABLED
Wallet pairing              DISABLED
Core evolution commits      DISABLED
```

The current system is designed to validate the compute network, worker scheduling, inference pipeline, result delivery and future settlement architecture before production payment automation is enabled.

---

# No Worker Available?

If your job remains:

```text
QUEUED
```

it generally means an appropriate worker has not accepted the job yet.

Do not repeatedly resubmit the same request.

Once an eligible worker becomes available, NEROMESH can lease the queued job for processing.

---

# Simple Example

A customer enters:

```text
Explain the difference between proof-of-work and proof-of-stake
in language suitable for someone new to blockchain.
```

NEROMESH creates:

```text
JOB
│
├── Type: text.generate
├── Status: QUEUED
└── Customer: authenticated
```

An AXON worker becomes available:

```text
QUEUED
   ↓
LEASED
```

The worker runs the approved AI model locally and returns the generated response:

```text
LEASED
   ↓
COMPLETED
```

The customer opens:

```text
VIEW RESULT / DETAILS
```

and receives the finished output.

---

# Quick Start

The entire customer process is:

```text
Visit tru.neromesh.space
          │
          ▼
Launch AI Console
          │
          ▼
Enter nmc_ Customer Token
          │
          ▼
Choose Job Type
          │
          ▼
Enter Prompt
          │
          ▼
Set Maximum Output
          │
          ▼
Accept Community Processing
          │
          ▼
Submit Job
          │
          ▼
QUEUED
          │
          ▼
Worker Accepts Job
          │
          ▼
LEASED
          │
          ▼
AI Runs on Worker
          │
          ▼
COMPLETED
          │
          ▼
View Result
```

---

# NEROMESH

**The Axon Compute Network**

Independent compute. Coordinated intelligence.

**Website**

https://tru.neromesh.space
