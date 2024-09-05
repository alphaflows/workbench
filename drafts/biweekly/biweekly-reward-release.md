# Biweekly CP Reward Release Design

Currently when a task duration expires, after one-day cooling down, the task will be completed and the reward will be released to CPs immediately, which costs lots of gas. 

## Biweekly Reward Release Logic

A gas saving strategy: biweekly releasing reward for CP and let CP claim reward by himself (cost their gas).

Business Logic:

- When a task is completed on contract, the rewards will be only kept in CP's reward account
- Biweekly on Friday, the two-week rewards will be calculated (`get_claimable_reward`) and the swanhub backend will generate a signature for this pay period for CP to claim its reward amount
- Frontend will show the CP's owner (by owner address) list of claimable rewards and let owner to claim reward (using signature):
  - [Owner Reward Overview](https://github.com/swanchain/Private-Doc/blob/main/Swan/Orchestrator/Document/API/API_cp_reward_claim.md#get-owner-reward-overview)
  - [CP reward list in the pay period](https://github.com/swanchain/Private-Doc/blob/main/Swan/Orchestrator/Document/API/API_cp_reward_claim.md#get-cp-list-in-the-reward-period)
  - [CP's job list](https://github.com/swanchain/Private-Doc/blob/main/Swan/Orchestrator/Document/API/API_cp_reward_claim.md#get-jobs-in-reward-period)

