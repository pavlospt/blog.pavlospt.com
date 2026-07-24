---
title: "Academic Schedule 4.0: Reviving an 11-Year-Old University App with AI Agents"
author: "Pavlos-Petros Tournaris"
date: 2026-07-24T12:00:00+03:00
lastmod: 2026-07-24T12:00:00+03:00
description: "How Antigravity CLI, Gemini 3.6 Flash, and DeepSeek v4 helped revive an abandoned Android app after 11 years — from legacy Java to modern Kotlin Compose with Cloudflare Workers and cross-platform cloud sync."
subtitle: "How Antigravity CLI, Gemini 3.6 Flash, and DeepSeek v4 helped revive an abandoned Android app after 11 years."
image: "/images/2026-07-24_academic-schedule-4-0-revival/banner.png"
images:
  - "/images/2026-07-24_academic-schedule-4-0-revival/banner.png"
  - "/images/2026-07-24_academic-schedule-4-0-revival/1.png"
  - "/images/2026-07-24_academic-schedule-4-0-revival/2.png"
  - "/images/2026-07-24_academic-schedule-4-0-revival/3.png"
  - "/images/2026-07-24_academic-schedule-4-0-revival/4.png"
tags: ["android", "kotlin", "compose", "cloudflare-workers", "ai", "antigravity", "gemini", "deepseek", "aueb"]
categories: ["android", "ai"]
series: []
# Academic Schedule 4.0

### *Reviving an 11-Year-Old University App with Antigravity, Gemini 3.6 Flash & DeepSeek v4*

---

![Play Store Promo Banner Scene 1](/images/2026-07-24_academic-schedule-4-0-revival/banner.png)

---

## 1. A Personal Note: Seeking Closure After 11 Years

Back in **2012**, when I was a student at the Athens University of Economics and Business (AUEB), I launched **Academic Schedule** (`com.auebcsschedule.ppt`). It was a native Android app meant to solve a daily frustration for thousands of students: finding course timetables, classroom locations, professor contacts, and exam dates without digging through clunky PDFs.

Life took unexpected turns. I never completed my degree and was eventually deleted from the university student registry. For almost **11 years**, the app sat abandoned on GitHub and Google Play. 

Yet, every time I looked at the old repository, I felt an unfinished story. I wanted to give myself—and the student community—true closure by publishing the definitive, ultimate version of this application.

With the arrival of autonomous AI coding agents, that dream became a reality.

---

## 2. Rebuilding the Backend: From Server Crash (2022) to Serverless XLSX Edge Scraping

In **2022**, the legacy backend server hosting the original database went offline permanently. Without a backend, the Android app was left stranded.

Rather than maintaining a costly traditional server, we built a brand-new backend on **Cloudflare Workers**:

{{< mermaid >}}
graph TD
    A["University Public Website"] -->|"Raw .xlsx Timetables"| B["Cloudflare Worker Scraper"]
    B -->|"Edge Parsing & Normalization"| C["Cloudflare D1 / KV Database"]
    C -->|"REST & JSON Endpoints"| D["Android App (100% Kotlin Compose)"]
    C -->|"REST & JSON Endpoints"| E["Web App (acschedule.app)"]
    D <-->|"Settings & Extra Lessons Sync"| C
    E <-->|"Settings & Extra Lessons Sync"| C
{{< /mermaid >}}

