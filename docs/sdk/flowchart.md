
Steps using SDK:

- user get API-Key
- user use API-Key to login in Orchestrator using SDK
- get a list of instance resources
- choose an instance type
- use `create_task` to deploy task (auto_pay=True) or to only initialize task (auto_pay=False)
- use `get_deployment_info` to follow up task's status
- use `get_real_url` to get APP URLs and to check if APP is running
- use `renew_task` to extend task's duration
- still use `get_deployment_info` to follow up task's status
- or use `terminate_task` to early terminate a task
- still use `get_deployment_info` to follow up task's status
- if task is only initialized, use `make_payment` to do payment and deploy task



```mermaid
flowchart TD
    A[Start] --> B[User gets API-Key]
    B --> C[User logs in to Orchestrator using SDK with API-Key]
    C --> D[Get list of instance resources]
    D --> E[Choose an instance type]
    E --> F{Auto-pay?}
    F -->|Yes| G[Use create_task to deploy task]
    F -->|No| H[Use create_task to initialize task]
    G --> I[Use get_deployment_info to follow up task's status]
    H --> I
    I --> J[Use get_real_url to get APP URLs and check if APP is running]
    J --> K{Extend duration?}
    K -->|Yes| L[Use renew_task to extend task's duration]
    L --> M[Use get_deployment_info to follow up task's status]
    K -->|No| N{Terminate early?}
    N -->|Yes| O[Use terminate_task to end task]
    O --> P[Use get_deployment_info to follow up task's status]
    N -->|No| Q{Task only initialized?}
    Q -->|Yes| R[Use make_payment to do payment and deploy task]
    R --> S[Use get_deployment_info to follow up task's status]
    Q -->|No| T[End]
    M --> N
    P --> T
    S --> T
```
