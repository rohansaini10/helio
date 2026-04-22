# Helio Project - Comprehensive Diagrams

This document contains diagram codes for all architecture, flow, and system diagrams for the Helio Solana Wallet application. Use these with text-to-diagram tools like Mermaid.

---

## 1. Complete Project Diagram (System Overview)

```mermaid
graph TB
    subgraph Client ["Mobile Client (React Native/Expo)"]
        UI["UI Layer<br/>Screens & Components"]
        Context["State Management<br/>Network & Wallet Context"]
        Hooks["Custom Hooks<br/>useWalletScreen<br/>useSwapScreen"]
    end

    subgraph Features ["Feature Modules"]
        Wallet["Wallet Feature<br/>- View Balance<br/>- Token Listing<br/>- Send SOL"]
        Swap["Swap Feature<br/>- Get Quote<br/>- Execute Swap<br/>- Track History"]
        Token["Token Details<br/>- Metadata<br/>- Market Stats<br/>- Holdings"]
        Watch["Watchlist<br/>- Add Tokens<br/>- Monitor Prices"]
    end

    subgraph Libs ["Library Layer"]
        SolanaLib["Solana Library<br/>- RPC Calls<br/>- Balance Fetch<br/>- Token Metadata"]
        SwapLib["Swap Library<br/>- Quote Fetch<br/>- Transaction Build"]
        Storage["Storage<br/>MMKV Cache<br/>Local State"]
    end

    subgraph External["External Services"]
        JupAPI["Jupiter API<br/>- Swap Quotes<br/>- Token Metadata"]
        HeliusAPI["Helius RPC<br/>- Token Info<br/>- Asset Metadata"]
        SolanaRPC["Solana RPC<br/>- Blockchain Data<br/>- Transactions"]
        MWA["Mobile Wallet Adapter<br/>- Sign Transactions<br/>- Get Pubkey"]
    end

    UI --> Context
    UI --> Hooks
    Hooks --> Features
    Features --> Libs
    SolanaLib --> SolanaRPC
    SolanaLib --> HeliusAPI
    SwapLib --> JupAPI
    Context --> Storage
    Libs --> External
    External --> MWA

    style Client fill:#e1f5ff
    style Features fill:#f3e5f5
    style Libs fill:#fff3e0
    style External fill:#e8f5e9
```

---

## 2. ER Diagram (Entity Relationship)

```mermaid
erDiagram
    USER ||--o{ TOKEN : holds
    USER ||--o{ TRANSACTION : initiates
    USER ||--o{ SWAP : executes
    TOKEN ||--o{ TOKEN-METADATA : has
    TRANSACTION ||--o{ TOKEN-TRANSFER : contains
    SWAP ||--o{ SWAP-HISTORY : logs
    WATCHLIST ||--o{ TOKEN : monitors

    USER {
        string publicKey PK
        string base64Address
        string authToken
        bigint balance
        string network
    }

    TOKEN {
        string mint PK
        string symbol
        string name
        string logoURI
        int decimals
        bigint totalSupply
    }

    TOKEN-METADATA {
        string mint PK
        string tokenName
        string symbol
        string logoURI
        int decimals
        string tokenProgram
        bigint totalSupply
    }

    TRANSACTION {
        string signature PK
        string publicKey FK
        string blockTime
        string status
        string error
        timestamp createdAt
    }

    TOKEN-TRANSFER {
        string signature FK
        string mint
        string source
        string destination
        bigint amount
    }

    SWAP {
        string id PK
        string signature
        string publicKey FK
        string inputMint
        string outputMint
        bigint inputAmount
        bigint outputAmount
        int slippageBps
        timestamp createdAt
    }

    SWAP-HISTORY {
        string id PK
        string signature FK
        string inputSymbol
        string outputSymbol
        bigint inputAmount
        bigint outputAmount
        int slippageBps
        string status
        timestamp completedAt
    }

    WATCHLIST {
        string id PK
        string publicKey FK
        string mint FK
        timestamp addedAt
    }
```

---

## 3. UML Class Diagram

