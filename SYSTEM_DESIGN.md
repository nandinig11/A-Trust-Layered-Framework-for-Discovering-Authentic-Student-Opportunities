# CHAPTER 4. SYSTEM DESIGN

## 4.1 System Architecture

StuScout is a role-based skill-first hiring platform built with **Next.js**, **Clerk Authentication**, and **Supabase PostgreSQL**. The system supports two primary user roles: **Student** and **Recruiter**. Students create rich technical profiles, while recruiters post job roles, discover matched students, and manage shortlist pipelines.

The application follows a modern web architecture with:
- **Presentation Layer**: Next.js App Router pages, layouts, and reusable React components.
- **Application Layer**: Server actions, routing logic, dashboard data mappers, scoring engine, and matching engine.
- **Authentication Layer**: Clerk handles sign-up, sign-in, session management, and user identity.
- **Data Layer**: Supabase PostgreSQL stores profiles, companies, student profiles, job roles, and shortlists.

### Architectural Style
The project mainly follows a **layered architecture**:
- UI Layer
- Business Logic Layer
- Data Access Layer
- Persistence Layer

### Core Modules
- Landing and public authentication flow
- Onboarding flow
- Student dashboard and profile management
- Recruiter dashboard and role management
- Matching and ranking module
- Shortlist and pipeline management
- Theme management and shared UI components

### High-Level Architecture Diagram

```mermaid
flowchart TD
    A[User Browser] --> B[Next.js Frontend]
    B --> C[Clerk Authentication]
    B --> D[Server Actions / Route Logic]
    D --> E[Supabase Client Server-Side]
    E --> F[(Supabase PostgreSQL)]

    B --> G[Student Dashboard]
    B --> H[Recruiter Dashboard]

    D --> I[Scoring Engine]
    D --> J[Matching Engine]
```

### Codebase Evidence
- Root app shell: `src/app/layout.tsx`
- Global providers: `src/components/Providers.tsx`
- Auth/session bridge: `src/lib/session.ts`
- Server-side Supabase client: `src/lib/supabase/server.ts`
- Student data aggregation: `src/lib/data/student.ts`
- Recruiter data aggregation: `src/lib/data/recruiter.ts`
- Matching logic: `src/lib/matching/v1.ts`
- Scoring logic: `src/lib/scoring/v1.ts`

---

## 4.2 Data Flow Diagram

### 4.2.1 Level 0 Data Flow Diagram

The system receives input from users, validates identity through Clerk, processes role-based logic, and persists/retrieves records from Supabase.

```mermaid
flowchart LR
    U[User] --> S[StuScout System]
    S --> A[Clerk Auth]
    S --> D[(Supabase Database)]
    A --> S
    D --> S
    S --> U
```

### 4.2.2 Level 1 Data Flow Diagram

```mermaid
flowchart TD
    U1[Student] --> P1[Authentication & Onboarding]
    U2[Recruiter] --> P1

    P1 --> D1[(Profiles)]
    P1 --> D2[(Companies)]
    P1 --> D3[(Student Profiles)]

    U1 --> P2[Update Student Profile]
    P2 --> D3
    P2 --> P3[Scoring Engine]
    P3 --> D3

    U2 --> P4[Create Job Role]
    P4 --> D4[(Job Roles)]

    U2 --> P5[View Matches]
    P5 --> D3
    P5 --> D4
    P5 --> P6[Matching Engine]

    U2 --> P7[Manage Shortlist]
    P7 --> D5[(Shortlists)]

    D1 --> O1[Student Dashboard]
    D3 --> O1
    D1 --> O2[Recruiter Dashboard]
    D2 --> O2
    D4 --> O2
    D5 --> O2
```

### Main Data Flows
1. User signs in with Clerk.
2. System checks whether a matching profile exists in Supabase.
3. If absent, user is redirected to onboarding.
4. Student onboarding creates `profiles` and `student_profiles`.
5. Recruiter onboarding creates `profiles`, `companies`, and links `company_id`.
6. Student profile updates recompute composite score.
7. Recruiter job roles are matched against student profiles.
8. Recruiter can shortlist and update candidate pipeline status.

---

## 4.3 UML Diagrams

## 4.3.1 Class Diagram
`images/class-diagram.jpg`

**Figure 4.3: Class Diagram**

Since the project is implemented with functional TypeScript modules instead of traditional OOP-heavy classes, the class diagram is modeled using the main domain entities and service modules.

