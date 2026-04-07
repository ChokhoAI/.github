# Chokho AI

Chokho is an intelligent waste management and monitoring system built for Dehradun and the surrounding high-traffic areas of Haridwar and Rishikesh. The name derives from the local word for "clean." The system addresses the problem of unmanaged public waste on roads, providing a complete accountability loop from citizen complaint to verified cleanup.

The project was developed as a two-person academic initiative at UPES, Dehradun, within a one-month timeline, using a microservices architecture that mirrors real-world enterprise patterns.

---

## The Problem

Public waste accumulation on roads in Dehradun and surrounding areas is a persistent civic challenge, worsened during high-footfall seasons. There is no structured mechanism for citizens to report waste, no optimized dispatch system for municipal workers, and no verifiable record of whether reported waste was actually cleaned.

---

## The Solution

Chokho operates in two phases.

**Phase 1 — Citizen Reporting and AI Validation**

A citizen photographs waste on a public road. The system validates the complaint through a two-stage AI pipeline: YOLOv8n performs binary trash detection, and Google Gemini analyzes the image to extract trash type, volume estimate, severity score, indoor or outdoor classification, and location context using the GPS coordinates. Invalid complaints — indoor photos, images with no trash, or duplicate locations — are rejected before reaching the database. Valid complaints appear on a live severity heatmap.

**Phase 2 — Route Optimization and Verified Cleanup**

An administrator triggers route optimization, which clusters pending complaints using K-Means and solves the within-cluster ordering using a nearest-neighbor TSP algorithm. Each of the ten municipal vehicles receives an optimized route. Workers view their assigned stops, navigate to each location, and submit a verification photo. The system validates GPS coordinates against the original complaint location within a 20-meter threshold, then uses Gemini AI to compare before and after images. The citizen is notified upon verified cleanup.

---

## Architecture

The system is split into two backend services with clear separation of responsibilities.

| Service | Technology | Responsibility |
|---|---|---|
| Main Backend | Spring Boot (Java) | Authentication, complaint management, route assignment, worker verification flow, business logic |
| ML Microservice | FastAPI (Python) | YOLOv8n inference, Gemini AI integration, route optimization algorithm |
| Frontend | React (TypeScript) | Citizen, worker, and admin interfaces — role-based single application |
| Database | PostgreSQL + PostGIS | Geospatial queries, complaint storage, route and user management |
| Image Storage | Cloudinary | Complaint images and verification photos |

The two backend services communicate over REST. Spring Boot owns all business logic and database writes; FastAPI handles all ML inference and optimization computation.

---

## Repositories

| Repository | Language | Description |
|---|---|---|
| [chokho-backend](https://github.com/ChokhoAI/chokho-backend) | Java | Spring Boot main service — authentication, complaints, routing, verification |
| [python-backend](https://github.com/ChokhoAI/python-backend) | Python | FastAPI ML service — YOLO, Gemini, route optimization |
| [chokho-frontend](https://github.com/ChokhoAI/chokho-frontend) | TypeScript | React web application — citizen, worker, and admin views |

---

## Key Features

**Anti-Fake Complaint Mechanisms**

Gemini AI classifies whether a photo was taken indoors, detects the absence of trash, and cross-references GPS coordinates against existing complaints to prevent duplicates within a defined radius. Invalid complaints are rejected prior to database insertion.

**AI-Powered Analysis**

Every valid complaint is enriched with structured metadata — trash type (plastic, organic, hazardous, construction), volume estimate (small, medium, large), severity score on a scale of 1 to 10, and location context derived from GPS coordinates without a separate areas database.

**Geospatial Infrastructure**

The database uses PostGIS for all location-sensitive operations: duplicate detection via `ST_DWithin`, GPS verification against original complaint coordinates, and spatial data storage for route optimization input.

**Route Optimization**

Pending complaints are clustered by geographic proximity using K-Means and assigned to vehicles. Within each cluster, stop ordering is solved using a nearest-neighbor TSP heuristic. The algorithm runs on demand via an admin-triggered endpoint.

**Verification Accountability**

Worker verification requires a GPS coordinate match within 20 meters of the original complaint and passes before and after images to Gemini for comparison. This creates an auditable record from submission through resolution.

**Public Heatmap**

A publicly accessible severity heatmap displays complaint density and intensity across the city without requiring authentication, providing civic transparency.

---

## Tech Stack

- **Spring Boot** — Main service, Spring Security, Spring Data JPA, Hibernate, JWT (jjwt 0.12.x)
- **FastAPI** — ML microservice, Pydantic, Uvicorn
- **YOLOv8n** — Binary trash detection (Ultralytics)
- **Google Gemini** — Image analysis, severity scoring, verification comparison (`gemini-2.0-flash`)
- **PostgreSQL + PostGIS** — Primary database with geospatial extension
- **React + TypeScript** — Frontend, Leaflet.js for maps, Tailwind CSS
- **Cloudinary** — Image storage and retrieval
- **Scikit-learn** — K-Means clustering for route optimization

---

## Team

Developed at the University of Petroleum and Energy Studies, Dehradun as a third-year B.Tech Computer Science (AI/ML) project.

| Member | Responsibility |
|---|---|
| Mann | Spring Boot backend, FastAPI ML service, system architecture |
| Collaborator | Frontend integration, Gemini integration support |

---

## Status

Core features implemented and validated: complaint submission with AI validation, route optimization with geographic clustering, worker route assignment, and GPS-verified cleanup confirmation. Frontend development in progress.