```mermaid
classDiagram
    class UserWalletContext {
        -publicKey: Address
        -authToken: string
        -connected: boolean
        -connecting: boolean
        -sending: boolean
        -signing: boolean
        +connect(): Promise
        +disconnect(): Promise
        +getBalance(): Promise
        +sendSOL(toAddress, amountSOL): Promise
        +signAndSendTransaction(bytes): Promise
    }

    class NetworkContext {
        -network: Network
        -rpc: SolanaRpc
        -toggleNetwork(): void
        -getNetwork(): Network
    }

    class WalletScreen {
        -publicKey: Address
        -connected: boolean
        -tokens: Token[]
        -transactions: Transaction[]
        -loading: boolean
        +handleSearch(address): void
        +handleRefresh(): void
        +sendSOL(): void
    }

    class SwapScreen {
        -fromToken: Token
        -toToken: Token
        -payAmount: string
        -slippage: number
        -quoteLoading: boolean
        +handleAmountChange(amount): void
        +handleFlip(): void
        +handleSelectFrom(token): void
        +handleSelectTo(token): void
        +handleSlippage(value): void
        +executeSwap(): Promise
    }

    class Token {
        -mint: string
        -symbol: string
        -name: string
        -logoURI: string
        -decimals: number
        -amount: number
    }

    class Transaction {
        -signature: string
        -blockTime: string
        -status: string
        -error: string
        +getDetail(): Promise
    }

    class SwapQuote {
        -inputMint: string
        -outputMint: string
        -inAmount: bigint
        -outAmount: bigint
        -priceImpact: number
        -route: string[]
    }

    class SolanaService {
        +getBalance(address): Promise
        +getAllTokens(address): Promise
        +getAllTransactions(address): Promise
        +getTransactionDetail(signature): Promise
        +getAllTokenMetadata(mints): Promise
    }

    class SwapService {
        +fetchSwapQuote(params): Promise
        +getSwapTransaction(params): Promise
        +executeSwap(userKey, quote): Promise
    }

    class Storage {
        +getItem(key): string
        +setItem(key, value): void
        +removeItem(key): void
        -mmkv: MMKV
    }

    UserWalletContext --> Storage
    NetworkContext --> SolanaService
    WalletScreen --> UserWalletContext
    WalletScreen --> SolanaService
    WalletScreen --> Token
    WalletScreen --> Transaction
    SwapScreen --> UserWalletContext
    SwapScreen --> SwapService
    SwapScreen --> Token
    SwapScreen --> SwapQuote
    SwapService --> SolanaService
    SolanaService --> Token
    SolanaService --> Transaction
```

---

## 4. UML Sequence Diagram - User Wallet Connection

```mermaid
sequenceDiagram
    participant User
    participant App as Helio App
    participant MWA as Mobile Wallet<br/>Adapter
    participant SolanaRPC as Solana RPC
    participant Storage as MMKV Storage

    User->>App: Tap Connect Wallet
    App->>MWA: Call transact()
    MWA->>MWA: Launch wallet app
    User->>MWA: Approve connection
    MWA->>SolanaRPC: Verify chain
    SolanaRPC-->>MWA: Chain confirmed
    MWA-->>App: Return authToken + address
    App->>Storage: Cache token & base64Address
    App-->>User: Wallet connected
    Note over App,Storage: User session persisted
```

---

## 5. UML Sequence Diagram - Token Balance Fetch

```mermaid
sequenceDiagram
    participant App as Helio App
    participant SolanaLib as Solana Library
    participant SolanaRPC as Solana RPC
    participant JupiterAPI as Jupiter API
    participant HeliusAPI as Helius RPC

    App->>SolanaLib: getBalance(publicKey)
    SolanaLib->>SolanaRPC: getBalance(pubKey)
    SolanaRPC-->>SolanaLib: balance (lamports)
    SolanaLib->>SolanaLib: Convert lamports to SOL
    SolanaLib-->>App: balance (SOL)

    App->>SolanaLib: getAllTokens(publicKey)
    SolanaLib->>SolanaRPC: getTokenAccountsByOwner()
    SolanaRPC-->>SolanaLib: token accounts
    SolanaLib->>SolanaLib: Filter by amount > 0
    SolanaLib-->>App: tokens array

    App->>SolanaLib: getAllTokenMetadata(mints)
    SolanaLib->>JupiterAPI: /tokens/v2/search?query=mints
    JupiterAPI-->>SolanaLib: token metadata
    Note over SolanaLib: If Jupiter fails, try Helius
    SolanaLib->>HeliusAPI: getAssetBatch(mints)
    HeliusAPI-->>SolanaLib: asset metadata
    SolanaLib-->>App: metadata map

    App-->>User: Display wallet<br/>with balances
```