### **How the New Edge Backend Works**:
1. **Automated XLSX Scraping**: Cloudflare Workers scrape publicly published university schedule `.xlsx` spreadsheets.
2. **Serverless Edge Parsing**: The worker parses raw tabular data on the edge, standardizing department names, timeslots, and room codes into clean JSON schemas.
3. **Global Low-Latency API**: Served via Cloudflare KV/D1 with instant edge caching, powering both the **Android App** and the **Live Web UI** at **[acschedule.app](https://acschedule.app)**.

---

## 3. The 24-Hour AI-Driven Refactoring Story

Paired with **Antigravity CLI**, we deployed a multi-model workflow:
- **Gemini 3.6 Flash**: High-speed codebase traversal, rapid multi-file refactoring, and UI layout orchestration.
- **DeepSeek v4 (Flash + Pro)**: Complex architectural reasoning, multi-module DAG resolution, and UDF state management.

{{< mermaid >}}
timeline
    title Academic Schedule Overhaul Timeline
    2012 - 2015 : Initial Java Release : Native Java App with SQLite & AsyncTasks
    2015 - 2022 : Active Usage & Legacy Server : Server went offline in 2022
    July 23, 2026 : Phase 1 - 11-Module DAG : Modularized into :core:* and :feature:*
    July 23, 2026 : Phase 2 - Modern Android Stack : Jetpack Compose, Koin DI, Room DB, UDF & DataStore
    July 24, 2026 : Phase 3 - Web UI & Cloud Sync : Cloudflare Worker + React + TypeScript + Glassmorphism UI
    July 24, 2026 : Phase 4 - Play Store CI/CD : Triple-T Publisher, Maestro E2E, 1Password vault & v4.0 Release
{{< /mermaid >}}

---

## 4. Deep Tech: Architecture & Engineering Breakdown

For the developers and tech-savvy readers, here is the exact technical breakdown of how **Academic Schedule 4.0** was engineered:

### **A. Multi-Module DAG (11 Gradle Modules)**
To enforce clean separation of concerns and parallel build compilation, the monolith was refactored into **11 decoupled Gradle modules**:

```
Academic-Schedule/
├── ACSchedule/             # App Entry point & Application manifest
├── core/
│   ├── model/             # Pure Kotlin domain entities & DTOs
│   ├── database/          # Room DB entities, DAOs, & Migrations
│   ├── network/           # Ktor HTTP client & Worker API contracts
│   ├── data/              # Repositories & DataStore settings
│   ├── designsystem/      # Compose theme, colors, & typography
│   └── common/            # Utilities, Coroutines, & Dispatchers
└── feature/
    ├── schedule/          # Timetable view & day navigation
    ├── search/            # Room, time, & course search with filters
    ├── exams/             # Exam schedule timetable
    ├── professors/        # Faculty directory & bio details
    ├── settings/          # Cloud sync, Dept selection, & DataStore
    └── widget/            # Home screen AppWidget provider
```

---

### **B. Reactive State & Event Bus Elimination**
We eliminated the legacy static `EventBus` in favor of a thread-safe, reactive state model (`SessionState`):

```kotlin
// Legacy 2012 EventBus (Deprecated)
// AppEventBus.post(LessonSelectedEvent(id))

// Modern 2026 SessionState with SharedFlows
class SessionState {
    private val _selectedDepartment = MutableStateFlow<String?>(null)
    val selectedDepartment: StateFlow<String?> = _selectedDepartment.asStateFlow()

    private val _extraLessons = MutableStateFlow<Set<String>>(emptySet())
    val extraLessons: StateFlow<Set<String>> = _extraLessons.asStateFlow()

    fun updateExtraLessons(lessons: Set<String>) {
        _extraLessons.value = lessons
    }
}
```

---

### **C. Declarative Web UI System (`acschedule.app`)**
The web app (`worker/frontend`) was built without bulky UI frameworks, relying on a custom Vanilla CSS token system:
- **Backdrop Blur & Glassmorphism**: `backdrop-filter: blur(4px)`, custom HSL surface colors.
- **Custom Animated Dropdowns**: Accessible select components with floating menus and active checkmark indicators (`CustomSelect.tsx`).
- **Synchronized Local & Cloud State**: Automatic merge between browser `localStorage` and Cloudflare KV cloud backups upon authentication.

---

### **D. Release Automation & Secret Vaulting**
- **Triple-T Gradle Play Publisher (`com.github.triplet.play` v4.0.0)**: Integrated directly into `ACSchedule/build.gradle.kts` for 1-click automated releases:
  ```bash
  ./gradlew publishReleaseBundle
  ```
- **1Password CLI (`op`) Integration**: Automatically backed up all PEPKS private keys, upload keystores, and service account JSONs to 1Password.

---

## 5. Visual Showcase

````carousel
![Android App Schedule View](/images/2026-07-24_academic-schedule-4-0-revival/1.png)
<!-- slide -->
![Android Search & Filters](/images/2026-07-24_academic-schedule-4-0-revival/2.png)
<!-- slide -->
![Android Exam Timetable](/images/2026-07-24_academic-schedule-4-0-revival/3.png)
<!-- slide -->
![Android Cloud Settings Sync](/images/2026-07-24_academic-schedule-4-0-revival/4.png)
````

---

## 6. Experience it Live

- 🌐 **Live Web Application**: [acschedule.app](https://acschedule.app)
- 📱 **Google Play Store**: [Download Academic Schedule 4.0](https://play.google.com/store/apps/details?id=com.auebcsschedule.ppt)

---

### **Final Thought**
Academic Schedule 4.0 represents more than just a software update. It proves that with AI agents as force multipliers, no project is ever truly dead—and every unfinished story can get the closure it deserves.
