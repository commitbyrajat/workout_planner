C4Context
    title Clean Tiered Architecture: Atlas AI (Workspace & Optimized Layout - 2026)

    %% ACCESS & PERIMETER
    Person(user, "Data User", "Analyst (On-Prem/VPN)")
    System_Ext(lb, "Cloud Load Balancer", "Google Cloud LB", "L7 Global Traffic Management")

    Boundary(dmz, "DMZ (On-Prem Bank Network)") {
        System(nginx, "Nginx Reverse Proxy", "Nginx Plus", "SSL Termination & WAF")
    }

    %% INTERNAL COMPUTE (GKE + ASM)
    Enterprise_Boundary(k8s, "Internal GKE Cluster (Anthos Service Mesh)") {
        
        Boundary(tier1, "Management Layer") {
            System(ui, "Atlas UI", "Next.js 16", "Multi-Tenant Workspace Portal")
        }

        Boundary(tier2, "Intelligence Layer") {
            System(ai_service, "Atlas AI Service", "Python 3.12", "Tenant-Aware Orchestrator")
            System(litellm_local, "LiteLLM Proxy (ACL)", "Proxy Service", "Governance & Quotas")
        }

        Boundary(tier3, "Semantic Execution Layer") {
            System(engine, "Atlas Engine", "MDL", "Logic Layer")
            System(core, "Atlas Core", "Rust", "Transpiler")
            System(ibis, "Ibis Server", "Unified API", "DB Gateway")
        }
    }

    %% EXTERNAL PLATFORMS & DATA
    Boundary(tier4, "External Platforms & Data") {
        SystemDb_Ext(data_sources, "Enterprise Data", "Postgres / MS SQL Server / BigQuery", "Physical Data")
    }

    %% PERSISTENCE & DATA
    Boundary(ext_storage, "Persistence Layer (GCP Cloud SQL)") {
        SystemDb(ui_db, "Metadata DB", "PostgreSQL 17", "Workspaces & Multi-Tenant MDL")
        SystemDb(vector_db, "Vector DB", "PostgreSQL + pgvector", "Tenant-Isolated Embeddings")
    }

    %% REPOSITIONED: SMALL LOCAL MODEL LAYER
    Boundary(tier_local, "Small Local Model Layer") {
        System(lingua, "LLMLingua-2", "vLLM / ONNX", "Prompt Context Compression")
    }

    Boundary(tier5, "External Platforms & Data") {
        System_Ext(genai_platform, "Enterprise GenAI Platform", "LiteLLM Gateway", "Gemini 2.5 | BGE-M3")
    }

    %% RELATIONSHIPS
    Rel(user, lb, "Request Access", "HTTPS/443")
    Rel(lb, nginx, "Distribute Traffic", "HTTPS/443")
    Rel(nginx, ui, "Proxy Traffic", "ASM Ingress Gateway")
    
    Rel(ui, ui_db, "Save/Load MDL", "mTLS / PSC")
    Rel(ai_service, vector_db, "RAG Retrieval", "pgvector (PSC)")
    Rel(ui, ai_service, "Process NLQ", "mTLS (Internal REST)")

    %% Optimized Intelligence Flow
    Rel(ai_service, lingua, "Compress Prompt Context", "gRPC (mTLS)")
    Rel(ai_service, litellm_local, "Chat Request (Compressed)", "OpenAI API (mTLS)")
    
    %% FIX: Vertical LLM path (Tier 2 to Tier 4)
    Rel_D(litellm_local, genai_platform, "Relay to Global LLM", "HTTPS/Egress Proxy")
    
    %% Semantic Execution Path
    Rel(ai_service, engine, "Send Logic SQL", "MCP over mTLS")
    
    %% FIX: Horizontal Dialect path (Tier 3 Internal)
    Rel_R(engine, core, "Logic -> Dialect", "FFI")
    
    Rel(core, ibis, "Standardized SQL", "API")
    Rel(ibis, data_sources, "Query Data", "Native/PrivateLink")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="3")

