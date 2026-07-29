# 1. First Mail: AI Outbound Automation Flow

## Overview
This workflow automates the first touchpoint of a Go-To-Market (GTM) outbound sequence. It pulls lead data, verifies deduplication to protect sender reputation, and uses a modular AI prompt chain to generate hyper-personalized emails at scale.

## Workflow Diagram

```mermaid
graph TD
    A[Trigger: Data Event / Schedule] --> B[List Rows Present in Table]
    B --> C[Apply to Each Prospect]
    C --> D[Check Output for Duplicates]
    D --> E{Condition 1: Unique Lead?}
    
    E -- Yes / Continue --> F[Run Prompt: Intro & Body]
    E -- No / Terminate --> G[Skip Record]
    
    F --> H[Run Prompt: Value Proposition]
    H --> I[Run Prompt: Subject Line]
    I --> J[Send an Email V2]
    J --> K[Add Row to Master Tracking Table]
