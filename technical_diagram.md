# BlueOcean Invoice Automation System Architecture

## System Overview Diagram

```mermaid
graph TD
    %% Main System Components
    User([User])
    Master[Master Process]
    Driver[Invoice Driver]
    Portal[(Vendor Portal)]
    SharePoint[(SharePoint)]
    KeyVault[(Azure Key Vault)]
    Email[Email Service]
    
    %% Subcomponents
    Setup[Browser Setup]
    Login[Authentication]
    DataLoad[Load Lookup Data]
    Selection[Invoice Selection]
    Download[Invoice Download]
    Validation[Validation]
    Transfer[File Transfer]
    
    %% Connections
    User -->|Initiates| Master
    Master -->|Orchestrates| Driver
    Driver -->|Uses| Setup
    Driver -->|Manages| Login
    Driver -->|Processes| DataLoad
    Driver -->|Controls| Selection
    Driver -->|Executes| Download
    Driver -->|Performs| Validation
    Driver -->|Handles| Transfer
    
    %% External System Connections
    Login -->|Authenticates with| Portal
    KeyVault -->|Provides credentials to| Login
    DataLoad -->|Retrieves data from| SharePoint
    Portal -->|Provides invoices to| Download
    Download -->|Saves to| LocalStorage[(Local Storage)]
    LocalStorage -->|Files transferred to| SharePoint
    Driver -->|Sends notifications via| Email
    
    %% Styling
    classDef primary fill:#4361ee,stroke:#2b2d42,color:white;
    classDef secondary fill:#3f37c9,stroke:#2b2d42,color:white;
    classDef storage fill:#2b2d42,stroke:#4361ee,color:white;
    classDef external fill:#f72585,stroke:#2b2d42,color:white;
    classDef user fill:#38b000,stroke:#2b2d42,color:white;
    
    class Master,Driver primary;
    class Setup,Login,DataLoad,Selection,Download,Validation,Transfer secondary;
    class SharePoint,KeyVault,LocalStorage storage;
    class Portal,Email external;
    class User user;
```

## Process Flow Diagram

```mermaid
flowchart TD
    Start([Start]) --> ParseArgs[Parse Arguments]
    ParseArgs --> SetupDriver[Setup Browser]
    SetupDriver --> LoginToPortal[Login to Portal]
    LoginToPortal --> OTPAuth{OTP Auth}
    
    OTPAuth -->|Success| GetLookupData[Get Lookup Data]
    OTPAuth -->|Failure| RetryLogin{Retry?}
    RetryLogin -->|Yes| LoginToPortal
    RetryLogin -->|No| SendErrorEmail[Error Email]
    
    GetLookupData --> ProcessLookupFile[Process Lookup]
    ProcessLookupFile --> SelectInvoice[Select Invoice]
    SelectInvoice --> ConstructURL[Construct URL]
    ConstructURL --> DownloadInvoices[Download]
    
    DownloadInvoices --> ValidateDownloads{Validate?}
    ValidateDownloads -->|Success| MoveToSharePoint[Move to SharePoint]
    ValidateDownloads -->|Failure| SendErrorEmail
    
    MoveToSharePoint --> SendSuccessEmail[Success Email]
    SendSuccessEmail --> End([End])
    SendErrorEmail --> End
    
    classDef primary fill:#4361ee,stroke:#2b2d42,color:white;
    classDef success fill:#38b000,stroke:#2b2d42,color:white;
    classDef error fill:#d90429,stroke:#2b2d42,color:white;
    classDef decision fill:#3f37c9,stroke:#2b2d42,color:white;
    
    class Start,End,ParseArgs,SetupDriver,LoginToPortal,GetLookupData,ProcessLookupFile,SelectInvoice,ConstructURL,DownloadInvoices primary;
    class MoveToSharePoint,SendSuccessEmail success;
    class SendErrorEmail error;
    class OTPAuth,RetryLogin,ValidateDownloads decision;
```

## Data Flow Diagram

