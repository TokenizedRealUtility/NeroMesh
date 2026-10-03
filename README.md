# NEROMESH / AXON Compute Network

**Independent compute. Coordinated intelligence.**

NEROMESH is the coordination layer for the **AXON Compute Network**, a distributed AI compute marketplace integrated with the **TRU blockchain**.

Customers post AI work, independent workers explicitly choose jobs they want to run, TRU wallets fund non-custodial HTLC settlement contracts, workers execute approved AI workloads locally, and customers review the returned result before the final settlement leg unlocks.

This README replaces the older separate Customer and Worker quick-start guides and documents the current **UX-04 portal architecture**, **wallet-handoff funding flow**, **worker CLI**, **manual HTLC claiming flow**, and common recovery procedures.

---

## Current Release

```text
NEROMESH release: 0.5.0-axon-portals
Phase: public marketplace portals
```

Current public platform:

```text
https://tru.neromesh.space
```

Primary pages:

```text
/marketplace   Public marketplace + create work
/customer      Customer dashboard + notifications + approvals
/worker        Worker dashboard + offers + settlement
/network       Public AXON worker/network status
```

Current major capabilities:

```text
Durable jobs                         ENABLED
Community workers                    ENABLED
AI inference                         ENABLED
Public customer signup               ENABLED
AXON marketplace                     ENABLED
Non-custodial HTLC settlement        ENABLED
Automatic funding reconciliation     ENABLED
Wallet handoff intents               ENABLED
TRU deep-link / handoff URI          ENABLED
Wallet handoff QR                    ENABLED
Customer portal                      ENABLED
Worker portal                        ENABLED
Network dashboard                    ENABLED
Portal notifications                 ENABLED
Persistent wallet pairing            DISABLED
Automatic worker payout signing      DISABLED
GPU attestation                      DISABLED
Core evolution commit bridge         DISABLED
```

Persistent wallet pairing is intentionally **not** required. Funding is handed off to the customer's own TRU wallet/Core, and signing remains local.

---

# 1. What Is AXON?

**AXON** is the distributed compute layer used by NEROMESH.

A useful mental model is:

```text
Worker Machine      = Compute Node
NEROMESH            = Coordinator
AXON Job             = Work Request
TRU                  = Settlement Chain
HTLCs                = Non-custodial Payment Contracts
```

Customers do not connect directly to a worker's GPU or need to know a worker's IP address.

The normal path is:

```text
Customer
   │
   ▼
NEROMESH Marketplace
   │
   ▼
Worker explicitly accepts
   │
   ▼
Customer funds through local TRU wallet/Core
   │
   ▼
NEROMESH verifies funding
   │
   ▼
Worker performs AI inference
   │
   ▼
Result returned
   │
   ├── 20% compute reward unlocks
   │
   ▼
Customer reviews result
   │
   ▼
Customer accepts
   │
   ├── 80% acceptance reward unlocks
   │
   ▼
Worker manually claims rewards in TRU Core
```

---

# 2. Security Model

NEROMESH is designed so the coordinator does **not** need custody of user wallet keys.

## Never give NEROMESH

```text
TRU private keys
TRU seed phrases
wallet backup files
wallet encryption passwords
API secrets
authentication secrets unrelated to NEROMESH
```

Do not put those items inside AI prompts either.

Workers are independently operated machines. Only submit data appropriate for distributed processing.

## Credentials and identifiers

NEROMESH uses several intentionally different values:

| Item | Typical Prefix / Form | Purpose | Secret? |
|---|---|---|---|
| Customer Access Token | `nmc_...` | Customer portal and job authorization | **Yes** |
| Worker Install Invitation | `nmi_...` | One-use worker machine enrollment | **Yes** |
| Worker Account Key | `nmp_...` | Worker portal/account authorization | **Yes** |
| Worker Claim Public Key | compressed `02...` / `03...` pubkey | HTLC worker claim destination | No |
| TRU Receiving Address | TRU address | Public receiving identity | No |
| AXON Job ID | `AXN-...` | Marketplace job identifier | No |
| Wallet Handoff Intent | `AXI-...` | Short-lived funding intent identifier | Usually safe alone |
| Wallet Capability | `axc_...` | Authorizes a short-lived wallet handoff | **Yes until expired** |

### Browser sessions