---

## 6. UML Sequence Diagram - Swap Execution

```mermaid
sequenceDiagram
    participant User
    participant App as Helio App
    participant SwapLib as Swap Library
    participant JupAPI as Jupiter API
    participant MWA as Mobile Wallet<br/>Adapter
    participant SolanaRPC as Solana RPC

    User->>App: Enter swap amount
    App->>SwapLib: fetchSwapQuote(inputMint, outputMint, amount, slippage)
    SwapLib->>JupAPI: GET /swap/v1/quote?params
    JupAPI-->>SwapLib: SwapQuote (inAmount, outAmount, route)
    SwapLib-->>App: Display quote & route
    
    User->>App: Confirm swap
    App->>SwapLib: getSwapTransaction(quoteResponse, userPublicKey)
    SwapLib->>JupAPI: POST /swap/v1/swap
    JupAPI-->>SwapLib: swapTransaction (base64)
    SwapLib->>SwapLib: Decode transaction
    SwapLib-->>App: Unsigned transaction
    
    App->>MWA: signTransactions([transaction])
    MWA->>User: Show approval prompt
    User->>MWA: Approve transaction
    MWA-->>App: Signed transaction
    
    App->>SolanaRPC: sendTransaction(signedTx)
    SolanaRPC-->>App: Signature
    
    App->>App: Poll getSignatureStatuses()
    SolanaRPC-->>App: Status = finalized
    App-->>User: Swap complete!
```

---

## 7. UML Sequence Diagram - Send SOL

```mermaid
sequenceDiagram
    participant User
    participant App as Helio App
    participant SolanaLib as Solana Library
    participant SolanaRPC as Solana RPC
    participant MWA as Mobile Wallet<br/>Adapter

    User->>App: Enter recipient & amount
    App->>SolanaLib: sendSOL(toAddress, amountSOL)
    
    SolanaLib->>SolanaRPC: getLatestBlockhash()
    SolanaRPC-->>SolanaLib: blockhash + lastValidBlockHeight
    
    SolanaLib->>SolanaLib: Build transaction with<br/>SystemProgram.transfer instruction
    SolanaLib-->>App: Transaction ready
    
    App->>MWA: signTransactions([transaction])
    User->>MWA: Approve in wallet
    MWA-->>App: Signed transaction
    
    App->>SolanaRPC: sendTransaction(signedTx)
    SolanaRPC-->>App: Transaction signature
    
    loop Poll until confirmed
        App->>SolanaRPC: getSignatureStatuses([signature])
        SolanaRPC-->>App: Status check
    end
    
    SolanaRPC-->>App: Status = confirmed/finalized
    App-->>User: Transfer complete!
```

---

## 8. UML Activity Diagram - Token Balance Fetch Activity

```mermaid
activity
    start
    :Enter wallet address;
    :Validate public key;
    if (valid?) then (no)
        :Show error;
        stop
    else (yes)
        :Fetch SOL balance from RPC;
        :Fetch token accounts from RPC;
        :Extract token mints;
        :Batch fetch token metadata from Jupiter;
        if (Jupiter succeeds?) then (no)
            :Try Helius API;
        else (yes)
        endif
        :Map metadata to tokens;
        :Sort by balance;
        :Cache results in MMKV;
        :Update UI with tokens;
    endif
    stop
```

---

## 9. UML Activity Diagram - Swap Execution Flow

```mermaid
activity
    start
    :User enters swap amounts;
    :Select input & output tokens;
    :Set slippage tolerance;
    :Request quote from Jupiter;
    if (Quote available?) then (yes)
        :Display swap preview;
        :Show price impact;
        if (User confirms?) then (yes)
            :Authorize wallet session;
            :Fetch latest blockhash;
            :Request swap transaction from Jupiter;
            if (Transaction ready?) then (yes)
                :Show approval in wallet;
                if (User approves?) then (yes)
                    :Sign transaction via MWA;
                    :Send transaction to RPC;
                    :Poll for confirmation;
                    if (Confirmed?) then (yes)
                        :Store in swap history;
                        :Show success;
                    else (no)
                        :Show timeout error;
                    endif
                else (no)
                    :Show rejection;
                endif
            else (no)
                :Show build error;
            endif
        else (no)
            :Dismiss;
        endif
    else (no)
        :Show quote error;
        :Retry option;
    endif
    stop
```

