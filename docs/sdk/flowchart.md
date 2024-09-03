
# Steps using SDK

## Main Logic

To deploy task:

- user get API-Key
- user use API-Key to login in Orchestrator using SDK
- get a list of instance resources
- choose an instance type
- use `create_task` to deploy task
- use `get_deployment_info` to follow up task's status
- use `get_real_url` to get APP URLs and to check if APP is running

To renew task:

- use `renew_task` to extend task's duration
- use `get_deployment_info` to follow up task's status
- to check `duration` is updated

To terminate task:

- use `terminate_task` to early terminate a task
- use `get_deployment_info` to follow up task's status
- to check task status is updated to `terminated`


### 1. Deploying a Task

```mermaid
flowchart TD
    A[Start] --> B[User gets API-Key]
    B --> C[User logs in to Orchestrator using SDK with API-Key]
    C --> D[Get list of instance resources]
    D --> E[Choose an instance type]
    E --> F[Use create_task to deploy task]
    F --> G[Use get_deployment_info to follow up task's status]
    G --> H[Use get_real_url to get APP URLs and check if APP is running]
    H --> I[End]
```

### 2. Renewing a Task

```mermaid
flowchart TD
    A[Start] --> B[Use renew_task to extend task's duration]
    B --> C[Use get_deployment_info to follow up task's status]
    C --> D[Check task's duration updated]
    E --> G[End]
```

### 3. Terminating a Task

```mermaid
flowchart TD
    A[Start] --> B[Use terminate_task to early terminate a task]
    B --> C[Use get_deployment_info to follow up task's status]
    C --> D[Check task status is 'terminated']
    D --> G[End]
```