```mermaid
classDiagram
    class Profile {
        +uuid id
        +string clerk_user_id
        +string email
        +string name
        +string role
        +uuid company_id
    }

    class Company {
        +uuid id
        +string name
        +uuid owner_id
    }

    class StudentProfile {
        +uuid id
        +uuid user_id
        +string display_name
        +string college
        +string[] skills
        +jsonb projects
        +jsonb certifications
        +jsonb achievements
        +jsonb challenge_attempts
        +jsonb questionnaire
        +int composite_score
        +jsonb score_breakdown
    }

    class JobRole {
        +uuid id
        +uuid company_id
        +uuid created_by
        +string title
        +string[] skill_tags
        +string status
    }

    class Shortlist {
        +uuid id
        +uuid role_id
        +uuid recruiter_id
        +jsonb entries
    }

    class SessionService {
        +getAuthSession()
        +getProfileByClerkUserId()
    }

    class ScoringService {
        +computeCompositeScoreV1()
    }

    class MatchingService {
        +matchRankScore()
        +explainMatch()
    }

    Profile "1" --> "0..1" Company : belongs to
    Profile "1" --> "0..1" StudentProfile : owns
    Company "1" --> "*" JobRole : has
    JobRole "1" --> "0..1" Shortlist : has
    SessionService ..> Profile
    ScoringService ..> StudentProfile
    MatchingService ..> StudentProfile
    MatchingService ..> JobRole
```

---

## 4.3.2 Object Diagram
`images/object-diagram.jpg`

**Figure 4.4: Object Diagram**

This diagram shows example runtime objects in the system.

```mermaid
flowchart LR
    P1["profile: Profile
    name = Narayan
    role = recruiter"]

    C1["company: Company
    name = Infosys"]

    R1["role: JobRole
    title = Frontend Developer Intern"]

    P2["studentProfile: StudentProfile
    display_name = Asha
    skills = React, TypeScript"]

    S1["shortlist: Shortlist
    status = shortlisted"]

    P1 --> C1
    C1 --> R1
    R1 --> S1
    S1 --> P2
```

---

## 4.3.3 Collaboration Diagram
`images/collaboration-diagram.jpg`

**Figure 4.5: Collaboration Diagram**

```mermaid
flowchart TD
    Recruiter --> LoginPage
    LoginPage --> Clerk
    Clerk --> PostLogin
    PostLogin --> RecruiterDashboard
    RecruiterDashboard --> RoleService
    RoleService --> Supabase
    RecruiterDashboard --> MatchingService
    MatchingService --> StudentProfiles
    MatchingService --> ShortlistService
    ShortlistService --> Supabase
```

---

## 4.3.4 Use Case Diagram
`images/usecase-diagram.jpg`

**Figure 4.6: Use Case Diagram**

```mermaid
flowchart LR
    Student((Student))
    Recruiter((Recruiter))
    Clerk((Clerk Auth))
    System[StuScout System]

    Student -->|Sign up / Log in| Clerk
    Recruiter -->|Sign up / Log in| Clerk

    Student -->|Complete onboarding| System
    Student -->|Update profile| System
    Student -->|View dashboard| System
    Student -->|Manage projects, certifications, achievements| System

    Recruiter -->|Complete onboarding| System
    Recruiter -->|Create job roles| System
    Recruiter -->|View matches| System
    Recruiter -->|Shortlist students| System
    Recruiter -->|Update pipeline| System
    Recruiter -->|View recruiter dashboard| System
```

Department of Computer Science and Engineering 11

## 4.3 UML DIAGRAMS CHAPTER 4. SYSTEM DESIGN

## 4.3.5 Sequence Diagram
`images/sequence-diagram.jpg`

**Figure 4.7: Sequence Diagram**

Example: Recruiter logs in and views matched students for a role.

```mermaid
sequenceDiagram
    actor Recruiter
    participant UI as Next.js UI
    participant Clerk as Clerk Auth
    participant Session as Session Service
    participant DB as Supabase
    participant Match as Matching Engine

    Recruiter->>UI: Sign in
    UI->>Clerk: Authenticate user
    Clerk-->>UI: userId/session
    UI->>Session: getAuthSession()
    Session->>DB: fetch profile by clerk_user_id
    DB-->>Session: recruiter profile
    Session-->>UI: authenticated recruiter

    Recruiter->>UI: Open role matches page
    UI->>DB: fetch role details
    UI->>DB: fetch student_profiles
    UI->>Match: matchRankScore(roleTags, studentSkills, compositeScore)
    Match-->>UI: ranked matches
    UI-->>Recruiter: show top matched students
```

---

## 4.3.6 Activity Diagram
`images/activity-diagram.jpg`

**Figure 4.8: Activity Diagram**

Example: Student onboarding and profile completion activity flow.

```mermaid
flowchart TD
    A[Start] --> B[User signs in with Clerk]
    B --> C{Profile exists?}
    C -- No --> D[Open onboarding form]
    D --> E[Select role]
    E --> F{Student or Recruiter?}
    F -- Student --> G[Create profile row]
    G --> H[Create student_profile row]
    H --> I[Redirect to student dashboard]
    F -- Recruiter --> J[Create profile row]
    J --> K[Create company row]
    K --> L[Link company_id]
    L --> M[Redirect to recruiter dashboard]
    C -- Yes --> N[Redirect to role dashboard]
    I --> O[End]
    M --> O
    N --> O
```

