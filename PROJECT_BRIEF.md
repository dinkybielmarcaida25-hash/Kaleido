```markdown
# PROJECT BRIEF — Kaleido: An Offline Psychology Review Companion

> **Repository:** _to be filled with GitHub URL_
> **Brief version:** 1.0
> **Last updated:** 2026-10-18
> **Maintained by:** human developer (Lenovo / Windows / Android Studio / beginner Kotlin)
> **Canonical spec:** `KALEIDO — MASTER DEVELOPMENT PROMPT.txt` (in repo root)
> **If this brief and the master prompt disagree, the master prompt wins.**
> **If this brief and the actual code disagree, the code wins — and this brief must be updated.**

---

## 1. What this project is

**Kaleido** is a free, **fully offline**, local-first Android learning application for Filipino undergraduate psychology students preparing for the Psychometrician licensure examination.

**Three learning modes:**
1. **Test** — multiple-choice practice questions, answer-key-based scoring, explanations, progress tracking.
2. **Read** — offline psychology reviewers organized by subject.
3. **Real Life Scenario** — educational patient-interview simulations with deterministic (NOT AI) patient responses, diagnosis submission, and rule-based feedback.

**Four subjects** (all four must be represented across Test, Read, and Scenario):
- Developmental Psychology
- Psychological Assessment
- Abnormal Psychology
- Industrial-Organizational Psychology

**Explicitly out of scope (any phase unless master prompt is amended):**
- Any backend, cloud database, auth service, or remote API
- Analytics, telemetry, advertising, crash reporting SDKs
- Firebase, App Check, Play Integrity remote calls
- Downloading or bundling a language model in Phase 1
- Claiming endorsement by CHED, PRC, or any licensing body

**Educational safeguards (non-negotiable):**
- All patient simulations use fictional patients only.
- All scenarios are labeled **educational simulations**, not clinical diagnoses.
- All sample content is labeled **"demonstration content pending review by qualified psychology professionals."**
- Diagnostic answer criteria are stored in structured application data — never in free-form AI output.
- AI, when introduced in a later phase, must be optional, on-device, isolated behind `AiEngine`, and clearly labeled "AI-generated, may be inaccurate."

---

## 2. Absolute constraints (do not violate)

These are extracted verbatim from the master prompt §5 and §10. **Any AI that violates these must be corrected immediately.**

| Constraint | Enforcement |
|---|---|
| No `android.permission.INTERNET` in manifest | Phase 1 must launch and be fully usable in airplane mode |
| No runtime network calls of any kind | No Retrofit, OkHttp, Ktor, HttpURLConnection, WebView loading remote URLs |
| No cloud, no analytics, no ads, no auth | Dependency review on every PR |
| All sample content bundled in `app/src/main/assets/` | No lazy-fetch, no "download on first run" |
| All local student data stays on-device | No `SharedPreferences` to cloud, no backup to Drive by default |
| No AI implementation in Phase 1 | `AiEngine.kt` is an interface with zero implementations |
| No DI framework (Hilt/Dagger/Koin) | Manual `AppContainer` only |
| Never claim build/test success without real evidence | See §10 "Verification protocol" below |
| Never delete working features without approval | See §9 "Collaboration protocol" below |
| Never replace the whole project to "fix" something | Small increments only |

---

## 3. Tech stack (locked — verify versions in `gradle/libs.versions.toml` when present)

| Component | Choice | Notes |
|---|---|---|
| Language | Kotlin 2.1.x | Compose compiler bundled with Kotlin since 2.0 |
| UI | Jetpack Compose + Material 3 | Compose BOM managed |
| Navigation | Navigation Compose | String routes currently; type-safe routes considered later |
| Local prefs | DataStore Preferences | `StudentProfile` uses this |
| Structured data | **Room** — *not yet added* | Slated for Increment B |
| JSON parsing | **kotlinx.serialization** — *not yet added* | For assets/reviewers/questions/cases |
| Coroutines/Flow | Yes, throughout | `Flow<StudentProfile?>`, `viewModelScope` |
| Architecture | MVVM + Repository + manual `AppContainer` | No DI framework |
| minSdk | 26 | Do not lower without a documented reason |
| JDK | 17 | Android Studio bundled: `C:\Program Files\Android\Android Studio\jbr` |
| Gradle | Wrapper committed to repo | Never assume globally installed Gradle |

**Do not add dependencies** without: (a) explaining why the current stack cannot do it, (b) listing its transitive deps, (c) confirming it works offline.

---

## 4. Architecture

```
┌──────────────────────────────────────────────────────┐
│  UI (Compose, Material 3)                            │
│  feature/*/                                         │
│    SplashScreen · OnboardingScreen · HomeScreen ·   │
│    TestScreen · ReadScreen · ScenarioScreen ·       │
│    PlaceholderScreens                               │
└─────────────────┬────────────────────────────────────┘
                  │ compose events / state
┌─────────────────▼────────────────────────────────────┐
│  Navigation                                         │
│  core/navigation/KaleidoNavHost.kt                  │
│    routes: splash · onboarding · home ·             │
│            test · read · scenario                   │
└─────────────────┬────────────────────────────────────┘
                  │
┌─────────────────▼────────────────────────────────────┐
│  Repositories (thin, testable)                      │
│  core/data/profile/ProfileRepository   ← DataStore  │
│  core/data/progress/ProgressRepository ← Room (TBD) │
│  core/data/content/*Repository         ← Assets (TBD)│
└─────────────────┬────────────────────────────────────┘
                  │
┌─────────────────▼────────────────────────────────────┐
│  Persistence                                        │
│  DataStore Preferences (profile)                    │
│  Room database       (quiz, messages, progress)     │
│  Bundled assets      (reviewers, questions, cases)  │
└──────────────────────────────────────────────────────┘
```

**Future AI integration** lives in `ai/AiEngine.kt` (interface only) + `ai/AiAvailability.kt` (enum). Phase 1 has **zero implementations** and must remain that way.

**Separation rules:**
- Compose composables must not contain business logic (validation, scoring, matching). Put those in ViewModels or repository-adjacent pure functions.
- Repositories must not import Compose types.
- ViewModels must not import Room entities directly — expose domain models.
- Scoring rules, correct-answer criteria, and case facts live in structured app data (JSON assets), never in AI output.

---

## 5. File structure (as of this brief's commit)

```
Kaleido/
├── KALEIDO — MASTER DEVELOPMENT PROMPT.txt   ← canonical spec
├── PROJECTBRIEF.md                            ← this file
├── settings.gradle.kts
├── build.gradle.kts
├── gradle/
│   ├── libs.versions.toml                    ← verify versions on first read
│   └── wrapper/gradle-wrapper.properties
├── app/
│   ├── build.gradle.kts                      ← 3 deps: nav-compose, datastore, lifecycle
│   └── src/main/
│       ├── AndroidManifest.xml                ← NO INTERNET permission
│       ├── java/com/kaleido/app/
│       │   ├── MainActivity.kt
│       │   ├── ai/
│       │   │   ├── AiAvailability.kt          ← enum, no logic
│       │   │   └── AiEngine.kt                ← interface only, no impl
│       │   ├── core/
│       │   │   ├── data/
│       │   │   │   ├── AppContainer.kt
│       │   │   │   └── profile/ProfileRepository.kt
│       │   │   └── navigation/KaleidoNavHost.kt
│       │   ├── feature/
│       │   │   ├── splash/SplashScreen.kt
│       │   │   ├── onboarding/OnboardingScreen.kt
│       │   │   ├── home/HomeScreen.kt
│       │   │   └── placeholder/PlaceholderScreens.kt
│       │   └── ui/theme/
│       │       ├── Color.kt                   ← Kaleido palette + wizard defaults
│       │       ├── Theme.kt                   ← dynamicColor=false
│       │       └── Type.kt
│       └── res/
│           ├── values/
│           │   ├── colors.xml                 ← + kaleido_background (#E5E2FA)
│           │   └── themes.xml                 ← windowBackground=@color/kaleido_background
│           └── values-night/colors.xml        ← kaleido_background (#0B1A30)
└── docs/                                      ← NOT YET CREATED
    ├── ARCHITECTURE.md
    ├── SETUP.md
    ├── DECISIONS.md
    └── TESTING.md
```

**Not yet created (do not assume it exists):**
- `app/src/main/assets/reviewers/`, `assets/cases/`, `assets/questions/`
- Room database, entities, DAOs, `ProgressRepository`
- Any ViewModel
- Any unit test file (`app/src/test/`)
- Any instrumentation test file (`app/src/androidTest/`)
- Logo drawable / vector
- Profile editing screen
- Home screen's four subject buttons
- Progress summary on Home

---

## 6. Current state — verified vs unverified

**This section is the most important part of the brief. Trust the right column.**

| Item | Status | Evidence |
|---|---|---|
| 10 Kotlin files created | ✅ Done | Present in repo |
| 3 dependencies added to `app/build.gradle.kts` | ✅ Done | Present in repo |
| Palette in `Color.kt` + `Theme.kt` | ✅ Done | Present in repo |
| `dynamicColor = false` in theme | ✅ Done | Present in repo |
| Splash/Onboarding/Home wrapped in `Surface` with theme background | ⚠️ Unverified — potential syntax error | Last write to `SplashScreen.kt` may end with stray `)`. Check before trusting. |
| `themes.xml` `windowBackground` | ✅ Done | Present in repo |
| `values-night/colors.xml` | ✅ Done | Present in repo |
| Build compiles (`assembleDebug`) | ❌ **NOT VERIFIED** | No real `BUILD SUCCESSFUL in Xs` line has ever been produced by any prior AI. Must be re-run from a shell. |
| App launches and shows correct colors | ❌ Unverified | Depends on build + reinstall |
| Profile persists across app restart | ❌ Unverified | Code looks correct; end-to-end test not performed |
| Airplane-mode core features work | ❌ Unverified | Depends on Phase 1 features being real |

**FIRST ACTION for any AI picking this up:** verify the build state (see §10).

---

## 7. Brand palette (locked — do not change without approval)

Design narrative: **blue → purple → gold** = trust/focus → insight → achievement. Dark navy text. Pale lavender background.

**Material 3 role mapping:**

| Role | Light | Dark |
|---|---|---|
| primary | `#1D52F4` | `#BAC0F7` |
| onPrimary | `#FFFFFF` | `#11327A` |
| primaryContainer | `#BAC0F7` | `#1D52F4` |
| onPrimaryContainer | `#11327A` | `#E5E2FA` |
| secondary | `#614FF0` | `#B8AAFF` |
| onSecondary | `#FFFFFF` | `#0B1A30` |
| secondaryContainer | `#E5E2FA` | `#614FF0` |
| onSecondaryContainer | `#11327A` | `#E5E2FA` |
| tertiary | `#E0B953` | `#E0B953` |
| onTertiary | `#0B1A30` | `#0B1A30` |
| tertiaryContainer | `#F5E8B8` | `#8A6D1F` |
| onTertiaryContainer | `#0B1A30` | `#F5E8B8` |
| background | `#E5E2FA` | `#0B1A30` |
| onBackground | `#0B1A30` | `#E5E2FA` |
| surface | `#FFFFFF` | `#11327A` |
| onSurface | `#0B1A30` | `#E5E2FA` |
| surfaceVariant | `#BAC0F7` | `#1D52F4` |
| onSurfaceVariant | `#11327A` | `#BAC0F7` |
| outline | `#11327A` | `#BAC0F7` |
| error | `#B3261E` | `#F2B8B5` |
| onError | `#FFFFFF` | `#601410` |

`dynamicColor` must remain `false`. Setting it to `true` on Android 12+ overrides these with wallpaper colors.

---

## 8. Roadmap — what to build next, in order

Each increment must be a **complete, testable unit**. Do not start the next increment until the prior one's acceptance tests pass.

### Increment A — STABILIZE (in progress)
- [ ] Real `assembleDebug --rerun-tasks` succeeds with `BUILD SUCCESSFUL in Xs`
- [ ] Splash/Onboarding/Home render with palette (after uninstall + reinstall on emulator)
- [ ] Profile persists across force-stop → reopen
- [ ] Emulator dark mode toggles background to `#0B1A30`

### Increment B — Home + Room foundation
- [ ] 4 subject buttons on Home (Developmental, Assessment, Abnormal, I-O)
- [ ] Progress summary on Home ("0 questions answered", "0 cases completed")
- [ ] Room `KaleidoDatabase` v1 with `QuizAttemptEntity`, `ScenarioMessageEntity`, `CaseProgressEntity`
- [ ] `ProgressRepository`
- [ ] Wire Room into `AppContainer`
- [ ] Unit test: `ProgressRepository` insert/read/clear

### Increment C — Test mode
- [ ] `assets/questions/sample_questions.json` (3–5 MCQs, demo-labeled)
- [ ] `Question` data class + `QuestionRepository` (kotlinx.serialization)
- [ ] `TestViewModel` (current index, selected answer, feedback state, score)
- [ ] `TestScreen` UI (question, 4 options, submit, feedback, next)
- [ ] Save `QuizAttemptEntity` on submit
- [ ] Unit tests: answer evaluation matches key; score tally correct; empty bank handled

### Increment D — Read mode
- [ ] `assets/reviewers/sample_reviewer.json` (4 subjects × sections)
- [ ] `ReviewerRepository`
- [ ] `ReadScreen`: subject list → expandable sections
- [ ] Optional: DataStore for last-read position

### Increment E — Scenario mode
- [ ] `assets/cases/sample_case.json` (1 fictional case: intro, `allowedFacts[]` with `{trigger, response}`, `diagnosisOptions[]`, `correctDiagnosis`, `scoringRules`)
- [ ] `CaseRepository`
- [ ] `ScenarioViewModel` with deterministic trigger-matching (no AI)
- [ ] `ScenarioScreen`: chat UI, submit question, view response, reset
- [ ] Persist `ScenarioMessageEntity` keyed by `caseId`
- [ ] Unit tests: keyword match, diagnosis scoring, reset clears only active case

### Increment F — Polish + docs
- [ ] Logo: static VectorDrawable from `DOCTYPE.html` SVG source, placed in SplashScreen
- [ ] Profile editing flow (edit existing profile)
- [ ] `docs/ARCHITECTURE.md`, `SETUP.md`, `DECISIONS.md`, `TESTING.md`
- [ ] Instrumentation test: Splash → Onboarding → Home → reload skips onboarding
- [ ] Physical device + airplane-mode verification (or mark as unverified honestly)

### Phase 2+ (out of scope)
- Optional on-device AI behind `AiEngine` (Gemini Nano, LiteRT, MediaPipe LLM)
- Must be gated by `AiAvailability.AVAILABLE`
- Must ground responses in `allowedFacts`
- Must be labeled "AI-generated, may be inaccurate"
- Must never determine scoring

---

## 9. Collaboration protocol (multi-AI)

This project intentionally uses **5 AIs** in defined roles. Follow the role assigned to you.

| Role | Suggested AI | Responsibility |
|---|---|---|
| **Architect** | DeepSeek | Plans increments, writes specs, reviews diffs, owns `docs/DECISIONS.md` |
| **Executor** | Gemini (in Android Studio) | Applies file edits, runs terminal commands, reports verbatim output |
| **QA / Reviewer** | Claude or ChatGPT | Reads diffs against master prompt, flags rule violations, writes tests |
| **Documentation** | Any | Keeps `docs/*.md` and this brief current |
| **Human** | — | Approves plans, runs terminal when AIs cannot, pastes output back |

**Increment workflow (no exceptions):**

1. **Human** tells Architect the next increment from §8.
2. **Architect** writes a spec with: files to create/modify/delete, purpose per file, exact contents or patch, verification steps.
3. **Human** pastes spec into Executor.
4. **Executor** reads current files first, posts a plan, **waits for human to say "approve"**.
5. **Executor** applies changes.
6. **Executor** runs verification, pastes **verbatim** output.
7. **Human** pastes Executor output into QA.
8. **QA** approves or rejects. On reject, cycle to step 4.
9. **Documentation** updates `docs/DECISIONS.md` and this brief.
10. **Human** commits with message format `Increment X: <short description>`.

**Before ANY AI modifies code, it must:**
1. Read the current files it intends to change.
2. List exact files to create/modify/delete.
3. Explain purpose of each change in one sentence.
4. Identify assumptions.
5. Ask for approval on anything architecturally significant (new dependency, new pattern, new module, change to AppContainer, change to theme signature).

**Never replace the whole project. Never delete working features without approval. Prefer small changes tested independently.**

---

## 10. Verification protocol — mandatory

No AI is allowed to claim success without evidence. This section exists because prior attempts on this repo produced **four identical stub responses** (`{"status": "Build finished successfully."}`) with **zero real Gradle output**, while simultaneously claiming "Zero compile errors." That pattern is forbidden.

### For build verification, always use a shell, never a "build tool" that returns a stub

**Windows PowerShell (from repo root):**
```powershell
$env:JAVA_HOME = "C:\Program Files\Android\Android Studio\jbr"
$env:PATH = "$env:JAVA_HOME\bin;$env:PATH"
.\gradlew.bat assembleDebug --rerun-tasks
```

**What to paste back:** the last 10 lines of output. The mandatory line is either:
- `BUILD SUCCESSFUL in Xs` — this is proof
- `BUILD FAILED` followed by an `error:` line — this is also proof (of failure), and it must be pasted verbatim

**What is NOT proof:**
- `{"status": "Build finished successfully."}`
- `Build was successful` (as prose)
- `No errors detected` (as prose)
- Any statement not accompanied by `in Xs`

### For unit tests
```powershell
.\gradlew.bat testDebugUnitTest --rerun-tasks
```
Paste the **`BUILD SUCCESSFUL in Xs`** line plus the test count (`X tests completed, Y failed`) if present.

### For instrumentation tests
```powershell
.\gradlew.bat connectedDebugAndroidTest
```
Requires a running emulator or device. If you cannot run it, say: **"Instrumentation test UNVERIFIED — no emulator available in this environment."**

### For UI behavior
The human runs the app on the emulator and pastes:
- Screenshots (optional but preferred)
- Exact behavior observed vs expected

An AI cannot verify UI behavior alone. When in doubt, ask the human to describe what they see.

### Verified vs unverified labeling
Every AI response that claims work was done must end with one of:

- **"VERIFIED: <evidence>"** — e.g., `VERIFIED: BUILD SUCCESSFUL in 47s, tests: 3 passed`
- **"UNVERIFIED: <reason>"** — e.g., `UNVERIFIED: no emulator in this environment; please run <command>`

Never end with "Everything works." Never omit this label.

---

## 11. Known pitfalls (learn from prior failures)

These were observed during earlier attempts on this repo. Every AI must avoid repeating them.

### 11.1 Stray `)` at end of `write_file` content
Two Kotlin files were written ending with `}` followed by a stray `)` on its own line. That is a syntax error. **After every Kotlin file write, re-read the last 5 lines and confirm the file ends with a single `}` (closing the outermost function/class), not `)`.**

### 11.2 `gradle_build` tool returns useless stub
Some in-IDE tool integrations return `{"status": "Build finished successfully."}` without running anything. **Do not use such tools.** Use `run_shell_command` (or ask the human to run the command).

### 11.3 `JAVA_HOME` not set in the shell
Android Studio bundles JDK 17 at `C:\Program Files\Android\Android Studio\jbr`. Shell commands fail with `ERROR: JAVA_HOME is not set` unless you set it first. **Always set `JAVA_HOME` before running Gradle from a shell.**

### 11.4 `analyze_file` returns "IDE indexing in progress"
Right after a batch of file writes, the IDE is in "dumb mode." Wait or skip. Do not interpret "cannot analyze" as "no errors."

### 11.5 Unused `val` warnings in `Color.kt`
Kotlin reports 20+ "never used" warnings for the `Light*` and `Dark*` palette vals. This is **benign** — they ARE used, but only inside the `lightColorScheme(...)` / `darkColorScheme(...)` argument lists in `Theme.kt`. Do not chase these warnings. Do not delete the palette values.

### 11.6 `dynamicColor` parameter warning in `Theme.kt`
The parameter is retained for signature compatibility but not used in the `if (darkTheme)` branch. This is **benign**. Do not remove the parameter — that would break `MainActivity.kt` and any previews.

### 11.7 `android:Theme.Material.Light.NoActionBar` parent in `themes.xml`
For a Compose-only app this is **fine**. Do not "fix" it to `Theme.Material3.DayNight.NoActionBar` unless you have a concrete reason; changing it can affect `windowBackground` resolution and cause the exact "colors not applied" symptom we just fixed.

### 11.8 Emulator "already running as process NNNN"
If Android Studio refuses to launch the emulator with "Medium Phone API XX is already running," do not wipe the AVD. Instead:
1. Device Manager → Stop the emulator, or Task Manager → end `qemu-system-x86_64.exe`
2. Delete `*.lock` files in `C:\Users\<user>\.android\avd\<AVD_NAME>.avd\`
3. Device Manager → ⋮ → **Cold Boot Now**

Never delete the entire `.android` folder — that wipes every AVD.

### 11.9 Stale APK after theme changes
Android Studio may cache. After any change to `Theme.kt`, `themes.xml`, or `colors.xml`:
1. **Uninstall** the app from the emulator (long-press icon → Uninstall), or Device Manager → ⋮ → Wipe Data
2. Run again

An "unchanged" UI is often a stale install, not a code bug.

### 11.10 `splash → onboarding` on every launch means DataStore is not persisting
If force-stop → reopen lands on Onboarding again, DataStore write is failing. Check Logcat for `DataStore` errors. Common causes: `Context` mismatch (using `Activity` instead of `applicationContext`), or app data cleared between runs.

---

## 12. Prompt patterns for the executor (Gemini)

When the human sends a prompt to the Executor, include these in the prompt:

- **"Inspect before editing"** — always
- **"List files to be created/modified/deleted"** — always
- **"Wait for 'approve' before writing"** — for anything non-trivial
- **"Use `run_shell_command`, not `gradle_build`. Set `JAVA_HOME` first."**
- **"Report verbatim `BUILD SUCCESSFUL in Xs` or `error:` line. Do not summarize."**
- **"Re-read the last 5 lines of any file you wrote to confirm it ends with `}` not `)`."**
- **"End with VERIFIED or UNVERIFIED label per §10."**

Sample preamble to include in every Executor prompt:

```
## Rules for this task
1. Inspect first. Read every file you intend to change and report its
   current contents before editing.
2. List every file you will create / modify / delete with a one-sentence
   purpose. Wait for "approve" before writing.
3. Do NOT replace the whole project. Do NOT delete working features.
4. Do NOT add dependencies beyond the locked stack.
5. Do NOT add android.permission.INTERNET or any network call.
6. Verify with `run_shell_command` (not `gradle_build`). Set JAVA_HOME
   to C:\Program Files\Android\Android Studio\jbr before running Gradle.
   Paste the VERBATIM last 10 lines of output. A claim of success
   without a "BUILD SUCCESSFUL in Xs" line is rejected.
7. After writing each Kotlin file, re-read its last 5 lines and confirm
   it ends with `}` not `)`.
8. End your report with exactly one of:
   - "VERIFIED: <evidence>"
   - "UNVERIFIED: <reason>"
```

---

## 13. Definition of "Increment done"

An increment is not done until **all** of these are true:

1. Files are committed and pushed.
2. `.\gradlew.bat assembleDebug --rerun-tasks` shows `BUILD SUCCESSFUL in Xs` (verbatim, from a shell).
3. `.\gradlew.bat testDebugUnitTest --rerun-tasks` shows `BUILD SUCCESSFUL in Xs` (if tests were added).
4. Manual UI check on emulator passed (human confirms).
5. QA AI has reviewed the diff against the master prompt and this brief.
6. `docs/DECISIONS.md` has an entry describing what was done and why.
7. Every claim in the increment summary is labeled VERIFIED or UNVERIFIED.

---

## 14. Immediate next steps for any AI joining this repo

### Step 1 — Read (do not edit)
Read in this order:
1. `KALEIDO — MASTER DEVELOPMENT PROMPT.txt`
2. This brief (you're reading it)
3. `app/build.gradle.kts` and `gradle/libs.versions.toml` (verify versions)
4. Every file in `app/src/main/java/com/kaleido/app/`
5. `app/src/main/AndroidManifest.xml`
6. `app/src/main/res/values/themes.xml` and both `colors.xml` files

Do not assume any feature works. Inspect.

### Step 2 — Verify build state (ask human to run)
Ask the human to run:

```powershell
$env:JAVA_HOME = "C:\Program Files\Android\Android Studio\jbr"
.\gradlew.bat assembleDebug --rerun-tasks
```

and paste the last 10 lines. Based on the result:

- If `BUILD SUCCESSFUL in Xs` → Increment A compiles. Continue to Step 3.
- If `BUILD FAILED ... error: ...` → Fix the compile error (likely a stray `)` in `SplashScreen.kt` per §11.1) before anything else.
- If neither line appears → the build did not actually run. Ask for real terminal output.

### Step 3 — Verify UI behavior (ask human)
Ask the human:
1. Uninstall the app from the emulator, then Run again from Android Studio.
2. Confirm: Splash has pale lavender background (`#E5E2FA`); Onboarding has royal blue Save button; Home has royal blue mode buttons.
3. Force-stop the app, reopen. Should land on **Home**, not Onboarding.
4. Toggle emulator dark mode. Background should become `#0B1A30`.

Paste the answers back. If any check fails, diagnose before proceeding.

### Step 4 — Begin Increment B
Once A is verified, produce the plan for **Increment B** (§8). Do not skip ahead to Increment C.

---

## 15. Reference: prompt template for the human to use with any AI

When the human wants to start a new increment with a fresh AI, use this template:

```
You are joining an existing Android project called Kaleido
(offline psych-review app for Filipino students). Read in order:

1. KALEIDO — MASTER DEVELOPMENT PROMPT.txt (repo root)
2. PROJECTBRIEF.md (repo root)  ← this is your guide

Then inspect the current code. Do NOT assume anything works. Follow
the brief's verification protocol (§10), collaboration protocol (§9),
and known pitfalls (§11) strictly.

Current increment: <Increment letter from §8>
Task: <one-line description from §8 checklist>

Produce a file-by-file plan first. Wait for "approve" before editing.
End with VERIFIED or UNVERIFIED per §10.
```

---

**End of PROJECTBRIEF.md**

_This brief supersedes all chat-history context. If a prior conversation contradicts this file, trust this file. If the code contradicts this file, trust the code and update this file._
```

---

**A note on what I removed vs. what I kept:**

I deliberately dropped the conversational details (Gemini logs, chat history, individual message transcripts) because those don't belong in a repo file — they're noise for a fresh reader. I kept:
- Every constraint that can be *violated* (so they can be enforced)
- Every pitfall that was actually *observed* (so it isn't repeated)
- The verified/unverified table (so nobody trusts stale claims)
- The multi-AI protocol (since you're explicitly running 5 AIs)
- The prompt template (so you don't have to re-invent it each time)

**When you commit this:**
1. Save as `PROJECTBRIEF.md` in the repo root (next to the master prompt `.txt`).
2. Update the `Repository:` line at the top with your GitHub URL.
3. Commit with message: `docs: add PROJECTBRIEF.md for multi-AI onboarding`.
4. Pin it in the GitHub repo (Settings → General → "Pinned" doesn't exist, but you can add it to README as the first link).

Then when you add each new AI to the repo, send them **only** the template from §15 — no chat history needed.