---

## 10. Data Flow Diagram - Level 0 (System Context)

```mermaid
graph TB
    User["👤 User"]
    Phone["📱 Mobile Device<br/>(React Native App)"]
    Wallet["💰 Wallet App<br/>(Phantom, Solflare)"]
    Solana["⛓️ Solana Blockchain"]
    API["🔗 External APIs<br/>(Jupiter, Helius, RPC)"]

    User -->|Interact| Phone
    Phone -->|Sign Transactions| Wallet
    Phone -->|Query Blockchain| Solana
    Phone -->|Fetch Data| API
    Wallet -->|Approve/Sign| Solana
    Solana -->|Return Data| Phone
    API -->|Return Data| Phone
    
    style Phone fill:#e3f2fd
    style User fill:#fff3e0
    style Wallet fill:#f3e5f5
    style Solana fill:#e8f5e9
    style API fill:#fce4ec
```

---

## 11. Data Flow Diagram - Level 1 (Main System Process)

```mermaid
graph TB
    subgraph Input ["Input Processes"]
        I1["User Input<br/>Address/Amount"]
        I2["Wallet<br/>Connection"]
    end

    subgraph Processing ["Processing Layer"]
        P1["Validate Input"]
        P2["Fetch Blockchain<br/>Data"]
        P3["Calculate Swap<br/>Quote"]
        P4["Build Transaction"]
    end

    subgraph Storage ["Storage"]
        S1["MMKV Cache<br/>Tokens"]
        S2["MMKV Cache<br/>History"]
    end

    subgraph Output ["Output"]
        O1["Display Data"]
        O2["Show Status"]
    end

    I1 --> P1
    I2 --> P2
    P1 --> P2
    P2 --> S1
    P2 --> O1
    P3 --> S2
    P4 --> O2

    style Input fill:#fff3e0
    style Processing fill:#e3f2fd
    style Storage fill:#f3e5f5
    style Output fill:#e8f5e9
```

---

## 12. Data Flow Diagram - Level 2 (Detailed Process View)

```mermaid
graph LR
    subgraph Wallet ["Wallet Module"]
        W1["User<br/>Input"]
        W2["Validate<br/>Address"]
        W3["Query<br/>Balance"]
        W4["Fetch<br/>Tokens"]
        W5["Get<br/>Metadata"]
    end

    subgraph Swap ["Swap Module"]
        S1["Input<br/>Amounts"]
        S2["Validate<br/>Amounts"]
        S3["Get<br/>Quote"]
        S4["Calculate<br/>Impact"]
        S5["Build Tx"]
    end

    subgraph Sign ["Transaction"]
        T1["Sign Tx<br/>MWA"]
        T2["Send<br/>to RPC"]
        T3["Poll<br/>Status"]
    end

    subgraph Storage ["Storage"]
        D1["Cache<br/>Tokens"]
        D2["Cache<br/>History"]
        D3["Cache<br/>Auth"]
    end

    W1 --> W2 --> W3 --> W4 --> W5 --> D1
    S1 --> S2 --> S3 --> S4 --> S5
    S5 --> T1 --> T2 --> T3 --> D2
    W2 --> D3

    style Wallet fill:#e1f5ff
    style Swap fill:#f3e5f5
    style Sign fill:#fff3e0
    style Storage fill:#e8f5e9
```

---

## 13. Data Flow Diagram - Level 3 (Detailed Subprocess Decomposition)

