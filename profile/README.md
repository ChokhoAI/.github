# Chokho AI - Intelligent Waste Management & Route Optimization Platform

Chokho is an enterprise-grade, intelligent waste management and monitoring system built for high-traffic urban and suburban areas. The system addresses the critical problem of unmanaged public waste, providing a complete, data-driven accountability loop from citizen complaint to verified cleanup. 

By leveraging modern microservices, artificial intelligence, and geospatial algorithms, Chokho ensures optimized municipal worker dispatches and verifiable civic maintenance.

---

## The Problem

Public waste accumulation on roads in Dehradun and surrounding high-footfall areas (like Haridwar and Rishikesh) is a persistent civic challenge. The core issues are:
1. **Lack of Structured Reporting**: There is no unified mechanism for citizens to report waste reliably.
2. **Inefficient Dispatching**: Municipal workers are dispatched without optimized routing or severity-based prioritization.
3. **No Accountability**: There is no verifiable record of whether reported waste was actually cleaned.

## The Solution

Chokho resolves these issues by dividing the process into two intelligent phases:

1. **Phase 1 — Citizen Reporting and AI Validation**  
   Citizens photograph waste on public roads. The system validates the complaint through a two-stage AI pipeline: YOLOv8n performs high-speed binary trash detection, while Google Gemini extracts metadata (trash type, volume, severity) and filters out indoor images. This prevents fake or duplicate complaints from cluttering the system.

2. **Phase 2 — Route Optimization and Verified Cleanup**  
   The system clusters pending complaints using K-Means and determines the optimal visit order using a nearest-neighbor TSP algorithm. Municipal workers receive optimized routes and must submit a GPS-verified photo upon completion. Gemini AI visually compares the before-and-after images to establish a mathematically and visually proven accountability loop.

---

## High-Level Architecture

The platform operates on a microservices-inspired architecture, ensuring strict separation of concerns, scalability, and optimal performance for both I/O bound and CPU/GPU bound tasks.

```mermaid
graph TD;
    subgraph Client Layer
        CitizenUI["Citizen Dashboard<br/>(React / Next.js)"]
        WorkerUI["Worker Dashboard<br/>(React / Next.js)"]
        AdminUI["Admin Dashboard<br/>(React / Next.js)"]
    end

    subgraph API Gateway / Core Layer
        SpringBoot["Core Backend<br/>(Spring Boot / Java)"]
        Auth["JWT Authentication<br/>& Role Management"]
    end

    subgraph Intelligence Layer
        FastAPI["AI Engine<br/>(FastAPI / Python)"]
        YOLO["YOLOv8n Model<br/>(Binary Detection)"]
        Gemini["Google Gemini AI<br/>(Validation & Metadata)"]
        TSP["Clustering & Routing<br/>(K-Means & TSP)"]
    end

    subgraph Data & Storage Layer
        Postgres[(PostgreSQL + PostGIS)]
        Cloudinary[(Cloudinary Image Storage)]
    end

    CitizenUI -->|RESTful APIs| SpringBoot
    WorkerUI -->|RESTful APIs| SpringBoot
    AdminUI -->|RESTful APIs| SpringBoot
    
    SpringBoot <--> Auth
    SpringBoot <-->|Geo-Queries & CRUD| Postgres
    SpringBoot -->|Image Uploads| Cloudinary
    SpringBoot <-->|Inference & Optimization Requests| FastAPI
    
    FastAPI --> YOLO
    FastAPI --> Gemini
    FastAPI --> TSP
```

### Core Components
1. **Frontend Application**: Built with Next.js 16, React 19, TypeScript, and Tailwind CSS.
2. **Core Backend**: Built with Spring Boot 4 and Java 23. Handles all business logic, data persistence, and user state.
3. **AI Engine**: Built with FastAPI and Python. Specializes in heavy computations, computer vision, and spatial algorithms.
4. **Data Layer**: PostgreSQL extended with PostGIS for advanced geospatial queries. Cloudinary handles robust image hosting.

---

## System Workflows

### 1. Citizen Reporting and AI Validation
When a citizen reports a waste issue, the system employs a two-stage AI pipeline to filter out noise, fake reports, and duplicates.

