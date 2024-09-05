# Design of Refund Request Logic

## Refund request logic

- start in console interface
- user has options to claim refund for some particular job(cp)
- user fill information in a form (must include job_uuid, cp_account_address)
- the form send information into copper's refund pipeline
- BD team process the request
- if request is accepted, put it to `Refund in Process`
- if request is rejected, put it to `Rejected`

```mermaid
graph TD
    A[Start in console interface] --> B[User options to claim refund for job]
    B --> C[User fills form with job_uuid and cp_account_address]
    C --> D[Form sends information to copper's refund pipeline]
    D --> E[BD team processes the request]
    E --> F{Request accepted?}
    F -->|Yes| G[Put to 'Refund in Process']
    F -->|No| H[Put to 'Rejected']
```

## CRM process refund flow

- CRM backend scan copper API to get `Refund in Process` refund requests
- for each refund request, CRM call Orchestrator API to process refund
- Get signature from Orchestrator API
- Send signature and put refund to `Resolved` to copper pipeline


```mermaid
graph TD
    A[CRM backend scans copper API] --> B[Get 'Refund in Process' requests]
    B --> C[For each refund request]
    C --> D[Call Orchestrator API to process refund]
    D --> E[Get signature from Orchestrator API]
    E --> F[Send signature and put refund to 'Resolved' in copper pipeline]
```

## Orchestrator process refund logic

- Orchestrator receives refund request from CRM, parameters: `cp_account_address` and `job_uuid`
- Orchestrator generate signature for refund
- Slash CP by it's collateral * 5 (5 times penalty)
- Save refund request for CP in `refund_process` table (`job_uuid`, `cp_account_address`, slashed_amount, task_uuid, timestamp, etc)
- if task has been completed, **claim back CP's reward** (call `reclaimReward(task_uuid, cp_account_address)`)
- if task has not been completed yet, when complete task, don't release reward for this CP; when complete task later, call `completeTask(task_uuid, cp_address_list)` to exclude this CP


```mermaid
graph TD
    A[Orchestrator receives refund request from CRM] --> B[Generate signature for refund]
    B --> C[Slash CP by collateral * 5]
    C --> D[Save refund request in 'refund_process' table]
    D --> E{Task completed?}
    E -->|Yes| F[Claim back CP's reward]
    E -->|No| G[When task completes later]
    G --> H[Don't release reward for this CP; Call completeTask to exclude this CP]
```