With UX-04, customers and workers enter `nmc_...` or `nmp_...` once and receive a short-lived web session.

The site should not persist the original credential in:

```text
localStorage
sessionStorage
IndexedDB
URLs
normal browser cookies
```

The portal session itself is represented by a short-lived secure server session.

### Wallet handoff capability

A complete funding URI may look conceptually like:

```text
tru://axon/fund/AXI-...?...#cap=axc_...
```

Treat the **full URI as private until it expires**.

Do not post it in chat, screenshots, logs, GitHub issues, or public support channels.

---

# 3. Platform Layout

The current NEROMESH site separates the platform into four roles/views.

## Marketplace

```text
https://tru.neromesh.space/marketplace
```

Use Marketplace to:

```text
Create new AI work
Browse public open jobs
Review marketplace opportunities
See job/reward summaries
```

Marketplace is no longer the primary place to repeatedly monitor your private work.

---

## Customer

```text
https://tru.neromesh.space/customer
```

Use Customer to:

```text
Sign in once with nmc_...
View your jobs
See items requiring action
Create funding handoffs
Review results
Accept or reject results
Read notifications
Review completed/history states
```

---

## Worker

```text
https://tru.neromesh.space/worker
```

Use Worker to:

```text
Sign in once with nmp_...
See worker connection status
Browse open offers
Accept or decline jobs
View active work
Inspect settlement state
Copy claim TXIDs and preimages
See reward notifications
```

The worker web interface is a GUI over the same marketplace logic available through the CLI.

---

## Network

```text
https://tru.neromesh.space/network
```

Use Network to view public compute availability such as:

```text
connected workers
READY / COMPUTING / PAUSED / STALE / OFFLINE state
worker hardware summaries
supported capabilities
completed-job totals
aggregate network status
```

Private worker credentials and private IP information should not be exposed publicly.

---

# 4. Customer Quick Start

## Step 1 — Obtain customer access

A customer account uses a token beginning with:

```text
nmc_
```

Keep it private.

Go to:

```text
https://tru.neromesh.space/customer
```

and sign in.

With UX-04, you should not need to re-enter the customer token every time you move between your jobs during the active portal session.

---

## Step 2 — Create work

Open:

```text
https://tru.neromesh.space/marketplace
```

Choose **Create Work**.

For general AI work, use:

```text
text.generate
```

A typical request includes:

```text
Prompt
Maximum output tokens
Reward
```

The current marketplace settlement split is:

```text
20% compute reward
80% acceptance reward
```

Example:

```text
Total reward       0.01000000 TRU
Compute 20%        0.00200000 TRU
Acceptance 80%     0.00800000 TRU
```

Submit the job.

NEROMESH assigns an ID similar to:

```text
AXN-508EE6428A51
```

---

# 5. Marketplace Job Lifecycle

The current marketplace states are different from the older queue-only alpha lifecycle.

Normal AXON marketplace flow:

```text
OPEN
  ↓
RESERVED
  ↓
AWAITING_FUNDING / FUNDING_PENDING
  ↓
FUNDED
  ↓
RUNNING
  ↓
WAITING_ACCEPTANCE
  ↓
ACCEPTED
```

Other possible terminal/recovery states can include:

```text
REJECTED
EXPIRED
FAILED
REFUNDABLE
```

## OPEN

The job is visible to eligible workers.

## RESERVED

A worker explicitly accepted the offer.

The job is no longer a public open offer.

## FUNDING_PENDING

The customer wallet has broadcast funding and NEROMESH is verifying the exact HTLC outpoints, scripts, amounts, propagation, and active-chain state.

**Do not fund the job again.**

## FUNDED

Both canonical settlement legs have been verified.

## RUNNING

The assigned worker is executing the AI workload.

## WAITING_ACCEPTANCE

The worker returned a valid result.

At this point:

```text
20% compute preimage = available to worker
80% acceptance preimage = still locked
```

## ACCEPTED

The customer approved the result.

Now:

```text
20% compute preimage = available
80% acceptance preimage = available
```

The worker can manually claim both HTLC outputs.

---

# 6. Customer Funding — Wallet Handoff

Once a worker has reserved a job, the Customer dashboard should show a funding action.

Typical flow:

```text
Worker found
   ↓
Customer chooses / swipes FUND
   ↓
NEROMESH creates AXON_WALLET_HANDOFF_V1
   ↓
Customer copies or opens the tru:// handoff
   ↓
TRU Core/Desktop reviews the funding plan
   ↓
Customer types FUND
   ↓
Core signs locally
   ↓
20% HTLC broadcast
80% HTLC broadcast
   ↓
NEROMESH automatically reconciles them
```

NEROMESH does **not** receive the customer's wallet private key.

---

# 7. Funding from Headless TRU Core

For a headless TRU Core user, copy the complete wallet handoff URI from the Customer portal.

Do not publish the URI.

Attach to your running Core session if needed.

Example:

```bash
screen -d -r new_tru
```

Navigate to:

```text
AXON Compute Marketplace
```

then:

```text
6. Fund Wallet Handoff (tru:// AXI)
```

Paste the complete `tru://...` handoff **locally into TRU Core**.

Core should display a review similar to:

```text
Intent
Job
Work type
Total
Compute 20%
Accept 80%
Worker public claim key
Intent fingerprint
```

Review it carefully.

Then type:

```text
FUND
```

TRU Core:

```text
derives your refund public key locally
verifies/finalizes the funding plan
builds canonical 20% and 80% HTLCs
signs locally
broadcasts locally
returns only public funding outpoints
```

Successful funding should show two outpoints and:

```text
State : FUNDING_PENDING
```

At that point:

> **DO NOT FUND AGAIN.**

NEROMESH monitors the submitted funding automatically.

---

# 8. Funding Journal and Duplicate Protection

Headless Core records non-secret handoff recovery information under:

```text
~/.tru/axon-wallet-handoffs/
```

Example:

```text
~/.tru/axon-wallet-handoffs/AXI-XXXXXXXXXXXXXXXX.json
```

The journal may contain:

```text
AXI intent ID
AXN job ID
intent fingerprint
refund public key
refund time
compute funding TXID/vout
acceptance funding TXID/vout
```

It must not contain:

```text
wallet private key
wallet seed
nmc_ token
nmp_ key
axc_ capability
HTLC claim private key
```

If Core detects that a handoff was already funded or partially funded, it should reuse/recover the existing outpoints rather than create duplicate funding.

---

# 9. Customer Notifications

The Customer dashboard now provides a dedicated notification layer.

Typical notifications:

```text
Worker accepted your job
Funding required
Funding confirmed
Compute started
Result ready
Customer approval required
Job completed
Refund available
Job failed
```

Use the notification bell and **Needs Action** section instead of repeatedly searching Marketplace and re-entering the customer token.

---

# 10. Reviewing and Accepting a Result

When compute completes, the customer should see:

```text
RESULT READY
```

Review the returned AI output.

If the result is acceptable, use:

```text
SWIPE TO ACCEPT RESULT
```

Acceptance changes the settlement from:

```text
WAITING_ACCEPTANCE
```

to:

```text
ACCEPTED
```

and releases the worker's 80% acceptance secret.

Customer acceptance does **not** give NEROMESH custody of the worker wallet.

---

# 11. Becoming an AXON Worker

Workers contribute their own computer to the NEROMESH network.

The current Linux worker flow is:

```text
Visit NEROMESH
   ↓
Join the Mesh
   ↓
Register worker
   ↓
Receive nmi_ install invitation
   ↓
Receive nmp_ worker account key
   ↓
Download worker bundle
   ↓
Install worker
   ↓
Redeem nmi_ one time
   ↓
Worker connects
   ↓
Sign into Worker portal with nmp_
   ↓
Set/verify public TRU claim key
   ↓
Browse and accept work
```

---

# 12. Worker Registration

Open:

```text
https://tru.neromesh.space
```

Choose:

```text
JOIN THE MESH
```

Provide the requested public worker information.

You will receive two different private credentials:

```text
nmi_...   one-use machine installation invitation
nmp_...   worker account / portal key
```

Keep both private.

Do not substitute one for the other.

---

# 13. Install the Worker Bundle

Download and extract:

```text
neromesh-worker-bundle.zip
```

Example:

```bash
cd ~/Downloads
unzip neromesh-worker-bundle.zip
cd neromesh-worker-bundle
```

Standard Linux installation:

