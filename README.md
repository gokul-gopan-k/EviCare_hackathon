# EviCare: AI-Powered Clinical Guideline Intelligence

📋 Project Overview

EviCare is a high-precision Retrieval-Augmented Generation (RAG) platform engineered for the healthcare sector. It enables medical professionals to navigate thousands of pages of clinical guidelines (such as ICMR and WHO) to find evidence-based answers with 100% source attribution. By utilizing a two-stage retrieval pipeline, EviCare provides targeted access to specific protocol details, balancing the speed of vector search with the contextual intelligence of cloud-native reranking.

⚠️ Problem Statement

Medical practitioners face "information overload" when navigating unstructured clinical PDFs. Traditional keyword-based search fails to grasp medical semantics, and standard LLMs often hallucinate—a critical risk in clinical settings.

The Challenge: Build a high-fidelity system that extracts contextually accurate protocol data from medical guidelines while maintaining a low memory footprint for deployment on resource-constrained environments (AWS EC2 Free Tier).

🛠️ Tech Stack

- Language: Python 3.12
- LLM & Orchestration: Amazon Bedrock, LangChain
- Vector Database: ChromaDB (Persistent)
- API Framework: FastAPI with Uvicorn
- Frontend: Streamlit
- Infrastructure: AWS EC2 (t3.micro), IAM Role-based Security

🎯 Objectives

- Mitigate Information Overload: Enable instantaneous access to specific clinical protocols within massive documents, allowing practitioners to find exact answers without manual page-by-page searching.
- Two-Stage Semantic Precision: Implement a dual-retrieval strategy—using vector similarity for broad recall and Cloud-Native Reranking for final precision—ensuring that the specific text chunk provided to the LLM is the most clinically relevant.
- Infrastructure Optimization: Design a custom ingestion and retrieval engine optimized for low-resource environments, achieving production-grade performance on an AWS t3.micro instance.
- Preserve Clinical Integrity (Source Transparency): Eliminate "black box" behavior by providing strict source attribution, linking every generated response back to the original text chunk to ensure the clinician can verify the raw data.
