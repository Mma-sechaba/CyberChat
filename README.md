# 🔐 CyberChat — Cybersecurity Awareness Bot

A WPF desktop application that educates users on cybersecurity best practices through an interactive chatbot, a personal task manager, and a multiple-choice quiz. Built with C# and .NET 6 on Windows.

---

## Table of Contents

1. [Features](#features)
2. [Folder Structure](#folder-structure)
3. [File Descriptions](#file-descriptions)
4. [Requirements](#requirements)
5. [Setup & Installation](#setup--installation)
6. [Running the App](#running-the-app)
7. [Usage Instructions](#usage-instructions)
   - [Chatbot](#chatbot)
   - [Task Manager](#task-manager)
   - [Quiz](#quiz)
   - [Activity Log](#activity-log)
8. [Chat Command Reference](#chat-command-reference)
9. [Database Configuration](#database-configuration)
10. [Known Limitations](#known-limitations)
11. [Bug Fixes](#bug-fixes)

---

## Features

- **Chatbot** — Keyword-aware responses covering passwords, phishing, malware, privacy, Wi-Fi, and scams. Detects sentiment (worried, frustrated, curious) and remembers the user's topic interest within a session.
- **Task Manager** — Create, view, complete, and delete cybersecurity to-do items with optional reminder dates. Controllable from both the chat input and the UI panel.
- **Quiz** — Ten multiple-choice questions on core cybersecurity topics with immediate correct/incorrect feedback, explanations, and a final score with performance rating.
- **Activity Log** — Timestamped record of every task operation and quiz event during the session.
- **Audio Feedback** — Windows system sounds play on greeting, task creation, task completion, and quiz answers.

---

## Folder Structure

```
CyberChat/
│
├── CyberChat.sln                  # Visual Studio solution file
│
├── CyberChat/                     # Main project folder
│   │
│   ├── CyberChat.csproj           # Project file (targets net6.0-windows, UseWPF=true)
│   │
│   ├── App.xaml                   # WPF application entry point
│   ├── App.xaml.cs                # Application startup code
│   │
│   ├── MainWindow.xaml            # Main UI layout (all panels, tabs, controls)
│   ├── MainWindow.cs              # Code-behind (chat, task, quiz, log event handlers)
│   │
│   ├── ResponseManager.cs         # Keyword-based responses, topic detection, deeper info
│   ├── QuizManager.cs             # Quiz questions, answer submission, scoring
│   │
│   └── SupportingFiles.cs         # All supporting classes (see File Descriptions below):
│                                  #   - TaskManager
│                                  #   - TaskDatabase
│                                  #   - TaskItem
│                                  #   - ActivityLog
│                                  #   - MemoryManager
│                                  #   - AudioManager
│                                  #   - NlpHelper / NlpResult
│
└── README.md                      # This file
```

> **Note:** `SupportingFiles.cs` contains multiple classes in one file for convenience. You may split each class into its own `.cs` file — the namespace and class names do not need to change.

---

## File Descriptions

| File | Responsibility |
|---|---|
| `MainWindow.xaml` | Defines the entire UI: header, chat panel, right-panel tabs (Tasks / Quiz / Activity Log), and the bottom input bar. All controls are named with `x:Name` so WPF auto-generates their code-behind fields. |
| `MainWindow.cs` | Handles all user interactions: sending chat messages, NLP intent routing, task CRUD, quiz flow, sentiment/memory responses, and activity log display. |
| `ResponseManager.cs` | Stores keyword-to-response mappings for six topics. Exposes `GetResponse()`, `DetectTopic()`, and `GetMoreInfo()`. |
| `QuizManager.cs` | Holds the 10 `QuizQuestion` objects. Manages quiz state (current index, score). Exposes `Start()`, `Next()`, `SubmitAnswer()`, `GetScore()`. |
| `TaskManager.cs` (in SupportingFiles) | Business logic layer over `TaskDatabase`. Handles add, list, complete, delete. |
| `TaskDatabase.cs` (in SupportingFiles) | In-memory thread-safe store for `TaskItem` objects. Swap this class for a real DB implementation without touching `TaskManager`. |
| `TaskItem.cs` (in SupportingFiles) | Plain data model: `Id`, `Title`, `Description`, `ReminderAt`, `IsCompleted`, `CreatedAt`. |
| `ActivityLog.cs` (in SupportingFiles) | Thread-safe `LinkedList` capped at 100 entries. Prepends newest entries for O(1) insertion. |
| `MemoryManager.cs` (in SupportingFiles) | `ConcurrentDictionary` wrapper for session-scoped key/value memory (e.g. user's favourite topic). |
| `AudioManager.cs` (in SupportingFiles) | Static helpers wrapping `System.Media.SystemSounds` for five named audio events. |
| `NlpHelper.cs` (in SupportingFiles) | Regex + keyword parser. Returns an `NlpResult` with an `Intent`, extracted `Title`, optional `TaskId`, and parsed `ReminderAt`. |

---

## Requirements

| Requirement | Minimum version |
|---|---|
| Operating system | Windows 10 or later |
| .NET SDK | .NET 6.0 |
| IDE | Visual Studio 2022 (Community edition is free) |
| WPF workload | "**.NET desktop development**" workload installed in Visual Studio |

---

## Setup & Installation

### Step 1 — Install the .NET 6 SDK

Download and install the .NET 6 SDK from the official site:
```
https://dotnet.microsoft.com/en-us/download/dotnet/6.0
```
Verify the installation:
```bash
dotnet --version
# Expected output: 6.x.x
```

### Step 2 — Install Visual Studio 2022

Download Visual Studio 2022 (Community edition):
```
https://visualstudio.microsoft.com/vs/
```
During installation, select the **".NET desktop development"** workload. This includes WPF support.

### Step 3 — Clone or download the project

**Using Git:**
```bash
git clone https://github.com/your-username/CyberChat.git
cd CyberChat
```

**Or download the ZIP** from GitHub and extract it to a folder of your choice.

### Step 4 — Verify the project file

Open `CyberChat/CyberChat.csproj` and confirm it contains:
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net6.0-windows</TargetFramework>
    <UseWPF>true</UseWPF>
    <Nullable>enable</Nullable>
  </PropertyGroup>
</Project>
```

### Step 5 — Open the solution

Double-click `CyberChat.sln`, or open it from Visual Studio via **File → Open → Project/Solution**.

### Step 6 — Restore dependencies

Visual Studio restores NuGet packages automatically on first open. To do it manually:
```bash
dotnet restore
```

---

## Running the App

**From Visual Studio:**
Press **F5** to build and run with the debugger, or **Ctrl + F5** to run without.

**From the command line:**
```bash
cd CyberChat
dotnet run
```

**Build a release executable:**
```bash
dotnet publish -c Release -r win-x64 --self-contained true
# Output: bin/Release/net6.0-windows/win-x64/publish/CyberChat.exe
```

---

## Usage Instructions

### Chatbot

Type any message into the input bar at the bottom and press **Send** (or **Enter**).

The bot recognises six cybersecurity topics and responds with randomised tips:

| Topic keyword | Example input |
|---|---|
| `password` | `How do I create a strong password?` |
| `phishing` | `What is a phishing attack?` |
| `malware` | `How do I protect against malware?` |
| `privacy` | `How can I improve my privacy online?` |
| `wifi` | `Is public Wi-Fi safe to use?` |
| `scam` | `How do I spot an online scam?` |

**Example session:**
```
You:  What should I know about phishing?
Bot:  Scammers often pretend to be trusted organisations.
Bot:  Phishing scams often create fake urgency.

You:  I'm worried about my privacy settings.
Bot:  I understand your concern. Cybersecurity threats can feel overwhelming sometimes.
Bot:  Review your privacy settings regularly.
Bot:  Privacy settings help protect personal information online.
Bot:  Great! I'll remember that you're interested in privacy.
```

---

### Task Manager

Tasks can be managed two ways: via **chat commands** or the **Tasks tab** in the right panel.

#### Adding a task via the Tasks tab

1. Click the **Tasks** tab in the right panel.
2. Enter a **Title** (required) — e.g. `Enable two-factor authentication`.
3. Optionally enter a **Description** — e.g. `Set up an authenticator app on all accounts`.
4. Optionally pick a **Reminder date** using the date picker.
5. Click **Add Task**.

The task appears in the list below as:
```
[1] Enable two-factor authentication - Set up an authenticator app on all accounts
```

#### Managing existing tasks

1. Click a task in the list to select it.
2. Click **✔ Mark Complete** to mark it done — it will show `(Done)` in the list.
3. Click **✖ Delete** to remove it permanently.
4. Click **Refresh** to reload the list at any time.

---

### Quiz

#### Starting the quiz

Click **Start Quiz** in the **Quiz** tab, or type `start quiz` in the chat.

#### Answering questions

1. Read the question displayed in the Quiz tab.
2. Select one of the radio button options.
3. Click **Submit Answer**.
4. The chat panel shows whether your answer was correct and explains why.
5. The next question loads automatically.

#### Scoring

After all 10 questions, the Quiz tab shows your final score and a rating:

| Score | Rating |
|---|---|
| 8 – 10 | Great job! You're a cybersecurity pro! |
| 5 – 7  | Good work! Keep learning to stay safe online! |
| 0 – 4  | Keep learning to improve your cybersecurity skills! |

**Example quiz interaction:**
```
Bot:  Quiz started. Good luck!

[Quiz tab — Question 3 of 10]
What is 2FA (Two-Factor Authentication)?
○ A second password
● An additional verification method
○ Public Wi-Fi
○ A VPN

[Submit Answer]

Bot:  Correct! 2FA adds a second verification step such as an SMS code or authenticator app.
```

---

### Activity Log

The **Activity Log** tab keeps a timestamped record of all session events.

To view it:
- Click the **Activity Log** tab, or
- Type `show activity log` in the chat.

Click **Show More** to refresh the list. It displays the 10 most recent entries, for example:
```
2026-06-26 14:32:10 - Quiz completed - Score 8/10
2026-06-26 14:28:45 - Quiz answer: Correct - Q: What is 2FA?
2026-06-26 14:20:03 - Task marked complete (Id: 2)
2026-06-26 14:18:55 - Task added: 'Update antivirus software' (Id: 2)
2026-06-26 14:15:30 - Task added: 'Enable two-factor authentication' (Id: 1)
2026-06-26 14:15:00 - Quiz started
```

---

## Chat Command Reference

| Command | Intent | Example |
|---|---|---|
| `add task <title>` | Create a task | `add task review privacy settings` |
| `add a task to <title>` | Create a task | `add a task to install antivirus` |
| `remind me to <title>` | Create a task | `remind me to change my passwords` |
| `remind me to <title> tomorrow` | Task with next-day reminder | `remind me to update software tomorrow` |
| `remind me to <title> in 3 days` | Task with relative reminder | `remind me to backup files in 3 days` |
| `add task <title> on 2026-07-01` | Task with exact date reminder | `add task review accounts on 2026-07-01` |
| `show tasks` / `list tasks` | Open Tasks tab | `show tasks` |
| `complete task <id>` | Mark task done | `complete task 2` |
| `mark task <id>` | Mark task done | `mark task 3` |
| `delete task <id>` | Remove a task | `delete task 1` |
| `remove task <id>` | Remove a task | `remove task 4` |
| `start quiz` / `quiz me` | Launch the quiz | `start quiz` |
| `show activity log` | Open Activity Log tab | `show activity log` |

---

## Database Configuration

By default, tasks are stored **in memory** and are lost when the app closes.

To connect a persistent MySQL database:

### Step 1 — Install MySqlConnector

```bash
dotnet add package MySqlConnector
```

### Step 2 — Update the connection string

In `MainWindow.cs`, replace:
```csharp
private const string MySqlConnectionString =
    "Server=localhost;Database=cyberchat;User ID=root;Password=secret;SslMode=None;";
```
with your actual server credentials.

### Step 3 — Replace TaskDatabase

Rewrite the `TaskDatabase` class in `SupportingFiles.cs` to use MySqlConnector queries. The `TaskManager` class does not need any changes — it calls the same `InitializeAsync`, `AddTaskAsync`, `GetTasksAsync`, `UpdateTaskAsync`, and `DeleteTaskAsync` methods.

**Minimum required schema:**
```sql
CREATE DATABASE cyberchat;

USE cyberchat;

CREATE TABLE Tasks (
    Id          INT AUTO_INCREMENT PRIMARY KEY,
    Title       VARCHAR(255)  NOT NULL,
    Description TEXT,
    ReminderAt  DATETIME      NULL,
    IsCompleted TINYINT(1)    NOT NULL DEFAULT 0,
    CreatedAt   DATETIME      NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

---

## Known Limitations

- **NLP is keyword-based.** Complex or ambiguous phrasing may not be recognised. Stick to the command patterns listed above for best results.
- **Session memory only.** The `MemoryManager` stores data in a `ConcurrentDictionary` that is cleared when the app closes.
- **No task persistence by default.** Tasks are lost on restart unless a real database backend is configured (see above).
- **Windows only.** WPF and `System.Media.SystemSounds` are Windows-exclusive; the app will not compile or run on macOS or Linux.
- **Single user.** There is no login or multi-user support; all tasks and memory belong to whoever is running the app.

---

## Bug Fixes

The following bugs were identified and resolved:

| # | File | Bug | Fix applied |
|---|---|---|---|
| 1 | `MainWindow.cs` | `QuizScoreText` declared as `object` — `.Text` property inaccessible, compile error | Removed manual `FindName` properties; XAML `x:Name` generates typed fields automatically |
| 2 | `MainWindow.cs` | `RightTabControl`, `QuizOptionsPanel`, `QuizQuestionText`, `QuizScoreText` duplicated by both XAML auto-generation and manual `FindName` properties — ambiguity errors | Removed all four manual property declarations from the code-behind |
| 3 | `ResponseManager.cs` | Entire class defined twice in the same file — duplicate type compile error | Removed the second definition |
| 4 | `MainWindow.cs` | `DetectTopic()` result stored in `topic` but never used — dead code | `topic` is now passed to `GetMoreInfo()` and the result is displayed in the chat |
| 5 | `MainWindow.cs` | Memory recall message fired on every chat message after the first privacy mention | Changed the second `if` to `else if` so recall only fires when no new topic is being saved in the same turn |