```bash
bash install/worker-linux.sh \
  --gateway https://tru.neromesh.space \
  --accept-model-license \
  --start
```

The installer privately requests:

```text
nmi_...
```

Use the **one-use install invitation**, not the `nmp_` worker portal key.

The standard approved worker package currently uses:

```text
Qwen2.5-0.5B-Instruct
qwen2.5-0.5b-instruct-q4_k_m
```

---

# 14. NVIDIA / CUDA Worker Installation

Only choose CUDA mode when your NVIDIA/CUDA environment already works.

First verify:

```bash
nvidia-smi
nvcc --version
```

Then install:

```bash
bash install/worker-linux.sh \
  --gateway https://tru.neromesh.space \
  --accept-model-license \
  --cuda \
  --start
```

---

# 15. Worker Files

Installed worker application:

```text
~/.local/share/neromesh-worker
```

Worker private configuration:

```text
~/.config/neromesh/worker.json
```

Never upload `worker.json` to GitHub or publish it in screenshots.

It contains machine/account credentials.

---

# 16. Starting the Worker

Start manually:

```bash
~/.local/share/neromesh-worker/run-worker.sh
```

Or:

```bash
cd ~/.local/share/neromesh-worker
./run-worker.sh
```

The worker should display detected hardware and the NEROMESH gateway.

A connected worker remains available to receive work that it has explicitly accepted.

---

# 17. Worker Status CLI

From the worker machine:

```bash
cd ~/.local/share/neromesh-worker

python3 -m neromesh.worker status
```

This displays the enrolled machine and current worker state.

---

# 18. Worker Marketplace CLI

The CLI remains fully supported after UX-04.

Run commands from:

```bash
cd ~/.local/share/neromesh-worker
```

## List public open offers

```bash
python3 -m neromesh.worker marketplace list
```

Typical result:

```text
AXON OFFER: AXN-XXXXXXXXXXXX text.generate 0.01000000 TRU
  20% compute 0.00200000 / 80% acceptance 0.00800000
```

---

## Review an open job

```bash
python3 -m neromesh.worker marketplace show AXN-XXXXXXXXXXXX
```

Review:

```text
job type
prompt/input
input SHA-256
total reward
compute reward
acceptance reward
token/evolution details when applicable
```

Only accept work you actually want to perform.

---

## Accept a job

```bash
python3 -m neromesh.worker marketplace accept AXN-XXXXXXXXXXXX
```

A successful acceptance returns:

```text
state: RESERVED
worker_id: ...
worker_claim_pubkey: ...
```

The worker's configured default **public claim key** is used.

Explicit acceptance is required.

Workers are never supposed to automatically reserve all available work merely because they are online.

---

## Decline a job

```bash
python3 -m neromesh.worker marketplace decline AXN-XXXXXXXXXXXX
```

Declining does not require stopping the worker.

---

# 19. Important: `marketplace show` After Acceptance

`marketplace show` is for the **public OPEN marketplace**.

After you accept a job it is no longer OPEN.

Therefore this can legitimately return:

```text
404 Open marketplace job not found
```

after a successful reservation:

```bash
python3 -m neromesh.worker marketplace show AXN-XXXXXXXXXXXX
```

That does **not** mean the accepted job disappeared.

After reservation, inspect the private assigned settlement state instead:

```bash
python3 -m neromesh.worker marketplace settlement AXN-XXXXXXXXXXXX
```

---

# 20. Worker Settlement CLI

Use:

```bash
python3 -m neromesh.worker marketplace settlement AXN-XXXXXXXXXXXX
```

During funding you may see:

```text
RESERVED
FUNDING_PENDING
FUNDED
RUNNING
```

After the worker finishes:

```text
WAITING_ACCEPTANCE
```

with fields similar to:

```text
result_sha256
compute_funding_txid
compute_funding_vout
compute_preimage
worker_claim_pubkey
```

At `WAITING_ACCEPTANCE`, the compute leg is claimable.

The acceptance preimage should still be absent.

After the customer accepts:

```text
state: ACCEPTED
```

and settlement should expose both:

```text
compute_preimage
acceptance_preimage
```

---

# 21. Worker Portal GUI

Workers who do not want to use CLI can use:

```text
https://tru.neromesh.space/worker
```

Sign in with:

```text
nmp_...
```

The GUI supports the normal worker marketplace operations:

```text
Browse open offers
Review job
Accept
Decline
View active work
Inspect settlement
View claim-ready rewards
Copy funding TXID
Copy preimage
```

The GUI does **not** execute arbitrary shell commands.

Conceptually:

```bash
python3 -m neromesh.worker marketplace accept AXN-...
```

maps to:

```text
SWIPE TO ACCEPT JOB
```

and:

```bash
python3 -m neromesh.worker marketplace settlement AXN-...
```

maps to the worker's settlement/reward cards.

---

# 22. Worker Public Claim Key

AXON settlement requires a public worker claim key.

It is a compressed TRU public key similar to:

```text
02...
```

or:

```text
03...
```

The **private key stays inside the worker's TRU wallet**.

To save a default public claim key through CLI:

```bash
python3 -m neromesh.worker marketplace set-claim-pubkey <COMPRESSED_PUBLIC_KEY>
```

After it is saved, normal accept commands need only:

```bash
python3 -m neromesh.worker marketplace accept AXN-XXXXXXXXXXXX
```

Do not export or paste the private key.

---

# 23. Automatic Worker Execution

Once all conditions are satisfied:

```text
worker explicitly reserved the job
        +
both funding HTLCs verified
        +
worker is online
```

the assigned worker can lease/run the bound compute job automatically.

There is no normal marketplace command:

```text
marketplace run
```

required after funding.

Keep the worker running and watch its normal runtime output.

---

# 24. Worker Reward Settlement

Each funded marketplace job currently uses two HTLC legs:

```text
20% compute HTLC
80% acceptance HTLC
```

Example:

```text
Total             0.01000000 TRU
Compute HTLC      0.00200000 TRU
Acceptance HTLC   0.00800000 TRU
```

## Compute leg

The compute secret becomes available after valid compute/result completion.

## Acceptance leg

For a normal AI job, the acceptance secret becomes available only after the customer accepts the result.

For TRU token-evolution work, additional on-chain evolution verification rules may apply before the 80% leg is released.

---

# 25. Claiming Worker Rewards in TRU Core

Worker reward claims remain deliberately **manual**.

NEROMESH provides:

```text
funding TXID
funding vout
preimage
worker public claim key
```

The worker uses their own TRU wallet/Core to sign.

Typical Core path:

```text
19  Create
11  HTLC / Atomic Swap
claim
```

For each leg enter the appropriate:

```text
Funding TXID
Recipient: me
Secret preimage
```

Core signs with the wallet key matching the configured worker claim public key.

NEROMESH never receives that private key.

After claiming, Core should report:

```text
HTLC / ATOMIC SWAP V1 CLAIMED
Branch: CLAIM
Funding Outpoint: ...
Claim TXID: ...
Release Amount: ...
Fee: ...
Status: MEMPOOL ACCEPTED
```

Mine/await confirmation and verify that the funding outpoint is spent.

---

# 26. Why Claims Are Still Manual

Manual claiming preserves a deliberate custody boundary:

```text
NEROMESH determines when a settlement secret may be released

BUT

only the worker's TRU wallet can sign the spend
```

This keeps marketplace coordination separate from wallet custody.

A future automated claiming agent could be built separately, but it is not required by the current AXON design.

---

# 27. Notifications for Workers

Worker portal notifications can include:

```text
New matching offer
Job reserved
Funding confirmed
Compute reward unlocked
Acceptance reward unlocked
Job failed
Worker offline
```

A worker can keep the CLI/runtime open while also using the web Worker portal for easier job management.

---

# 28. Using `screen` Correctly

Many Linux deployments keep TRU Core, NEROMESH, or workers running inside GNU Screen.

Attach:

```bash
screen -d -r nero_worker
```

or:

```bash
screen -d -r nero_mesh
```

or:

```bash
screen -d -r new_tru
```

Detach cleanly while leaving the process running:

```text
Ctrl+A
D
```

Do **not** use `Ctrl+Z` to leave a screen session.

`Ctrl+Z` suspends the local `screen -r` process and can produce output similar to:

```text
Stopped  screen -r nero_worker
```

---

# 29. Common Troubleshooting

## Worker accepted job, then `show` returns 404

Expected after reservation.

Use:

```bash
python3 -m neromesh.worker marketplace settlement AXN-XXXXXXXXXXXX
```

