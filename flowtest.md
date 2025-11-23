# Isolated Components

This directory contains isolated, genericized versions of components from the XScape project. These components have been extracted and modified to remove dependencies on the larger codebase, according to proprietary guidelines.

## Components
### 1. TagManager
**File:** `TagManager.cs`
A component that manages tagging of items in a UI system. It maintains a dictionary of tagged items and their associated renderers.  
**Key Features:**
- Register/unregister tagged items
- Track multiple renderers per tagged item
- Query if items are tagged
- Editor-friendly debugging support

**Flow Diagram:**
```mermaid
flowchart TD
    A[RegisterThing Called] --> B{Thing Already Tagged?}
    B -->|Yes| C{Has Renderer?}
    B -->|No| D[Create New TagMap]
    C -->|Yes| E[Add Renderer to Existing List]
    C -->|No| F[Return]
    D --> G{Renderer Provided?}
    G -->|No| H[Set Name: Label + ID + Out of view]
    G -->|Yes| I{RendererName Valid?}
    I -->|Yes| J[Use RendererName]
    I -->|No| K[GetThingFieldValue 'name']
    K --> L{Field Value Found?}
    L -->|Yes| M[Use Field Value]
    L -->|No| N[Use Label + ID]
    H --> O[Store in Dictionary]
    J --> O
    M --> O
    N --> O
    E --> F
    O --> F
    
    P[IsThingTagged Called] --> Q{ID in Dictionary?}
    Q -->|Yes| R[Return True]
    Q -->|No| S[Return False]
    
    T[UnregisterThingTag Called] --> U[Remove from Dictionary]
```

### 2. AxelleBot
**File:** `AxelleBot.cs`  
A chatbot interface component that communicates with a backend API and provides a user-friendly chat interface.  
**Key Features:**
- Chat interface with input/output fields
- Web request handling for bot queries
- Command processing and event publishing
- Query history management
- Panel animation and UI controls

**Flow Diagram:**
```mermaid
flowchart TD
    A[User Clicks Send/Enter] --> B[Click_ChatButton]
    B --> C[Save Query to History]
    C --> D[Display User Message in Output]
    D --> E[Clear Input Field]
    E --> F[Show Thinking Panel]
    F --> G[Start Throbber Animation]
    G --> H[Start Request Timeout Coroutine]
    H --> I[ProcessBotRequest]
    I --> J{Query Contains '/'?}
    J -->|Yes| K[PublishExternalCommand]
    K --> L[Hide Thinking Panel]
    J -->|No| M[Encode Query for URL]
    M --> N[Get User Name]
    N --> O[Build API URL]
    O --> P[Send UnityWebRequest]
    P --> Q{Request Success?}
    Q -->|No| R[Display Error Message]
    R --> L
    Q -->|Yes| S[Extract Response Text]
    S --> T{Contains XSCEVENT?}
    T -->|Yes| U[Extract JSON from Response]
    U --> V[ParseJsonResponse]
    V --> W{Is UsrCmdE?}
    W -->|Yes| X[PublishExternalCommand with Parsed Cmd]
    W -->|No| Y[Display Text Response]
    X --> Y
    T -->|No| Y
    Y --> L
    
    Z[ToggleChatPanel] --> AA{Panel Visible?}
    AA -->|Yes| AB[Hide Panel]
    AB --> AC[Animate Panel Off Screen]
    AC --> AD[PublishToggleRenderZoom 'on']
    AA -->|No| AE[Show Panel]
    AE --> AF[Animate Panel On Screen]
    
    AG[Update Loop] --> AH{Panel Visible?}
    AH -->|Yes| AI{Key Pressed?}
    AI -->|Return| AJ[Trigger Send Button]
    AI -->|Up Arrow| AK[Access History]
    AH -->|Yes| AL[FreezeScrollIfNotOverUI]
```

### 3. Pagination
**File:** `Pagination.cs`  
A pagination component for managing large lists of UI items, extracted from the PanelUtils component.

**Key Features:**
- Paginate large lists of items
- Search functionality
- Page navigation (forward/backward)
- Configurable items per page
- World space canvas support

