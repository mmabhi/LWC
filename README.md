# ⚡ Lightning Web Components (LWC) — Complete Study Guide

> **Your one-stop resource for mastering Salesforce LWC — from zero to interview-ready.**

---

## 📋 Table of Contents

| Section | Description | Link |
|---------|-------------|------|
| 🚀 **3-Day Crash Course** | Intensive plan for quick learners | [Go →](./3-day-plan/README.md) |
| 📚 **6-Week Comprehensive Plan** | Deep-dive covering all aspects | [./comprehensive-plan/README.md](./comprehensive-plan/README.md) |
| 💼 **Interview Preparation** | Top questions, scenarios & coding challenges | [Go →](./interview-prep/README.md) |
| 📝 **Cheat Sheets** | Quick reference cards | [Go →](./cheat-sheets/README.md) |
| 💻 **Code Examples** | Hands-on projects & snippets | [Go →](./code-examples/README.md) |

---

## 🗺️ Study Plan Overview

### Plan 1: ⏱️ 3-Day Intensive Crash Course

> Best for: Developers with JavaScript/HTML experience who need to get productive fast.

```
┌─────────────────────────────────────────────────────────┐
│                   3-DAY CRASH COURSE                     │
├──────────┬──────────────────┬────────────────────────────┤
│  DAY 1   │     DAY 2        │         DAY 3              │
│          │                  │                            │
│ Basics & │  Data & Events   │  Advanced Topics &         │
│ Setup    │  Communication   │  Interview Prep            │
│          │                  │                            │
│ • What   │ • Wire Service   │ • Navigation               │
│   is LWC │ • Apex Calls     │ • Performance              │
│ • Setup  │ • Parent-Child   │ • Testing                  │
│ • First  │   Events         │ • Lightning Data           │
│   Comp.  │ • Pub-Sub        │   Service                  │
│ • Data   │ • Custom Events  │ • Best Practices           │
│   Binding│ • Forms          │ • Mock Interview            │
│ • Decor- │ • SLDS           │                            │
│   ators  │                  │                            │
└──────────┴──────────────────┴────────────────────────────┘
```

| Day | Topics | Time | Practice |
|-----|--------|------|----------|
| **Day 1** | LWC Fundamentals, Setup, Component Structure, Decorators, Data Binding | ~8 hrs | 15 questions + quiz |
| **Day 2** | Wire Service, Apex Integration, Events, Communication, SLDS | ~8 hrs | 15 questions + quiz |
| **Day 3** | Navigation, Testing, LDS, Performance, Best Practices, Mock Interview | ~8 hrs | 15 questions + quiz + mock interview |

---

### Plan 2: 📅 6-Week Comprehensive Plan

> Best for: Beginners or those preparing for Salesforce Platform Developer certifications.

```
┌───────────────────────────────────────────────────────────────────────┐
│                     6-WEEK COMPREHENSIVE PLAN                        │
├─────────┬─────────┬─────────┬─────────┬──────────┬──────────────────┤
│ Week 1  │ Week 2  │ Week 3  │ Week 4  │ Week 5   │ Week 6           │
│         │         │         │         │          │                  │
│ Foun-   │ Core    │ Data &  │ Adv.    │ Testing  │ Mastery &        │
│ dations │ Conc.   │ Integ.  │ Topics  │ & Deploy │ Interview        │
│         │         │         │         │          │                  │
│ HTML     │Events  │ Wire    │ LDS     │ Jest     │ System Design    │
│ CSS      │Slots   │ Apex    │ Nav     │ CI/CD    │ Mock Interviews  │
│ JS ES6+  │Compos. │ Forms   │ Perf    │ Package  │ Coding Rounds    │
│ Setup    │SLDS    │ Error   │ Custom  │ Debug    │ Scenario-based   │
│ First LWC│Iterate │ Handle  │ Types   │ Security │ HR + Behavioral  │
└─────────┴─────────┴─────────┴─────────┴──────────┴──────────────────┘
```