instead.

---

## Job remains OPEN

No worker has explicitly accepted it yet.

Keep the offer posted or wait for an eligible worker.

Do not create repeated duplicate jobs unless that is intentional.

---

## Job is RESERVED

The worker has been chosen, but funding has not completed.

Customer should check:

```text
/customer
```

for **Funding Required**.

---

## Job is FUNDING_PENDING

Do not fund again.

NEROMESH is verifying the existing funding outpoints.

---

## Funding commitments already frozen for a different refund key

Example:

```text
HTTP 409: Funding commitments already frozen for a different refund key
```

This means the funding plan is already bound to another customer refund public key.

Do not simply create new funding from another wallet.

First inspect the local wallet-handoff journal:

```bash
ls -lt ~/.tru/axon-wallet-handoffs/

grep -l 'AXN-XXXXXXXXXXXX' \
  ~/.tru/axon-wallet-handoffs/AXI-*.json 2>/dev/null
```

Then inspect the matching journal:

```bash
python3 -m json.tool ~/.tru/axon-wallet-handoffs/AXI-XXXXXXXXXXXXXXXX.json
```

Possible cases:

```text
no funding TXIDs
only compute TXID
both TXIDs
```

Recover the original handoff/wallet state rather than double funding.

---

## Partial funding

If the journal has:

```text
compute_txid
```

but no:

```text
acceptance_txid
```

the 20% leg already exists.

Do not create a second compute leg.

Return to the original wallet/refund key, ensure it has enough TRU for the missing leg and fees, and retry the same handoff.

Core should reuse the existing compute outpoint and create only the missing acceptance leg.

---

## Customer switched wallets after funding began

The refund key is part of the frozen contract plan.

A different wallet may derive a different refund public key and correctly trigger a conflict.

Use the original wallet for recovery unless a future explicit safe replacement flow is provided.

---

## Worker appears offline

Verify the local process:

```bash
cd ~/.local/share/neromesh-worker
python3 -m neromesh.worker status
```

Start it if needed:

```bash
~/.local/share/neromesh-worker/run-worker.sh
```

Then refresh:

```text
https://tru.neromesh.space/worker
```

or:

```text
https://tru.neromesh.space/network
```

---

## CUDA worker not using GPU

Verify:

```bash
nvidia-smi
nvcc --version
```

and confirm the worker was installed with:

```text
--cuda
```

Check the worker runtime log and detected hardware during startup.

---

## Customer result is ready but 80% is still locked

That is expected at:

```text
WAITING_ACCEPTANCE
```

The customer must review and accept the result.

After acceptance, worker settlement should show:

```text
state: ACCEPTED
acceptance_preimage: ...
```

---

# 30. Customer vs Worker Responsibilities

## Customer

Responsible for:

```text
creating the work request
choosing the offered reward
funding from their own TRU wallet
reviewing the result
accepting/rejecting the result
protecting nmc_ and wallet-handoff capabilities
```

## Worker

Responsible for:

```text
protecting nmi_ / nmp_
operating their own compute hardware
explicitly accepting desired work
maintaining their public TRU claim key
running the approved local worker software
reviewing settlement data
claiming rewards from their own TRU wallet
```

## NEROMESH

Responsible for:

```text
marketplace coordination
job state
worker reservation
funding plan commitments
active-chain funding verification
job dispatch
result routing
settlement secret release rules
notifications
portal sessions
public network status
```

---

# 31. TRU Settlement Architecture

AXON uses TRU-native:

```text
HTLC / Atomic Swap V1
```

The current marketplace settlement is non-custodial.

Conceptually:

```text
CUSTOMER WALLET
     │
     ├── 20% COMPUTE HTLC ─────────────► worker claim key
     │        │
     │        └── customer timelocked refund path
     │
     └── 80% ACCEPTANCE HTLC ──────────► worker claim key
              │
              └── customer timelocked refund path
```

NEROMESH must verify:

```text
exact amount
canonical contract script
correct worker claim key
correct refund key
correct commitment hashes
active-chain funding
required confirmation/propagation state
```

before authorizing compute.

---

# 32. Why There Are Two HTLCs

The two-leg design separates payment for **doing the compute** from payment for **customer acceptance**.

```text
20% = valid compute/result completion

80% = customer accepts returned result
```

