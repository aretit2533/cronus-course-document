# Module 1 Exercises: Introduction & Features

## 🎯 Goals

- Understand what the library provides
- Identify which features you want in your environment

## ✅ Exercise 1: Feature-to-problem mapping

1. Create a small table (in your notes) with columns:
   - Production problem
   - Feature in `CustomServerKafka`
   - How you would validate it works

Examples to include:

- consumer crashes
- missing topics
- memory pressure

## ✅ Exercise 2: Read the code paths

Open the implementation file and answer:

- Where does it load KafkaJS from?
- Which method connects admin?
- Which event triggers consumer recreation?

## 🧩 Challenge

Explain (1–2 paragraphs) why a team might prefer a shared custom strategy over per-service custom code.

## ✔️ Verification

- You can point to the methods: `listen()`, `start()`, `bindEvents()`, `handleMessage()`.
