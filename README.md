# EviCare (Evidence based Care): AI-Powered Clinical Guideline Intelligence

## 📋 Project Overview

EviCare is a high-precision Retrieval-Augmented Generation (RAG) platform designed to provide evidence-backed clinical recommendations. It helps healthcare professionals make informed decisions by semantically matching patient conditions with the latest publicly available clinical guidelines (e.g., ICMR, WHO), providing fully cited, transparent recommendations.

## ⚠️ Problem Statement

Medical practitioners face challenges in keeping up with rapidly evolving guidelines and translating them into patient-specific care. Traditional search and summarization tools are insufficient, and standard LLMs risk hallucination in clinical settings.

The Challenge:
Build a system that delivers accurate, guideline-based recommendations, maintains clinical integrity, and works efficiently on limited infrastructure (e.g., AWS EC2 Free Tier).

## 🛠️ Tech Stack

- Language: Python 3.12
- LLM & Orchestration: Amazon Bedrock, LangChain
- Vector Database: ChromaDB (Persistent)
- API Framework: FastAPI with Uvicorn
- Frontend: Streamlit
- Infrastructure: AWS EC2 (t3.micro), IAM Role-based Security

## 🎯 Objectives

- Evidence-Based Recommendations: Provide patient-specific recommendations strictly grounded in current clinical guidelines.
- Two-Stage Retrieval & Reasoning: Combine vector similarity retrieval with cloud-native reranking and LLM reasoning for precise recommendations.
- Source Transparency: Include citations for all recommendations so clinicians can verify the original guidelines.
- Infrastructure Optimization: Efficient performance on low-resource environments without compromising reliability.


## 📁 Project Structure

```
EviCare-hackathon/
├── README.md         # Project overview and documentation
├── requirements.md   # Stage 1 requirements document
├── design.md         # Stage 1 design document
├── EviCare_IdeaSubmission   # Hackathon Stage 1 slides
├── images/
│   ├── archi_design.png    # System architecture diagram
│   └── data_flow.png       # Data flow diagram
```