This gives the worker an earned compute component while preserving a much larger result-acceptance component.

---

# 33. Refund Protection

Each settlement leg includes a customer-controlled timelocked refund branch.

This means the customer does not simply send TRU into a NEROMESH custodial account.

The contract has two logical spend paths:

```text
worker claim path
OR
customer refund path after timeout
```

The refund private key remains in the customer's own wallet.

---

# 34. TRU Evolve / Token Evolution Work

AXON can also support TRU-native evolution jobs.

Evolution jobs may include:

```text
Token ID
Token Type
Target Authority
Evolution instructions
AI-generated preview/result
on-chain evolution verification
```

Important:

A worker-generated preview does not by itself prove that the token was successfully evolved on-chain.

For evolution settlement, the acceptance leg may require a matching confirmed TRU evolution record before the 80% secret is released.

Use normal general `text.generate` jobs when token evolution is not needed.

---

# 35. Approved AI Model

Current standard worker model:

```text
Qwen2.5-0.5B-Instruct
```

Current GGUF configuration:

```text
qwen2.5-0.5b-instruct-q4_k_m
```

The model executes locally on the participating worker hardware.

The worker bundle requires explicit model-license acceptance during installation.

---

# 36. CLI Command Reference

From:

```bash
cd ~/.local/share/neromesh-worker
```

## Worker status

```bash
python3 -m neromesh.worker status
```

## List open marketplace offers

```bash
python3 -m neromesh.worker marketplace list
```

## Show an OPEN marketplace offer

```bash
python3 -m neromesh.worker marketplace show AXN-XXXXXXXXXXXX
```

## Accept

```bash
python3 -m neromesh.worker marketplace accept AXN-XXXXXXXXXXXX
```

## Decline

```bash
python3 -m neromesh.worker marketplace decline AXN-XXXXXXXXXXXX
```

## Settlement / assigned-job status

```bash
python3 -m neromesh.worker marketplace settlement AXN-XXXXXXXXXXXX
```

## Save default public TRU claim key

```bash
python3 -m neromesh.worker marketplace set-claim-pubkey <COMPRESSED_PUBLIC_KEY>
```

## Start worker

```bash
~/.local/share/neromesh-worker/run-worker.sh
```

---

# 37. Typical CLI Worker Session

```bash
cd ~/.local/share/neromesh-worker

# Verify machine
python3 -m neromesh.worker status

# Browse
python3 -m neromesh.worker marketplace list

# Review
python3 -m neromesh.worker marketplace show AXN-XXXXXXXXXXXX

# Explicitly accept
python3 -m neromesh.worker marketplace accept AXN-XXXXXXXXXXXX

# Follow settlement
python3 -m neromesh.worker marketplace settlement AXN-XXXXXXXXXXXX
```

Then leave the worker runtime running.

After compute completes:

```bash
python3 -m neromesh.worker marketplace settlement AXN-XXXXXXXXXXXX
```

At:

```text
WAITING_ACCEPTANCE
```

claim the 20% compute HTLC.

After the customer accepts, run settlement again.

At:

```text
ACCEPTED
```

claim the 80% acceptance HTLC.

---

# 38. Typical GUI Customer Session

```text
/customer
   ↓
Sign in with nmc_
   ↓
Marketplace → Create Work
   ↓
Wait for worker
   ↓
Notification: Worker Found
   ↓
Needs Action → Fund
   ↓
Create/copy wallet handoff
   ↓
Fund from TRU Core/Desktop
   ↓
Wait for result
   ↓
Notification: Result Ready
   ↓
Review
   ↓
Swipe Accept
   ↓
Completed
```

---

# 39. Typical GUI Worker Session

```text
/worker
   ↓
Sign in with nmp_
   ↓
Verify worker ONLINE
   ↓
Open Offers
   ↓
Review Job
   ↓
Swipe Accept
   ↓
Wait for funding
   ↓
Worker executes automatically
   ↓
20% Reward Ready
   ↓
Claim in TRU Core
   ↓
Customer accepts
   ↓
80% Reward Ready
   ↓
Claim in TRU Core
```

---

# 40. Portal Notifications

UX-04 notifications are intended to prevent users from repeatedly searching for state changes.

Examples:

## Customer

