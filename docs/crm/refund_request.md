Creating a flowchart to represent the refund request process involves illustrating the sequence of steps and interactions with APIs. Below is a textual description of how the flowchart would be structured:

Part 1: User Flow

```mermaid
flowchart TD
    A[Start] --> B[User initiates refund request]
    B --> C[Upload screenshots and add description]
    C --> D[API: request_refund]
    D --> E[Upload pics to MCS]
    E --> F[Store pic URLs in DB]
    F --> G[Set request status: 'created']
```

Part 2: BD Team Flow

```mermaid
flowchart TD
    H[Start: BD Team] --> I[BD gets list of requests]
    I --> J[API: get list of refund requests]
    J --> K[Display list with pagination, filters, and sorting]
    K --> L{Process request?}
    L -->|Yes| M[Show accept/decline buttons]
    M --> N{Accept or Decline?}
    N -->|Accept| O[API: process request]
    O --> P[API: orchestrator create signature and call contract API]
    P --> Q[Update status to 'accepted']
    N -->|Decline| R[Update status to 'declined']
    Q --> S[End]
    R --> S
    L -->|No| S
```

1. **Step 1: User Request Refund**  
   - **Input**: User uploads screenshots (max 3, 5MB limit) and adds comments/description.  
   - **Process**:  
     - Call **API: request_refund**  
       - Upload pictures to MCS.  
       - Store picture URLs in the database.  
       - Save the description.  
       - Set request status to `created`.

2. **Step 2: BD Get a List of Requests**  
   - **Process**:  
     - Call **API: get list of refund requests**  
       - Implement pagination.  
       - Filter by status/date.  
       - Sort by date.  
   - **Output**: Display list of requests with `created` status for further processing.

3. **Step 3: Process a Request**  
   - **Decision**: Show buttons to either accept or decline the request.  
   - Call **API: process request**  
   - **If Accepted**:  
     - Call **API: orchestrator create signature and call contract API to refund**  
     - Update refund request status to `accepted` in CRM.  
   - **If Declined**:  
     - Update refund request status to `declined` in CRM.