```mermaid
sequenceDiagram
    participant Citizen
    participant CoreAPI as Spring Boot
    participant AI as FastAPI Engine
    participant DB as PostgreSQL
    
    Citizen->>CoreAPI: Submit Photo + GPS Data
    CoreAPI->>AI: Request Initial Validation
    
    rect rgba(255, 255, 255, 0.1)
        Note over AI: YOLOv8 Validation
        AI-->>AI: Detect Trash Presence (best.pt)
    end
    
    alt No Trash Detected
        AI-->>CoreAPI: Validation Failed
        CoreAPI-->>Citizen: Error: Invalid Image
    else Trash Detected
        rect rgba(255, 255, 255, 0.1)
            Note over AI: Gemini AI Deep Analysis
            AI-->>AI: Extract Type, Volume, Severity
            AI-->>AI: Indoor/Outdoor Classification
        end
        AI-->>CoreAPI: Analysis Metadata
        CoreAPI->>DB: Check ST_DWithin (Duplicates)
        
        alt Is Duplicate / Indoor
            CoreAPI-->>Citizen: Error: Duplicate / Invalid
        else Valid
            CoreAPI->>DB: Persist Complaint
            CoreAPI-->>Citizen: Success (Shows on Heatmap)
        end
    end
```

**Anti-Fake Mechanisms:**
- **Indoor Rejection**: Gemini AI classifies whether a photo was taken indoors to prevent false reporting from inside private properties.
- **Duplicate Prevention**: PostGIS spatial queries check against existing complaints within a specific radius.

---

### 2. Route Optimization and Task Assignment
To optimize municipal efforts, pending verified complaints are grouped and ordered for the municipal fleet.

```mermaid
flowchart TD
    Start([Admin Triggers Optimization]) --> Fetch[Fetch Pending Complaints from DB]
    Fetch --> Cluster[K-Means Clustering based on GPS coordinates]
    Cluster --> Assign[Assign Clusters to Available Vehicles]
    Assign --> TSP[Run Nearest-Neighbor TSP per Cluster]
    TSP --> Generate[Generate Ordered Waypoints]
    Generate --> UpdateDB[Save Routes to DB]
    UpdateDB --> End([Optimization Complete])
```

- **K-Means Clustering**: Groups tasks geographically so each vehicle handles a specific sector.
- **Nearest-Neighbor TSP**: Calculates the shortest path traversing all assigned complaints for a given vehicle.

---

### 3. Verified Cleanup & Accountability
To close the loop, municipal workers must mathematically and visually prove the completion of their tasks.

```mermaid
sequenceDiagram
    participant Worker
    participant CoreAPI as Spring Boot
    participant AI as FastAPI Engine
    participant DB as PostgreSQL
    participant Citizen
    
    Worker->>Worker: Arrive at location
    Worker->>CoreAPI: Submit Verification Photo + GPS
    
    CoreAPI->>DB: Query original complaint GPS
    CoreAPI->>CoreAPI: Calculate GPS Delta (< 20 meters)
    
    alt GPS mismatch
        CoreAPI-->>Worker: Error: You are not at the correct location
    else GPS match
        CoreAPI->>AI: Send Before & After Images
        AI-->>AI: Gemini visual comparison
        
        alt Images do not show cleanup
            AI-->>CoreAPI: Cleanup not verified
            CoreAPI-->>Worker: Error: Area still dirty
        else Verified
            AI-->>CoreAPI: Success
            CoreAPI->>DB: Update Complaint Status (Resolved)
            CoreAPI-->>Worker: Route Stop Completed
            CoreAPI-->>Citizen: Notification: Your report was resolved!
        end
    end
```

---


## Technology Stack Summary

- **Core Backend**: Java 23, Spring Boot 4, Spring Security, Spring Data JPA, Hibernate, JWT.
- **AI Microservice**: Python 3.10+, FastAPI, Pydantic, Uvicorn, Scikit-learn.
- **AI Models**: YOLOv8n (Ultralytics), Google Gemini.
- **Frontend App**: Next.js 16, React 19, TypeScript, Tailwind CSS, Leaflet.js, Radix UI.
- **Datastore & Storage**: PostgreSQL, PostGIS, Cloudinary.