```mermaid
flowchart LR
    A[(Azure Key Vault)] -->|Credentials| B(Invoice Driver)
    C[(SharePoint)] -->|Lookup File| B
    B -->|Login| D[Vendor Portal]
    D -->|Invoice Data| B
    B -->|Files| E[Local Storage]
    E -->|Transfer| C
    B -->|Notifications| F[Email System]
    
    classDef database fill:#2b2d42,stroke:#4361ee,color:white;
    classDef primary fill:#4361ee,stroke:#2b2d42,color:white;
    classDef system fill:#3f37c9,stroke:#2b2d42,color:white;
    
    class A,C,E database;
    class B primary;
    class D,F system;
```

## Component Interaction Sequence

```mermaid
sequenceDiagram
    participant User as User
    participant Master as Master Process
    participant Driver as Invoice Driver
    participant Portal as Vendor Portal
    participant KeyVault as Azure Key Vault
    participant SharePoint as SharePoint
    participant Email as Email Service

    User->>Master: Start automation
    Master->>Driver: Initialize
    Driver->>KeyVault: Request credentials
    KeyVault-->>Driver: Return credentials
    Driver->>Portal: Login
    Portal-->>Driver: Authentication challenge
    Driver->>Portal: Provide OTP
    Portal-->>Driver: Authentication success
    Driver->>SharePoint: Request lookup data
    SharePoint-->>Driver: Return lookup file
    Driver->>Driver: Process lookup data
    Driver->>Portal: Select invoices
    Portal-->>Driver: Return invoice list
    Driver->>Portal: Download invoices
    Portal-->>Driver: Invoice PDFs
    Driver->>Driver: Validate downloads
    Driver->>SharePoint: Upload invoices
    Driver->>Email: Send completion notification
    Email-->>User: Delivery notification
    Driver-->>Master: Complete process
    Master-->>User: End automation
```

## Error Handling Flow

```mermaid
flowchart TD
    Error[Error Detected] --> Classify{Classify Error}
    
    Classify -->|Authentication| AuthError[Auth Error]
    Classify -->|Data| DataError[Data Error]
    Classify -->|Download| DownloadError[Download Error]
    Classify -->|Transfer| TransferError[Transfer Error]
    
    AuthError --> AuthRetry{Retry?}
    AuthRetry -->|Yes| RetryAuth[Retry Authentication]
    AuthRetry -->|No| LogAuthError[Log Error]
    
    DataError --> DataRetry{Retry?}
    DataRetry -->|Yes| RetryData[Retry Data Load]
    DataRetry -->|No| LogDataError[Log Error]
    
    DownloadError --> DownloadRetry{Retry?}
    DownloadRetry -->|Yes| RetryDownload[Retry Download]
    DownloadRetry -->|No| LogDownloadError[Log Error]
    
    TransferError --> TransferRetry{Retry?}
    TransferRetry -->|Yes| RetryTransfer[Retry Transfer]
    TransferRetry -->|No| LogTransferError[Log Error]
    
    LogAuthError --> NotifyError[Send Notification]
    LogDataError --> NotifyError
    LogDownloadError --> NotifyError
    LogTransferError --> NotifyError
    
    NotifyError --> End([End With Error])
    
    classDef error fill:#d90429,stroke:#2b2d42,color:white;
    classDef decision fill:#3f37c9,stroke:#2b2d42,color:white;
    classDef action fill:#4361ee,stroke:#2b2d42,color:white;
    classDef end fill:#38b000,stroke:#2b2d42,color:white;
    
    class Error,LogAuthError,LogDataError,LogDownloadError,LogTransferError,NotifyError error;
    class AuthRetry,DataRetry,DownloadRetry,TransferRetry,Classify decision;
    class RetryAuth,RetryData,RetryDownload,RetryTransfer action;
    class End end;
```

## How to Use These Diagrams

1. **View in Markdown Editor**: Open this file in any Markdown editor that supports Mermaid syntax (like VS Code with Mermaid extension).

2. **Export as Images**:
   - Render the diagrams in a Markdown preview
   - Take screenshots or use browser tools to save as images
   - Upload the images to Miro

3. **Use Mermaid Live Editor**:
   - Copy the Mermaid code blocks
   - Paste into [Mermaid Live Editor](https://mermaid.live/)
   - Export as SVG/PNG for high-quality images
   - Import into Miro

4. **Modify as Needed**:
   - Update the diagrams by editing the Mermaid syntax
   - Add new components or connections as your system evolves