**Flow Diagram:**
```mermaid
flowchart TD
    A[Start/OnEnable] --> B[GetCommsManagerFromParent]
    B --> C[Wait for List Ready]
    C --> D[IndexList]
    D --> E[PurgeList]
    E --> F[GetItemsFromChildren]
    F --> G{For Each Item}
    G --> H{IsTemplateItem?}
    H -->|No| I[Add to trackedItems]
    H -->|Yes| J[Skip Item]
    I --> K[SetItemActive false]
    J --> K
    K --> L{More Items?}
    L -->|Yes| G
    L -->|No| M[Search]
    
    M --> N{Search Token Empty?}
    N -->|Yes| O[Add All Items to searchResults]
    N -->|No| P{For Each Item}
    P --> Q[GetItemFieldName]
    Q --> R{Name Contains Token?}
    R -->|Yes| S[Add to searchResults]
    R -->|No| T[Skip Item]
    S --> U{More Items?}
    T --> U
    U -->|Yes| P
    U -->|No| V[Paginate]
    
    V --> W{Found Results?}
    W -->|Yes| X[Calculate maxPages]
    W -->|No| Y[Display Empty State]
    X --> Z[Set currentPage = 1]
    Z --> AA[DisplayListContents]
    
    AA --> AB{Found Results?}
    AB -->|Yes| AC{maxPages > 1?}
    AC -->|Yes| AD[Show Pagination UI]
    AC -->|No| AE[Hide Pagination UI]
    AD --> AF[Show Page Counter]
    AE --> AF
    AF --> AG[Hide All Items]
    AG --> AH[Show Items in Range]
    AB -->|No| AI[Show No Results Message]
    
    AJ[OnNextPage] --> AK[currentPage++]
    AK --> AL{currentPage > maxPages?}
    AL -->|Yes| AM[currentPage = maxPages]
    AL -->|No| AN{currentPage == maxPages?}
    AM --> AN
    AN -->|Yes| AO[Hide Forward Button]
    AN -->|No| AP[Show Both Buttons]
    AO --> AA
    AP --> AA
    
    AQ[OnPreviousPage] --> AR[currentPage--]
    AR --> AS{currentPage <= 0?}
    AS -->|Yes| AT[currentPage = 1]
    AS -->|No| AU{currentPage == 1?}
    AT --> AU
    AU -->|Yes| AV[Hide Backward Button]
    AU -->|No| AW[Show Both Buttons]
    AV --> AA
    AW --> AA
```

### 4. S3UploadUI
**File:** `S3UploadUI.cs`
A UI component for uploading files to AWS S3  
**Key Features:**
- Input fields for cloud storage credentials
- File upload progress tracking
- Batch file upload support
- Error handling and status reporting
- Configurable build path

**Flow Diagram:**
```mermaid
flowchart TD
    A[Start] --> B[Initialize UI Components]
    B --> C[GetBuildPath]
    C --> D[Wait for User Input]
    
    D --> E[User Clicks Upload]
    E --> F{isUploading?}
    F -->|Yes| G[Show Error: Already Uploading]
    F -->|No| H[ValidateInput]
    H --> I{All Fields Valid?}
    I -->|No| J[Show Error: Fill All Fields]
    I -->|Yes| K[Get Credentials from Fields]
    K --> L[Start UploadToS3 Coroutine]
    
    L --> M[Set isUploading = true]
    M --> N[Show Progress Panel]
    N --> O[GetFilesToUpload]
    O --> P{Files Found?}
    P -->|No| Q[Show Error: No Files]
    P -->|Yes| R[InitializeStorageClient]
    R --> S{Client Initialized?}
    S -->|No| T[Show Error: Client Failed]
    S -->|Yes| U[For Each File]
    
    U --> V{File Exists?}
    V -->|No| W[Log Warning: Skip File]
    V -->|Yes| X[Calculate Relative Path]
    X --> Y[Calculate S3 Key]
    Y --> Z[Update Status: Uploading...]
    Z --> AA[UploadFile]
    AA --> AB{Upload Success?}
    AB -->|Yes| AC[Increment uploadedFiles]
    AB -->|No| AD[Show Error: Upload Failed]
    AC --> AE[Update Progress Bar]
    AE --> AF[Update Progress Text]
    AD --> AG{More Files?}
    AF --> AG
    W --> AG
    AG -->|Yes| U
    AG -->|No| AH{All Files Uploaded?}
    
    AH -->|Yes| AI[Show Success Message]
    AH -->|No| AJ[Show Partial Success Message]
    AI --> AK[Hide Progress Panel]
    AJ --> AK
    Q --> AK
    T --> AK
    AK --> AL[Set isUploading = false]
    
    AM[User Clicks Cancel] --> AN{isUploading?}
    AN -->|Yes| AO[CancelUpload]
    AN -->|No| AP[Hide UI]
    AO --> AQ[Set isUploading = false]
    AQ --> AR[Show Cancelled Message]
    AR --> AS[Hide Progress Panel]
```

## Proprietary Dependencies
- XBase.ThingPtr
- XScape.ThingRenderer
- Axomem.XScape.Core.Events.EventHub

## External Dependencies
- DOTween
- Newtonsoft.Json
- AWS SDK

## Notes
All proprietary and external dependencies have been replaced with generic function placeholders. Event system calls have been abstracted to generic functions. 
These components demonstrate:
- UI system architecture
- Event-driven programming patterns
- Async/await and coroutine patterns
- Generic programming and abstraction
- Cloud storage integration patterns
- Pagination and search algorithms

