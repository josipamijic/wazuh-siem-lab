# Scenario — NordDesk AB

**NordDesk AB** is a small, fictional company used to give this lab a realistic context.

- Small office, around 20–30 employees.
- A two-person **IT support team** handles day-to-day issues: password resets, new
  starters, laptop problems.
- Until now, each Windows computer kept its own logs locally and **no one was watching
  them centrally**. If someone's account was being attacked, nobody would notice.

**The task:** management asks IT support to start monitoring the company's Windows
computers so they can answer one question — *"How would we know if someone was trying
to break into one of our machines?"*

The answer is to introduce a **SIEM**. This lab builds the first, minimal version of
that: one monitoring server and one monitored Windows endpoint, proving the concept
before it would be rolled out to the whole company.

> This scenario is invented for learning purposes. All machines, names, and data are
> fictional and run in an isolated home lab.