```mermaid
graph TB
    subgraph Balance ["Balance Fetch Process"]
        B1["Receive<br/>Public Key"]
        B2["RPC:<br/>getBalance"]
        B3["RPC:<br/>getTokenAccountsByOwner"]
        B4["Parse<br/>Response"]
        B5["Filter<br/>Amount > 0"]
        B6["Extract<br/>Mints"]
        B7["Output<br/>Token List"]
    end

    subgraph Metadata ["Metadata Fetch Process"]
        M1["Receive<br/>Mint Array"]
        M2["Call<br/>Jupiter API"]
        M3["Fallback<br/>Helius API"]
        M4["Parse<br/>Response"]
        M5["Build<br/>Map"]
        M6["Output<br/>Metadata"]
    end

    subgraph Quote ["Quote Fetch Process"]
        Q1["Receive<br/>Token Pair<br/>& Amount"]
        Q2["Validate<br/>Input"]
        Q3["Call<br/>Jupiter Quote"]
        Q4["Calculate<br/>Min Received"]
        Q5["Output<br/>Quote"]
    end

    subgraph Execute ["Swap Execution"]
        E1["Receive<br/>Quote"]
        E2["Call<br/>Jupiter Swap<br/>Endpoint"]
        E3["Decode<br/>Transaction"]
        E4["Sign via MWA"]
        E5["Send via RPC"]
        E6["Poll Status"]
        E7["Output<br/>Signature"]
    end

    B1 --> B2 --> B4
    B1 --> B3 --> B4
    B4 --> B5 --> B6 --> B7
    B7 --> M1

    M1 --> M2 --> M4 --> M5 --> M6

    Q1 --> Q2 --> Q3 --> Q4 --> Q5

    E1 --> E2 --> E3 --> E4 --> E5 --> E6 --> E7

    style Balance fill:#e3f2fd
    style Metadata fill:#f3e5f5
    style Quote fill:#fff3e0
    style Execute fill:#e8f5e9
```

---

## 14. Use Case Diagram

```mermaid
usecase UC1 as Connect Wallet
usecase UC2 as View Balance
usecase UC3 as View Tokens
usecase UC4 as View Token Details
usecase UC5 as Send SOL
usecase UC6 as Swap Tokens
usecase UC7 as View History
usecase UC8 as View Watchlist
usecase UC9 as Switch Network
usecase UC10 as Search Wallet

actor User

User --> UC1
User --> UC9

UC1 --> UC2
UC1 --> UC10

UC10 --> UC3
UC10 --> UC7

UC3 --> UC4
UC3 --> UC5
UC3 --> UC6
UC3 --> UC8

UC6 --> UC7
UC5 --> UC7

UC4 --> UC8

UC2 ..> UC1 : requires

UC6 ..> UC2 : checks balance
UC5 ..> UC2 : checks balance

UC9 ..> UC2 : affects data

rect rgb(200, 150, 255)
    note right of UC6, UC5
        Requires wallet connection
    end rect
```

---

## 15. Component Interaction Diagram

```mermaid
graph TB
    subgraph Screens ["Screen Components"]
        WalletScreen["WalletScreen"]
        SwapScreen["SwapScreen"]
        TokenDetail["TokenDetailScreen"]
        SettingsScreen["SettingsScreen"]
    end

    subgraph Contexts ["Context Providers"]
        NetworkCtx["NetworkContext<br/>- Toggle Network<br/>- Manage RPC"]
        WalletCtx["UserWalletContext<br/>- Connect/Disconnect<br/>- Sign Transactions"]
    end

    subgraph Hooks ["Custom Hooks"]
        useWalletScreen["useWalletScreen<br/>- Fetch tokens<br/>- Search wallet"]
        useSwapScreen["useSwapScreen<br/>- Get quotes<br/>- Execute swap"]
        useWatchlist["useWatchlist<br/>- Manage list"]
    end

    subgraph Services ["Service Layer"]
        SolanaService["Solana Service<br/>- RPC calls<br/>- Balance fetch"]
        SwapService["Swap Service<br/>- Quote fetch<br/>- Swap execution"]
        TokenService["Token Service<br/>- Metadata fetch<br/>- Details"]
    end

    subgraph External ["External APIs"]
        SolanaRPC["Solana RPC"]
        JupAPI["Jupiter API"]
        HeliusAPI["Helius API"]
        MWA["Mobile Wallet<br/>Adapter"]
    end

    WalletScreen --> NetworkCtx
    WalletScreen --> WalletCtx
    WalletScreen --> useWalletScreen
    
    SwapScreen --> useSwapScreen
    SwapScreen --> WalletCtx
    
    TokenDetail --> useWalletScreen
    
    SettingsScreen --> NetworkCtx
    SettingsScreen --> WalletCtx
    
    useWalletScreen --> SolanaService
    useSwapScreen --> SwapService
    useWatchlist --> SolanaService
    
    SolanaService --> SolanaRPC
    SolanaService --> HeliusAPI
    
    SwapService --> JupAPI
    SwapService --> SolanaRPC
    
    TokenService --> JupAPI
    TokenService --> HeliusAPI
    
    WalletCtx --> MWA
    WalletCtx --> SolanaRPC

    style Screens fill:#e3f2fd
    style Contexts fill:#f3e5f5
    style Hooks fill:#fff3e0
    style Services fill:#e8f5e9
    style External fill:#fce4ec
```