| Week | Focus Area | Hours/Week | Practice |
|------|-----------|------------|----------|
| **Week 1** | Web Foundations + LWC Setup | 10-12 hrs | 20 questions + project |
| **Week 2** | Core LWC Concepts | 10-12 hrs | 20 questions + project |
| **Week 3** | Data & Server Integration | 10-12 hrs | 20 questions + project |
| **Week 4** | Advanced LWC Features | 10-12 hrs | 20 questions + project |
| **Week 5** | Testing, Debugging & Deployment | 10-12 hrs | 20 questions + project |
| **Week 6** | Mastery & Interview Preparation | 10-12 hrs | 50 questions + mock interviews |

---

## 🧭 How to Use This Guide

1. **Choose your plan** based on your timeline and experience level
2. **Follow the daily/weekly modules** in order — each builds on the previous
3. **Complete ALL practice questions** and quizzes before moving on
4. **Build the mini-projects** — hands-on coding is essential
5. **Use the cheat sheets** for quick revision before interviews
6. **Practice with the interview prep section** for mock interview rounds

---

## 🛠️ Prerequisites

| Requirement | Details |
|-------------|---------|
| **Salesforce Org** | Free [Developer Edition](https://developer.salesforce.com/signup) or Trailhead Playground |
| **VS Code** | With [Salesforce Extension Pack](https://marketplace.visualstudio.com/items?itemName=salesforce.salesforcedx-vscode) |
| **Salesforce CLI** | Install via `npm install @salesforce/cli --global` |
| **Node.js** | v18+ (LTS recommended) |
| **Git** | For version control |
| **Browser** | Chrome with Salesforce Inspector extension |

---

## 📂 Folder Structure

```
LWC-Study-Guide/
├── README.md                          ← You are here
├── 3-day-plan/
│   ├── README.md                      ← 3-day overview & schedule
│   ├── day-1-fundamentals/
│   │   ├── README.md                  ← Day 1 study material
│   │   ├── practice-questions.md      ← 15 practice questions
│   │   └── quiz.md                    ← Day 1 quiz (10 questions)
│   ├── day-2-data-and-events/
│   │   ├── README.md                  ← Day 2 study material
│   │   ├── practice-questions.md      ← 15 practice questions
│   │   └── quiz.md                    ← Day 2 quiz (10 questions)
│   └── day-3-advanced-and-interview/
│       ├── README.md                  ← Day 3 study material
│       ├── practice-questions.md      ← 15 practice questions
│       └── quiz.md                    ← Day 3 quiz (10 questions)
├── comprehensive-plan/
│   ├── README.md                      ← 6-week overview & schedule
│   ├── week-1-foundations/
│   ├── week-2-core-concepts/
│   ├── week-3-data-integration/
│   ├── week-4-advanced/
│   ├── week-5-testing-deployment/
│   └── week-6-mastery-interview/
├── interview-prep/
│   ├── README.md                      ← Interview guide overview
│   ├── top-50-questions.md            ← Most asked LWC questions
│   ├── coding-challenges.md           ← Hands-on coding problems
│   ├── scenario-based-questions.md    ← Real-world scenarios
│   └── behavioral-questions.md        ← HR & behavioral round prep
├── cheat-sheets/
│   ├── README.md                      ← Cheat sheets index
│   ├── lwc-syntax.md                  ← Complete syntax reference
│   ├── lifecycle-hooks.md             ← Lifecycle hooks deep-dive
│   ├── decorators.md                  ← @api, @track, @wire reference
│   └── wire-adapters.md              ← All wire adapters catalog
└── code-examples/
    ├── README.md                      ← Examples index
    ├── 01-hello-world/                ← Basic component
    ├── 02-data-binding/               ← Reactive properties
    ├── 03-conditional-rendering/      ← if:true, template loops
    ├── 04-event-handling/             ← Custom events
    ├── 05-wire-service/               ← Wire service patterns
    ├── 06-imperative-apex/            ← Imperative Apex calls
    ├── 07-navigation/                 ← NavigationMixin
    ├── 08-parent-child/               ← Component communication
    ├── 09-lightning-data-service/     ← LDS patterns
    └── 10-datatable/                  ← lightning-datatable
```

---

> [!TIP]
> **Bookmark this page!** Use it as your home base throughout your LWC learning journey.

---

*Created on June 10, 2026 • Last updated: June 10, 2026*