```text
Worker found
Funding required
Funding confirmed
Result ready
Acceptance required
Completed
Refund available
```

## Worker

```text
New matching offer
Reserved
Funding confirmed
Compute reward unlocked
Acceptance reward unlocked
Offline/stale warning
```

Notifications support read/unread state.

---

# 41. Public Network States

The Network page may display:

```text
ONLINE
READY
COMPUTING
PAUSED
STALE
OFFLINE
```

These states describe availability, not authority.

An online worker does **not** automatically agree to run every job.

Explicit marketplace acceptance remains mandatory.

---

# 42. Health Check

Public health:

```bash
curl -sS https://tru.neromesh.space/api/health \
  | python3 -m json.tool
```

Expected current release:

```text
release: 0.5.0-axon-portals
phase: public marketplace portals
```

Important features should include:

```text
axon_marketplace: true
noncustodial_htlc_settlement: true
automatic_funding_reconciliation: true
wallet_handoff_intents: true
portal_sessions: true
customer_dashboard: true
worker_dashboard: true
network_dashboard: true
portal_notifications: true
wallet_pairing: false
```

---

# 43. What NEROMESH Does Not Do

Current NEROMESH does not require:

```text
customer wallet private keys
worker wallet private keys
customer-to-worker direct networking
public worker IP disclosure
persistent wallet pairing
automatic worker claim signing
```

The coordinator coordinates.

The wallets sign.

The worker computes.

TRU settles.

---

# 44. Safe Support Information

If asking for support, it is generally okay to provide:

```text
AXN Job ID
AXI intent ID without the capability
public worker ID
public worker claim key
public funding TXIDs/vouts
public claim TXID
result SHA-256
job state
redacted error messages
```

Do **not** provide:

```text
nmc_
nmi_
nmp_
axc_
private keys
seed phrases
wallet passwords
worker.json
full unexpired tru:// handoff URI
```

---

# 45. Recommended Recovery Rule

If something involving money appears inconsistent:

> **Do not fund again until the existing funding state is inspected.**

Check:

```text
Customer dashboard
Worker settlement
TRU transaction state
wallet-handoff journal
```

The system intentionally favors refusing ambiguous duplicate funding over silently creating another set of contracts.

---

# 46. Current Design Principles

NEROMESH / AXON is built around several core principles:

```text
Explicit worker choice
Non-custodial settlement
Local wallet signing
Customer refund protection
Two-stage worker compensation
Chain-verified funding
Role-separated credentials
Recoverable/idempotent funding
Inspectable settlement
GUI and CLI parity
```

The GUI is intended to make the protocol easier to use, not hide what the protocol is doing.

---

# 47. Quick Reference

## Customer

```text
Website:
https://tru.neromesh.space/customer

Credential:
nmc_...

Create:
https://tru.neromesh.space/marketplace

Flow:
Create → Worker → Fund → Compute → Review → Accept
```

## Worker

```text
Website:
https://tru.neromesh.space/worker

Credential:
nmp_...

Install credential:
nmi_...

CLI:
cd ~/.local/share/neromesh-worker
python3 -m neromesh.worker marketplace list
python3 -m neromesh.worker marketplace show AXN-...
python3 -m neromesh.worker marketplace accept AXN-...
python3 -m neromesh.worker marketplace settlement AXN-...
```

## Network

```text
https://tru.neromesh.space/network
```

## Health

```text
https://tru.neromesh.space/api/health
```

---

# 48. End-to-End Summary

```text
CUSTOMER                              WORKER

Create AXON job
     │
     ├────────── OPEN OFFER ────────────►
     │                                  Review
     │                                  Accept
     │◄────────── RESERVED ─────────────┤
     │
Fund through local TRU wallet
     │
     ├── 20% HTLC
     └── 80% HTLC
     │
NEROMESH verifies chain
     │
     ├────────── FUNDED ───────────────►
     │                                  AI runs locally
     │                                  Result returned
     │
     │◄──────── RESULT READY ───────────
     │                                  20% secret unlocked
Review result                          Claim 20%
     │
Accept
     │
     ├────────── ACCEPTED ─────────────►
                                        80% secret unlocked
                                        Claim 80%
```

---

# NEROMESH

**The Axon Compute Network**

**Independent compute. Coordinated intelligence.**

```text
https://tru.neromesh.space
```