---

## 16. State Management Diagram

```mermaid
graph TB
    subgraph UserWalletState ["UserWallet State"]
        publicKey["publicKey: Address"]
        connected["connected: boolean"]
        connecting["connecting: boolean"]
        signing["signing: boolean"]
        sending["sending: boolean"]
        authToken["authToken: string"]
    end

    subgraph NetworkState ["Network State"]
        network["network: 'mainnet' | 'devnet'"]
        rpc["rpc: SolanaRpc"]
    end

    subgraph WalletScreenState ["WalletScreen State"]
        value["value: string"]
        loading["loading: boolean"]
        tokens["tokens: Token[]"]
        transactions["transactions: Transaction[]"]
        refreshing["refreshing: boolean"]
    end

    subgraph SwapScreenState ["SwapScreen State"]
        fromToken["fromToken: Token"]
        toToken["toToken: Token"]
        payAmount["payAmount: string"]
        slippage["slippage: number"]
        quote["quote: SwapQuote | null"]
        quoteLoading["quoteLoading: boolean"]
    end

    subgraph Storage ["MMKV Storage"]
        authStorage["auth-token-{network}"]
        addrStorage["wallet-address-{network}"]
        cacheStorage["token-metadata-cache"]
        historyStorage["swap-history"]
    end

    UserWalletState --> authStorage
    UserWalletState --> addrStorage
    NetworkState --> cacheStorage
    WalletScreenState --> cacheStorage
    SwapScreenState --> historyStorage

    style UserWalletState fill:#e3f2fd
    style NetworkState fill:#f3e5f5
    style WalletScreenState fill:#fff3e0
    style SwapScreenState fill:#e8f5e9
    style Storage fill:#fce4ec
```

---

## Usage Instructions

### For Mermaid Diagrams:
1. **Online**: Use [Mermaid Live Editor](https://mermaid.live)
2. **In Documentation**: Add code blocks with ` ```mermaid ` syntax
3. **In Notion/Confluence**: Use respective diagram plugins
4. **Generate as Images**: Use `mermaid-cli` package

### Installation & Usage:
```bash
npm install -g @mermaid-js/mermaid-cli

# Generate PNG
mmdc -i diagram.md -o diagram.png

# Generate SVG
mmdc -i diagram.md -o diagram.svg -t forest
```

### Themes:
- `default` - Light theme
- `forest` - Green theme
- `dark` - Dark theme
- `neutral` - Neutral theme

---

## Diagram Summary

| Diagram | Purpose | Type | Audience |
|---------|---------|------|----------|
| 1. Complete Project | System overview | Graph | Everyone |
| 2. ER Diagram | Data model | Entity Relationship | Developers, DBAs |
| 3. UML Class | Object design | Class Diagram | Developers |
| 4. Sequence - Connection | Wallet flow | Sequence | Developers |
| 5. Sequence - Balance | Data fetch flow | Sequence | Developers |
| 6. Sequence - Swap | Swap execution | Sequence | Developers |
| 7. Sequence - Send SOL | Transfer flow | Sequence | Developers |
| 8. Activity - Balance | Process steps | Activity | Business Analysts |
| 9. Activity - Swap | Process flow | Activity | Business Analysts |
| 10. DFD Level 0 | System context | Data Flow | Stakeholders |
| 11. DFD Level 1 | Main processes | Data Flow | Architects |
| 12. DFD Level 2 | Detailed modules | Data Flow | Developers |
| 13. DFD Level 3 | Subprocess detail | Data Flow | Developers |
| 14. Use Cases | User interactions | Use Case | Product Managers |
| 15. Component Interaction | Module relationships | Component | Developers |
| 16. State Management | App state | State Diagram | Frontend Developers |

---

Generated for Helio Solana Wallet Project
