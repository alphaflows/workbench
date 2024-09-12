# Design of Refund Request Logic

## Refund request logic

- start in console interface
- user has options to claim refund for some particular job(cp)
- user fill information in a form (must include job_uuid, cp_account_address)
- the form send information into copper's refund pipeline
- BD team process the request
- if request is accepted, put it to `Refund Accepted`
- if request is rejected, put it to `Rejected`

```mermaid
graph TD
    A[Start in console interface] --> B[User options to claim refund for job]
    B --> C[User fills form with job_uuid and cp_account_address]
    C --> D[Form sends information to copper's refund pipeline]
    D --> E[BD team processes the request]
    E --> F{Request accepted?}
    F -->|Yes| G[Put to 'Refund Accepted']
    F -->|No| H[Put to 'Rejected']
```

## CRM process refund flow

- CRM backend scan copper API to get `Refund Accepted` refund requests
- For each scanned refund request, change stage to `Refund in Process`
- for each refund request, CRM call Orchestrator API to process refund
- Get signature from Orchestrator API
- Send signature and change refund request stage to `Refund Complete` to copper pipeline


```mermaid
graph TD
    O[Start] --> A[CRM backend scans copper API] 
    A --> B[Get 'Refund Accepted' requests]
    B --> C[For each refund request]
    C --> G[change stage to 'Refund in Process' via copper API]
    G --> D[Call Orchestrator API to process refund]
    D --> H{Refund processed?}
    H --> |Yes| E[Get signature from Orchestrator API]
    H --> |No| S[Get error message from Orchestrator API]
    E --> F[Send signature and change stage to 'Refund Complete' in copper pipeline]
    S --> I[Send error message and change stage to 'Refund Reject' in copper pipeline]
    F --> Q[End]
    I --> Q
```

## Orchestrator process refund logic

- Orchestrator receives refund request from CRM, parameters: `cp_account_address` and `job_uuid`, `task_uuid`
- Orchestrator generate signature for refund
- Slash CP by it's collateral * 5 (5 times penalty) `slashCollateral(task_uuid, cp_account_address, 5*base_collateral)`
- Save refund request for CP in `refund_record` table (`job_uuid`, `cp_account_address`, slashed_amount, task_uuid, timestamp, etc)
- if task has been completed, **claim back CP's reward** (call `reclaimReward(task_uuid, cp_account_address)`)
- if task has not been completed yet, when complete task, don't release reward for this CP; when complete task later, call `completeTask(task_uuid, cp_address_list)` to exclude this CP


```mermaid
graph TD
    O[Start] --> A[Orchestrator receives refund request from CRM] 
    A --> M{Job eligible for refund?}
    M --> |Yes| B[Generate signature for refund]
    M --> |No| N[Return refund failed message]
    B --> C[Slash CP by collateral * 5]
    C --> E{Task completed?}
    E -->|Yes| F[Claim back CP's reward]
    E -->|No| G[This CP will be excluded from complete task logic]
    F --> D[Save refund record]
    G --> D
    D --> P[Return refund signature and success info]
    P --> Q[End]
    N --> Q
```

### Failed refund cases

- `{job_uuid=} refund record already exists`
- `Task not found, no action taken`
- `Task is terminated, not eligible for refund`
- `Job not found, no action taken`
- `Job not ended, not eligible for refund`
- `Job cp_account_address not match, no action taken`
- `CP not found, no action taken`
- `Wallet not match, no action taken`
- `no cp in task, no action taken` (get info from contract, very rare)
- `Failed to process refund for user...` (slash failed contract, will retry)


### `refund_request` Table (in CRM)

```sql
CREATE TABLE `refund_request` (
    `id` int NOT NULL AUTO_INCREMENT,
    `copper_id` int NOT NULL,
    `user_email` varchar(255) DEFAULT NULL COMMENT 'Email address of the user',
    `user_wallet` varchar(255) NOT NULL COMMENT 'Wallet address of the user',
    `task_uuid` varchar(64) NOT NULL COMMENT 'UUID of task associated with the refund request',
    `job_uuid` varchar(64) NOT NULL COMMENT 'UUID of job associated with the refund request',
    `cp_account_address` varchar(64) NOT NULL COMMENT 'cp account address associated with the refund request',
    `stage` varchar(64) DEFAULT NULL,
    `refund_signature` varchar(255) DEFAULT NULL COMMENT 'Signature for refund (if approved)',
    `created_at` int DEFAULT NULL,
    `updated_at` int DEFAULT NULL,
    `processed_at` int DEFAULT NULL,
    `completed_at` int DEFAULT NULL,
    `deleted_at` int DEFAULT NULL,
    `comments` varchar(255) DEFAULT NULL,
    PRIMARY KEY (`id`)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 COLLATE = utf8mb4_0900_ai_ci;
```


### `refund_record` Table (in Orchestrator)


```sql
CREATE TABLE `refund_record` (
    `id` int NOT NULL AUTO_INCREMENT,
    `task_uuid` varchar(64) NOT NULL,
    `job_uuid` varchar(64) NOT NULL,
    `user_wallet` varchar(255) NOT NULL,
    `cp_account_address` varchar(255) NOT NULL,
    `owner_address` varchar(255) NOT NULL,
    `refund_signature` varchar(255) DEFAULT NULL,
    `task_status` varchar(45) DEFAULT NULL,
    `job_status` varchar(45) DEFAULT NULL,
    `slash_amount` double DEFAULT NULL,
    `reclaim_reward` double DEFAULT NULL,
    `tx_hash` varchar(255) NOT NULL COMMENT 'txhash for slash',
    `status` varchar(45) DEFAULT NULL COMMENT 'status for refund process itself',
    `created_at` int DEFAULT NULL,
    `updated_at` int DEFAULT NULL,
    `comments` varchar(255) DEFAULT NULL,
    PRIMARY KEY (`id`)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 COLLATE = utf8mb4_0900_ai_ci;
```