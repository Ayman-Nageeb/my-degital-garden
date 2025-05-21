---
{"dg-publish":true,"permalink":"/system-analysis/simulation/simulation/","tags":["gardenEntry"]}
---

The simulation system is designed to:
1. Test game mechanics
2. Verify RTP (Return to Player)
3. Analyze volatility
4. Ensuring game fairness and reliability
5. Verifying mathematical models
6. Validate bonus features
	- Game volatility
	- Win distribution
	- Maximum win potential
	- Risk levels
7. Generate comprehensive reports
8. Collect detailed metrics
9. Discover Win patterns
10. Performance Testing
11. Balance Verification
	- Game balance
	- Win limits
	- Payout ratios
	- Feature frequencies


- [ ] Service Structure
- [ ] Configuration
- [ ] Simulation Flow
- [ ] Spin Generation
- [ ] Result Collection
- [ ] Simulation Process
- [ ] Metrics Collection
- [ ] Report Generation
- [ ] Parallel Processing
- [ ] Progress Tracking
- [ ] Statistics Calculation
- [ ] Output Reports





```mermaid
graph TD
    A[Start Simulation] --> B[Initialize Config]
    B --> C[Create Worker Pool]
    C --> D[Generate Spins]
    D --> E[Process Results]
    E --> F[Calculate Statistics]
    F --> G[Generate Report]
    G --> H[Save Report]
```


```mermaid
sequenceDiagram
    participant S as Simulator
    participant W as Workers
    participant F as Spin Factory
    participant M as Metrics

    S->>W: Initialize Workers
    W->>F: Generate Spins
    F-->>W: Return Results
    W->>M: Collect Metrics
    M-->>S: Aggregate Results
    S->>S: Generate Report
```

```mermaid
sequenceDiagram
    participant Client
    participant Handler as Game Flow Handler
    participant Facade
    participant GameFlow as Game Flow Service
    participant History as History Service
    participant Overlord
    participant DB as History DB
    participant IP2Country

    Client->>Handler: POST /core/spin_indexes/update
    Note over Client,Handler: Payload: {session_token, restoring_indexes, record_id}

    %% Request Validation
    Handler->>Handler: Validate Request
    Handler->>Handler: Get Metadata
    Handler->>Facade: UpdateSpinIndexes()

    %% Facade Processing
    Facade->>Facade: Validate Metadata
    Facade->>Facade: Parse Request
    Facade->>Facade: Check HistoryHandlingType

    alt ParallelRestoring
        Facade->>History: UpdateSpinIndexes()
        History->>DB: GetByID()
        DB-->>History: Spin Record
        History->>History: Update RestoringIndexes
        History->>DB: Update Record
        DB-->>History: Updated Record
        History-->>Facade: History Record
        Facade-->>Handler: null balance
    else SequentialRestoring
        Facade->>GameFlow: UpdateRestoringIndexes()
        GameFlow->>Overlord: GetStateBySessionToken()
        Overlord-->>GameFlow: Game State
        GameFlow->>History: lastRecord()
        History->>DB: Get Last Record
        DB-->>History: Spin Record
        History-->>GameFlow: History Record

        alt Record Can Be Updated
            GameFlow->>GameFlow: Update RestoringIndexes
            alt Need Bet Closing
                GameFlow->>Overlord: CloseBet()
                Overlord-->>GameFlow: Updated Balance
            end
            GameFlow->>History: update()
            History->>DB: Update Record
            DB-->>History: Updated Record
            History-->>GameFlow: Updated Record
            GameFlow-->>Facade: Updated Balance
            Facade-->>Handler: Updated Balance
        else Record Cannot Be Updated
            GameFlow-->>Facade: Error
            Facade-->>Handler: Error
        end
    end

    %% Response Handling
    alt Success
        Handler-->>Client: 200 OK
        Note over Handler,Client: {balance: number|null}
    else Error
        Handler-->>Client: Error Response
        Note over Handler,Client: 400/401/422/409/500
    end

    %% Additional Notes
    Note over DB: MongoDB Indexes:
    Note over DB: - created_at
    Note over DB: - session_token
    Note over DB: - internal_user_id
    Note over DB: - game
    Note over DB: - currency

    Note over IP2Country: Optional:
    Note over IP2Country: Adds country info
    Note over IP2Country: to spin records
```