---

## 4.3.7 Component Diagram
`images/component-diagram.jpg`

**Figure 4.9: Component Diagram**

```mermaid
flowchart TD
    A[Landing Module]
    B[Authentication Module]
    C[Onboarding Module]
    D[Student Dashboard Module]
    E[Recruiter Dashboard Module]
    F[Scoring Module]
    G[Matching Module]
    H[Supabase Data Layer]
    I[Clerk Provider]
    J[Theme Provider]

    A --> B
    B --> I
    B --> C
    C --> H
    D --> H
    D --> F
    E --> H
    E --> G
    E --> D
    J --> D
    J --> E
```

### Major Components
- `LandingHome`, `LandingNav`, section components
- Auth pages: login, register, post-login
- Onboarding form and server action
- Student shell, student overview, profile form
- Recruiter shell, recruiter overview, role pages, pipeline forms
- Scoring engine
- Matching engine
- Supabase data access utilities
- Theme provider and shared UI primitives

---

## 4.3.8 Deployment Diagram
`images/deployment-diagram.jpg`

**Figure 4.10: Deployment Diagram**

```mermaid
flowchart TD
    C[Client Browser]
    N[Next.js Application Server]
    K[Clerk Cloud]
    S[Supabase Cloud PostgreSQL]

    C --> N
    N --> K
    N --> S
```

### Deployment Description
- The **client browser** accesses the Next.js web app.
- The **Next.js server** renders pages, executes server actions, and talks to Clerk and Supabase.
- **Clerk Cloud** manages authentication and sessions.
- **Supabase** stores persistent application data.

---

## 4.4 Database Design Summary

### Main Tables
1. `profiles`
   - Stores core user identity mapped to Clerk.
   - Fields: `id`, `clerk_user_id`, `email`, `name`, `role`, `company_id`

2. `companies`
   - Stores recruiter organization data.
   - Fields: `id`, `name`, `owner_id`

3. `student_profiles`
   - Stores student-specific profile data and scoring signals.
   - Fields: `display_name`, `college`, `skills`, `projects`, `certifications`, `achievements`, `challenge_attempts`, `questionnaire`, `composite_score`, `score_breakdown`

4. `job_roles`
   - Stores recruiter-posted roles.
   - Fields: `company_id`, `created_by`, `title`, `skill_tags`, `status`

5. `shortlists`
   - Stores candidate shortlist data per role.
   - Fields: `role_id`, `recruiter_id`, `entries`

---

## 4.5 Important Functional Flows

### 4.5.1 Authentication Flow
- User signs in or signs up via Clerk.
- Clerk returns authenticated session.
- App checks `profiles` table.
- If no profile exists, redirect to onboarding.
- If profile exists, redirect to student or recruiter dashboard.

### 4.5.2 Student Profile Flow
- Student updates profile fields and questionnaire.
- Server action validates form using Zod.
- Skills are normalized.
- URLs are sanitized.
- Composite score is recomputed.
- Updated profile is stored in Supabase.
- Dashboard reads the refreshed data and renders insights.

### 4.5.3 Recruiter Hiring Flow
- Recruiter creates a job role.
- System stores role with normalized skill tags.
- Matching engine compares role skill tags with student skills.
- Students are ranked using:
  - skill overlap
  - Jaccard similarity
  - student composite score
- Recruiter shortlists students and updates pipeline states:
  - shortlisted
  - scheduled
  - offered

---

## 4.6 Algorithms Used

### 4.6.1 Composite Score Calculation
The student composite score is computed using weighted signals:
- Projects
- Questionnaire
- Skills
- Assessment stub score
- Endorsements

### 4.6.2 Match Ranking Algorithm
The recruiter match ranking combines:
- Skill overlap similarity
- Student composite score

General formula:
- `match score = 0.6 * skill similarity + 0.4 * normalized composite score`

---

## 4.7 Technology Stack

- **Frontend**: Next.js 16, React 19, Tailwind CSS
- **Authentication**: Clerk
- **Database**: Supabase PostgreSQL
- **Validation**: Zod
- **UI/Animation**: Lucide React, Framer Motion, Sonner
- **Language**: TypeScript

---

## 4.8 Conclusion

The StuScout system is designed as a scalable, role-based recruitment platform focused on skill-first hiring. Its architecture separates presentation, authentication, business logic, and persistence concerns cleanly. Clerk ensures secure authentication, while Supabase provides structured relational storage. The scoring and matching modules add domain-specific intelligence for ranking students and helping recruiters make data-informed hiring decisions.
