**文档版本**: v2.0  
**日期**: 2026-04-19  
**目标部门**: Digital Security Platform (DSP), ~200 人  
**文档分类**: Internal  
**作者**: Enterprise Architecture Agent

---

## 目录

1. [Executive Summary](https://openwebui.adrian6476.top:8443/c/1d52f966-0e1e-4477-9685-c1c15b5d2ad7#1-executive-summary)
2. [业务愿景与使用场景](https://openwebui.adrian6476.top:8443/c/1d52f966-0e1e-4477-9685-c1c15b5d2ad7#2-%E4%B8%9A%E5%8A%A1%E6%84%BF%E6%99%AF%E4%B8%8E%E4%BD%BF%E7%94%A8%E5%9C%BA%E6%99%AF)
3. [架构设计原则](https://openwebui.adrian6476.top:8443/c/1d52f966-0e1e-4477-9685-c1c15b5d2ad7#3-%E6%9E%B6%E6%9E%84%E8%AE%BE%E8%AE%A1%E5%8E%9F%E5%88%99)
4. [总体架构](https://openwebui.adrian6476.top:8443/c/1d52f966-0e1e-4477-9685-c1c15b5d2ad7#4-%E6%80%BB%E4%BD%93%E6%9E%B6%E6%9E%84)
5. [Ingestion Pipeline 详细设计](https://openwebui.adrian6476.top:8443/c/1d52f966-0e1e-4477-9685-c1c15b5d2ad7#5-ingestion-pipeline-%E8%AF%A6%E7%BB%86%E8%AE%BE%E8%AE%A1)
6. [存储层设计](https://openwebui.adrian6476.top:8443/c/1d52f966-0e1e-4477-9685-c1c15b5d2ad7#6-%E5%AD%98%E5%82%A8%E5%B1%82%E8%AE%BE%E8%AE%A1)
7. [查询与编排层设计](https://openwebui.adrian6476.top:8443/c/1d52f966-0e1e-4477-9685-c1c15b5d2ad7#7-%E6%9F%A5%E8%AF%A2%E4%B8%8E%E7%BC%96%E6%8E%92%E5%B1%82%E8%AE%BE%E8%AE%A1)
8. [知识库作为基础设施：多场景支撑架构](https://openwebui.adrian6476.top:8443/c/1d52f966-0e1e-4477-9685-c1c15b5d2ad7#8-%E7%9F%A5%E8%AF%86%E5%BA%93%E4%BD%9C%E4%B8%BA%E5%9F%BA%E7%A1%80%E8%AE%BE%E6%96%BD%E5%A4%9A%E5%9C%BA%E6%99%AF%E6%94%AF%E6%92%91%E6%9E%B6%E6%9E%84)
9. [安全、合规与治理](https://openwebui.adrian6476.top:8443/c/1d52f966-0e1e-4477-9685-c1c15b5d2ad7#9-%E5%AE%89%E5%85%A8%E5%90%88%E8%A7%84%E4%B8%8E%E6%B2%BB%E7%90%86)
10. [可观测性与运维](https://openwebui.adrian6476.top:8443/c/1d52f966-0e1e-4477-9685-c1c15b5d2ad7#10-%E5%8F%AF%E8%A7%82%E6%B5%8B%E6%80%A7%E4%B8%8E%E8%BF%90%E7%BB%B4)
11. [技术栈总览](https://openwebui.adrian6476.top:8443/c/1d52f966-0e1e-4477-9685-c1c15b5d2ad7#11-%E6%8A%80%E6%9C%AF%E6%A0%88%E6%80%BB%E8%A7%88)
12. [成本拆分](https://openwebui.adrian6476.top:8443/c/1d52f966-0e1e-4477-9685-c1c15b5d2ad7#12-%E6%88%90%E6%9C%AC%E6%8B%86%E5%88%86)
13. [维护策略](https://openwebui.adrian6476.top:8443/c/1d52f966-0e1e-4477-9685-c1c15b5d2ad7#13-%E7%BB%B4%E6%8A%A4%E7%AD%96%E7%95%A5)
14. [实施路线图](https://openwebui.adrian6476.top:8443/c/1d52f966-0e1e-4477-9685-c1c15b5d2ad7#14-%E5%AE%9E%E6%96%BD%E8%B7%AF%E7%BA%BF%E5%9B%BE)
15. [风险评估与缓解](https://openwebui.adrian6476.top:8443/c/1d52f966-0e1e-4477-9685-c1c15b5d2ad7#15-%E9%A3%8E%E9%99%A9%E8%AF%84%E4%BC%B0%E4%B8%8E%E7%BC%93%E8%A7%A3)
16. [未来演进路线](https://openwebui.adrian6476.top:8443/c/1d52f966-0e1e-4477-9685-c1c15b5d2ad7#16-%E6%9C%AA%E6%9D%A5%E6%BC%94%E8%BF%9B%E8%B7%AF%E7%BA%BF)

---

## 1. Executive Summary

### 1.1 项目定位

本方案为 HSBC Digital Security Platform (DSP) 部门设计一套**企业级知识库基础设施（Knowledge Infrastructure Platform, KIP）**。KIP 不是一个单一应用，而是一个**可复用的平台层**——任何需要基于 DSP 代码、文档、运营知识进行智能问答、分析、生成的场景，都可以通过统一的 API 接入 KIP，而无需重复建设 RAG 管线。

### 1.2 核心目标

|目标|描述|
|---|---|
|**平台化**|统一的知识 Ingestion、存储、检索、推理基础设施，支撑多场景（Production Support 仅为第一个场景）|
|**可扩展**|新场景接入只需定义 Prompt Template + 检索策略，无需改动基础设施|
|**合规**|满足 FCA/PRA、SOX 审计要求，Confidential 数据全程加密、可审计|
|**低维护**|全 AWS Managed Services，DevOps 团队兼职 ≤10% 时间可运维|
|**成本可控**|目标 ≤ $40K/月（含 $10K 缓冲）|

### 1.3 关键设计决策摘要

| 决策        | 选择                                                | 理由                                             |
| --------- | ------------------------------------------------- | ---------------------------------------------- |
| 编排框架      | **LlamaIndex**                                    | 企业级成熟度、原生 Bedrock/Neptune/pgvector 集成、代码解析原生支持 |
| 检索策略      | **Hybrid GraphRAG**（向量 + BM25 + 知识图谱三路融合）         | 继承 LightRAG PoC 精髓，适配代码+文档双模态                  |
| LLM       | **Claude Opus 4 / Sonnet 4 / Haiku 4** on Bedrock | 代码理解业界顶级，智能路由控制成本                              |
| Embedding | **Cohere Embed v4** (Text + Image)                | 多模态支持架构图，质量优于 Titan v2                         |
| 图数据库      | **Amazon Neptune Serverless**                     | 代码调用图、依赖图、知识关系图                                |
| 向量数据库     | **Aurora PostgreSQL Serverless v2 + pgvector**    | 已批准、Serverless 自动伸缩、元数据+向量一体                   |
| 全文检索      | **OpenSearch Serverless**                         | BM25 精确匹配类名/方法名/异常名                            |

---

## 2. 业务愿景与使用场景

### 2.1 KIP 的平台定位

```mermaid
graph TB
    subgraph "知识库基础设施 KIP (Platform Layer)"
        direction TB
        ING[Ingestion Pipeline]
        STORE[Unified Storage<br/>Vector + Graph + BM25 + Object]
        RET[Hybrid Retrieval Engine]
        LLM_LAYER[LLM Orchestration<br/>Router + Guardrails + Citation]
        API[Knowledge API<br/>统一 REST/gRPC 接口]
    end

    subgraph "场景层 (Applications) — 按需接入"
        S1[🔥 Production Support<br/>Stacktrace 分析]
        S2[🔍 Code Discovery<br/>代码搜索与理解]
        S3[📝 Onboarding Assistant<br/>新人入职引导]
        S4[📋 Change Impact Analysis<br/>变更影响评估]
        S5[🛡️ Security Review<br/>安全合规审查]
        S6[📊 Incident RCA<br/>事件根因分析]
        S7[🤖 CI/CD Copilot<br/>Pipeline 故障诊断]
        S8[📖 Documentation Gen<br/>文档自动生成]
        S9[❓ 未来场景 N...]
    end

    S1 --> API
    S2 --> API
    S3 --> API
    S4 --> API
    S5 --> API
    S6 --> API
    S7 --> API
    S8 --> API
    S9 --> API
    API --> LLM_LAYER --> RET --> STORE
    ING --> STORE
```

### 2.2 场景详细说明

|场景|描述|知识源|优先级|
|---|---|---|---|
|**Production Support**|输入 Stacktrace，输出根因分析 + 修复建议 + 历史相似事件|代码 + 图谱 + Confluence Runbooks|🔴 P0 (PoC 已验证)|
|**Code Discovery**|"支付服务是如何验证 IBAN 的？" "哪些服务依赖 Redis？"|代码 + 图谱|🔴 P0|
|**Onboarding Assistant**|新人问 "DSP 的整体架构是什么？" "我负责的模块有哪些上下游？"|Confluence + 代码 + 架构图|🟡 P1|
|**Change Impact Analysis**|"如果我修改 `AuthService.validateToken()`，会影响哪些服务？"|图谱 (CALLS/DEPENDS_ON)|🟡 P1|
|**Security Review**|"这个 API 是否有输入校验？" "这段代码是否符合 OWASP Top 10？"|代码 + 合规文档|🟡 P1|
|**Incident RCA**|输入 JIRA Incident ID + 时间窗口，关联代码变更 + 部署记录|图谱 (Commit-modifies-Method) + 部署清单|🟠 P2|
|**CI/CD Copilot**|"这个 Pipeline 为什么失败了？" 输入 build log|Pipeline 代码 + 历史 build 模式|🟠 P2|
|**Documentation Gen**|根据代码自动生成/更新 API 文档、架构文档|代码 + 现有文档（避免重复）|🟠 P2|

### 2.3 场景接入模型

每个新场景接入 KIP 只需提供**三样东西**：

```yaml
# 场景配置示例：production_support.yaml
scene:
  name: "production_support"
  description: "Stacktrace 分析与根因定位"
  
  retrieval_strategy:
    primary: "graph"          # Neptune: 调用链展开
    secondary: "bm25"         # OpenSearch: 精确匹配异常类名
    tertiary: "vector"        # pgvector: 语义相似的历史异常
    fusion: "reciprocal_rank" # RRF 融合
    rerank: true
    top_k: 8

  llm_routing:
    complexity_threshold: 0.7  # Haiku 评分 > 0.7 → Opus 4
    default_model: "sonnet-4"
    complex_model: "opus-4"

  prompt_template: |
    你是 HSBC DSP 部门的高级 Production Support 专家。
    基于以下代码上下文和调用关系图，分析此 Stacktrace 的根因。
    
    ## 规则
    1. 每个事实性陈述必须附引用 [repo/file:line@commitSHA]
    2. 如果无法确定根因，明确说明并建议排查方向
    3. 优先检查最近 30 天的代码变更
    ...

  guardrails:
    require_citations: true
    max_hallucination_rate: 0.20
    pii_detection: true
    
  rbac:
    inherit_github_permissions: true
```

> **新场景接入成本**：一个 YAML 配置 + 可选的自定义 Prompt，**无需修改基础设施代码**。

---

## 3. 架构设计原则

|原则|具体要求|
|---|---|
|**Platform-First**|KIP 是基础设施，非应用。所有场景通过 API + 配置接入，非硬编码|
|**Separation of Concerns**|Ingestion、Storage、Retrieval、Reasoning 四层解耦，各自独立演进|
|**Security by Default**|Confidential 数据全程 KMS CMK 加密（静态+传输）；所有网络走 VPC PrivateLink|
|**Zero-Trust**|每次 API 调用验证 JWT + RBAC；无隐式信任|
|**Audit Everything**|每次查询、每次数据变更、每次权限同步——全部可审计，保留 7 年|
|**Cost-Aware**|智能路由减少 Opus 4 调用；Serverless 组件自动降档；无闲置资源|
|**Graceful Degradation**|图数据库不可用 → 退化为 Vector+BM25；Reranker 不可用 → 退化为 RRF 直出|
|**GitOps**|全部基础设施 Terraform 管理；场景配置存 Git，变更走 PR Review|

---

## 4. 总体架构

### 4.1 总体架构图

```mermaid
graph TB
    subgraph "用户与身份"
        U[👤 DSP 工程师 ~200]
        IDP[Microsoft Entra ID<br/>SAML/OIDC SSO]
        U -->|SSO| IDP
    end

    subgraph "数据源"
        GH[GitHub Enterprise<br/>Java / Go / React / Svelte<br/>IaC / CI-CD Pipelines<br/>DB Schema / Stored Procs]
        CF[Confluence<br/>架构文档 / 合规文档<br/>Runbooks / 设计评审]
    end

    subgraph "Ingestion Pipeline【AWS 私有 VPC】"
        SF[AWS Step Functions<br/>每日 02:00 UTC 调度<br/>+ 手动触发 API]
        
        subgraph "Connectors"
            GHC[GitHub Connector<br/>增量 commit diff<br/>REST API v3]
            CFC[Confluence Connector<br/>CQL 增量拉取<br/>REST API v2]
        end
        
        subgraph "Parsers"
            AST[CodeHierarchyNodeParser<br/>Tree-sitter AST<br/>Java/Go/JS/Svelte]
            MDP[MarkdownNodeParser<br/>+ Image Extractor]
        end
        
        subgraph "Enrichment"
            CGE[CodeGraphExtractor<br/>Haiku 4 → Entity-Relation<br/>借鉴 LightRAG]
            META[Metadata Enricher<br/>repo/file/SHA/authors<br/>classification/version]
        end
        
        EMB[Cohere Embed v4<br/>Text + Image<br/>via Bedrock VPC Endpoint]
    end

    subgraph "存储层"
        PG[(Aurora PostgreSQL<br/>Serverless v2<br/>pgvector HNSW<br/>+ 元数据)]
        NEP[(Neptune Serverless<br/>openCypher<br/>代码知识图谱)]
        OS[(OpenSearch Serverless<br/>BM25 全文索引)]
        S3[(S3 + KMS CMK<br/>原始内容<br/>版本快照<br/>Object Lock)]
        DDB_PERM[(DynamoDB<br/>权限缓存<br/>User → Repos)]
    end

    subgraph "查询与编排层【LlamaIndex on ECS Fargate】"
        APIGW[API Gateway<br/>+ WAF + 限流]
        AUTH[JWT Authorizer<br/>Entra ID 验证<br/>+ RBAC 注入]
        
        subgraph "Scene Router"
            SR[SceneRouter<br/>根据 scene_id<br/>加载 YAML 配置]
        end
        
        subgraph "LLM Router"
            HR[Haiku 4<br/>意图分类<br/>复杂度评估]
        end
        
        subgraph "Retrieval Engine"
            VR[VectorRetriever<br/>→ pgvector]
            BR[BM25Retriever<br/>→ OpenSearch]
            GR[GraphRetriever<br/>→ Neptune]
            RRF[RRF 融合]
            RK[Cohere Rerank 3.5]
        end
        
        subgraph "Reasoning"
            OP4[Claude Opus 4<br/>复杂推理]
            SN4[Claude Sonnet 4<br/>日常查询]
            GD[Guardrails<br/>引用校验<br/>PII 检测<br/>Grounding Check]
            CIT[Citation Extractor<br/>溯源链接生成]
        end
    end

    subgraph "治理与可观测"
        CT[CloudTrail Org Trail<br/>7 年留存 → S3 Glacier]
        CW[CloudWatch<br/>Metrics + Logs]
        OT[OpenTelemetry<br/>→ X-Ray 分布式追踪]
        DDB_AUDIT[(DynamoDB<br/>查询审计日志)]
        DDB_FB[(DynamoDB<br/>用户反馈<br/>👍👎 + 评论)]
        EVAL[评估管线<br/>Ragas + Golden Set<br/>每周 Step Functions]
        BUDGET[AWS Budgets<br/>Cost Anomaly Detection]
    end

    %% 数据流
    IDP -->|JWT| APIGW
    GH --> SF
    CF --> SF
    SF --> GHC --> AST
    SF --> CFC --> MDP
    AST --> META --> EMB
    MDP --> META --> EMB
    AST --> CGE
    EMB --> PG
    CGE --> NEP
    AST --> OS
    MDP --> OS
    META --> S3

    %% 查询流
    APIGW --> AUTH --> SR
    SR --> HR
    HR --> VR
    HR --> BR
    HR --> GR
    VR --> RRF
    BR --> RRF
    GR --> RRF
    RRF --> RK
    RK --> OP4
    RK --> SN4
    OP4 --> GD
    SN4 --> GD
    GD --> CIT --> APIGW

    %% 治理
    AUTH --> DDB_PERM
    APIGW --> DDB_AUDIT
    CIT --> DDB_FB
    APIGW --> CT
    SR --> CW
    SR --> OT
    DDB_FB --> EVAL
    EVAL --> CW
```

### 4.2 网络拓扑

```mermaid
graph TB
    subgraph "AWS Account: dsp-kip-prod"
        subgraph "VPC: 10.0.0.0/16"
            subgraph "Private Subnet A (AZ-a)"
                ECS_A[ECS Fargate<br/>RAG Orchestrator]
                ING_A[ECS Fargate<br/>Ingestion Workers]
            end
            subgraph "Private Subnet B (AZ-b)"
                ECS_B[ECS Fargate<br/>RAG Orchestrator]
                ING_B[ECS Fargate<br/>Ingestion Workers]
            end
            subgraph "Data Subnet"
                PG_DB[(Aurora PostgreSQL<br/>Multi-AZ)]
            end
            
            VPE_BR[VPC Endpoint<br/>Bedrock Runtime]
            VPE_S3[VPC Endpoint<br/>S3 Gateway]
            VPE_OS[VPC Endpoint<br/>OpenSearch]
            VPE_NEP[VPC Endpoint<br/>Neptune]
            VPE_DDB[VPC Endpoint<br/>DynamoDB]
            VPE_SM[VPC Endpoint<br/>Secrets Manager]
        end
        
        APIGW_PUB[API Gateway<br/>Regional Endpoint<br/>+ WAF]
    end

    subgraph "HSBC Internal Network"
        CORP[企业网络<br/>Direct Connect / VPN]
    end

    CORP -->|Private| APIGW_PUB
    ECS_A --> VPE_BR
    ECS_A --> VPE_S3
    ECS_A --> VPE_OS
    ECS_A --> VPE_NEP
```

> ⚠️ **关键安全设计**：所有服务间通信走 VPC Endpoint（PrivateLink），**无流量经过公网**。API Gateway 通过 HSBC Direct Connect / VPN 接入企业内网。

---

## 5. Ingestion Pipeline 详细设计

### 5.1 整体流程

```mermaid
flowchart TD
    START[Step Functions<br/>每日 02:00 UTC] --> P1{并行}
    
    P1 --> GH_STEP[GitHub 增量拉取]
    P1 --> CF_STEP[Confluence 增量拉取]
    
    GH_STEP --> GH_DIFF[获取上次同步后<br/>所有 commit diff]
    GH_DIFF --> GH_CLASSIFY{文件类型分类}
    GH_CLASSIFY -->|.java .go .js .svelte| CODE_PARSE[AST 代码解析<br/>Tree-sitter]
    GH_CLASSIFY -->|.tf .yaml .hcl| IAC_PARSE[IaC 解析<br/>结构化提取]
    GH_CLASSIFY -->|Jenkinsfile .github/| CICD_PARSE[CI/CD 解析<br/>Stage/Step 提取]
    GH_CLASSIFY -->|.sql .ddl| DB_PARSE[Schema 解析<br/>表/列/关系提取]
    GH_CLASSIFY -->|.md .rst| DOC_PARSE[文档解析]
    
    CF_STEP --> CF_CQL[CQL: lastModified > last_sync]
    CF_CQL --> CF_PARSE[Confluence 页面解析<br/>Markdown + Image 提取]
    
    CODE_PARSE --> ENRICH[元数据注入<br/>repo/file/SHA/authors<br/>version/classification]
    IAC_PARSE --> ENRICH
    CICD_PARSE --> ENRICH
    DB_PARSE --> ENRICH
    DOC_PARSE --> ENRICH
    CF_PARSE --> ENRICH
    
    ENRICH --> P2{并行}
    P2 --> EMB_STEP[Cohere Embed v4<br/>Text + Image]
    P2 --> GRAPH_STEP[CodeGraphExtractor<br/>Haiku 4 实体关系抽取]
    P2 --> BM25_STEP[OpenSearch 索引]
    P2 --> S3_STEP[S3 原始内容存储]
    
    EMB_STEP --> PG_WRITE[(pgvector 写入)]
    GRAPH_STEP --> NEP_WRITE[(Neptune 写入)]
    BM25_STEP --> OS_WRITE[(OpenSearch 写入)]
    
    PG_WRITE --> VALIDATE[校验完整性<br/>chunk count 比对]
    NEP_WRITE --> VALIDATE
    OS_WRITE --> VALIDATE
    VALIDATE --> NOTIFY[Slack 通知<br/>成功/失败/统计]
```

### 5.2 代码解析策略（按语言）

|语言|解析工具|Chunk 粒度|Chunk Header（始终保留）|
|---|---|---|---|
|**Java**|Tree-sitter + JavaParser|Class / Method / Inner Class|`package` + `imports` + `class signature` + `JavaDoc`|
|**Go**|Tree-sitter Go|`func` / `type struct` / `interface`|`package` + `imports`|
|**React (JS/TS)**|Tree-sitter TypeScript|Component (function/class)|`imports` + `prop types`|
|**Svelte**|Tree-sitter HTML + JS|`<script>` / `<template>` 分别处理|Component name + `exports`|
|**Terraform**|HCL Parser|`resource` / `module` / `data` block|`provider` + `variable refs`|
|**CI/CD (YAML/Jenkinsfile)**|自定义 Parser|`stage` / `job` / `step`|Pipeline name + trigger conditions|
|**SQL/DDL**|sqlparse|`CREATE TABLE` / `CREATE PROCEDURE`|Database + Schema|
|**Confluence**|Confluence REST + Beautiful Soup|`<h2>` 节级别|Page title + Space key + breadcrumb|
|**Confluence 图片**|直接提取 `<ac:image>`|整张图作为一个 chunk|Page title + 图片 alt text|

### 5.3 Chunk 元数据 Schema

```json
{
  "chunk_id": "uuid-v4",
  "source_type": "code | confluence | iac | cicd | schema",
  "repo": "payments-service",
  "file_path": "src/main/java/com/hsbc/payment/OrderService.java",
  "symbol_name": "OrderService.processPayment",
  "symbol_type": "method",
  "language": "java",
  "start_line": 42,
  "end_line": 87,
  "commit_sha": "a1b2c3d",
  "branch": "main",
  "is_prod_version": true,
  "authors": ["john.doe@hsbc.com", "jane.smith@hsbc.com"],
  "last_modified": "2026-04-18T10:30:00Z",
  "classification": "confidential",
  "confluence_space": null,
  "confluence_page_id": null,
  "content_hash": "sha256:...",
  "embedding_model": "cohere.embed-v4",
  "chunk_tokens": 512,
  "ingestion_timestamp": "2026-04-19T02:15:00Z"
}
```

### 5.4 增量同步机制

```mermaid
sequenceDiagram
    participant SF as Step Functions
    participant DDB as DynamoDB<br/>sync_state 表
    participant GH as GitHub Enterprise
    participant ING as Ingestion Worker

    SF->>DDB: 获取 last_sync_sha (per repo)
    DDB-->>SF: sha: "abc123"
    SF->>GH: GET /repos/{repo}/compare/abc123...HEAD
    GH-->>SF: diff: 47 files changed
    SF->>ING: 仅处理变更的 47 个文件
    
    Note over ING: 对于 DELETED 文件:<br/>标记 chunk 为 soft-delete<br/>Neptune 中标记边为 inactive
    
    Note over ING: 对于 MODIFIED 文件:<br/>旧 chunk 保留(版本感知)<br/>新 chunk 插入 is_prod=false<br/>直到部署清单更新
    
    ING->>DDB: 更新 last_sync_sha = HEAD
```

---

## 6. 存储层设计

### 6.1 四层存储架构

```mermaid
graph LR
    subgraph "语义层 (Why / What)"
        PG[(Aurora PostgreSQL<br/>pgvector HNSW<br/>1024 维 Cohere Embed v4)]
    end
    
    subgraph "精确层 (Where / Who)"
        OS[(OpenSearch Serverless<br/>BM25 + Keyword)]
    end
    
    subgraph "关系层 (How / Depends)"
        NEP[(Neptune Serverless<br/>openCypher<br/>代码知识图谱)]
    end
    
    subgraph "原始层 (Truth)"
        S3[(S3 + KMS<br/>Object Lock<br/>版本化原始内容)]
    end
```

### 6.2 Neptune 图谱 Schema

```mermaid
graph LR
    REPO((Repository)) -->|CONTAINS| FILE((File))
    FILE -->|DEFINES| CLASS((Class))
    CLASS -->|HAS_METHOD| METHOD((Method))
    METHOD -->|CALLS| METHOD
    CLASS -->|EXTENDS| CLASS
    CLASS -->|IMPLEMENTS| INTERFACE((Interface))
    METHOD -->|THROWS| EXCEPTION((Exception))
    METHOD -->|READS_TABLE| TABLE((DB Table))
    METHOD -->|WRITES_TABLE| TABLE
    COMMIT((Commit)) -->|MODIFIES| METHOD
    COMMIT -->|AUTHORED_BY| DEVELOPER((Developer))
    METHOD -->|REFERENCED_IN| CONF_PAGE((Confluence Page))
    CONF_PAGE -->|LINKS_TO| CONF_PAGE
    CONF_PAGE -->|BELONGS_TO| SPACE((Confluence Space))
    PIPELINE((CI/CD Pipeline)) -->|BUILDS| REPO
    PIPELINE -->|DEPLOYS_TO| ENV((Environment))
    ENV -->|RUNS_VERSION| COMMIT
```

**核心 openCypher 查询示例**：

```cypher
// Stacktrace 分析：从异常方法沿调用链展开 3 跳 + 最近变更
MATCH path = (m:Method {name: 'OrderService.processPayment'})<-[:CALLS*1..3]-(caller:Method)
OPTIONAL MATCH (c:Commit)-[:MODIFIES]->(m) WHERE c.date > datetime() - duration('P30D')
RETURN path, collect(c) as recent_changes
ORDER BY c.date DESC
```

```cypher
// Change Impact Analysis：修改某方法，影响哪些下游？
MATCH (m:Method {name: 'AuthService.validateToken'})-[:CALLS*1..5]->(downstream:Method)
MATCH (downstream)<-[:HAS_METHOD]-(cls:Class)<-[:DEFINES]-(f:File)<-[:CONTAINS]-(r:Repository)
RETURN DISTINCT r.name, f.path, downstream.name
```

### 6.3 pgvector 表设计

```sql
CREATE TABLE knowledge_chunks (
    chunk_id        UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    source_type     VARCHAR(20) NOT NULL,  -- code, confluence, iac, cicd, schema
    repo            VARCHAR(255),
    file_path       VARCHAR(1024),
    symbol_name     VARCHAR(512),
    symbol_type     VARCHAR(50),           -- class, method, function, component, resource, stage
    language        VARCHAR(20),
    start_line      INT,
    end_line        INT,
    commit_sha      VARCHAR(40),
    branch          VARCHAR(255) DEFAULT 'main',
    is_prod_version BOOLEAN DEFAULT false,
    authors         TEXT[],
    last_modified   TIMESTAMPTZ,
    classification  VARCHAR(20) DEFAULT 'internal',
    confluence_space VARCHAR(100),
    confluence_page_id VARCHAR(50),
    content_text    TEXT NOT NULL,
    content_hash    VARCHAR(64),
    embedding       VECTOR(1024) NOT NULL,  -- Cohere Embed v4
    chunk_tokens    INT,
    ingestion_ts    TIMESTAMPTZ DEFAULT NOW(),
    is_deleted      BOOLEAN DEFAULT false,
    
    -- 索引
    CONSTRAINT idx_source UNIQUE (repo, file_path, symbol_name, commit_sha)
);

-- HNSW 向量索引
CREATE INDEX idx_embedding ON knowledge_chunks 
    USING hnsw (embedding vector_cosine_ops) 
    WITH (m = 16, ef_construction = 200);

-- 元数据过滤索引
CREATE INDEX idx_repo ON knowledge_chunks (repo) WHERE NOT is_deleted;
CREATE INDEX idx_prod ON knowledge_chunks (is_prod_version) WHERE NOT is_deleted;
CREATE INDEX idx_source_type ON knowledge_chunks (source_type) WHERE NOT is_deleted;
CREATE INDEX idx_symbol ON knowledge_chunks (symbol_name) WHERE NOT is_deleted;
```

### 6.4 版本感知查询策略

```mermaid
flowchart TD
    Q[用户查询] --> V{版本上下文?}
    V -->|"Production Support<br/>线上出了问题"| PROD["过滤: is_prod_version = true<br/>使用 deployment_registry 中的 SHA"]
    V -->|"Code Discovery<br/>最新代码怎么写的"| MAIN["过滤: branch = 'main'<br/>AND is_deleted = false"]
    V -->|"指定版本<br/>v2.3 的实现"| TAG["过滤: commit_sha IN<br/>tag_registry WHERE tag = 'v2.3'"]
    V -->|"变更分析<br/>最近改了什么"| ALL["不过滤版本<br/>按 last_modified DESC 排序"]
```

---

## 7. 查询与编排层设计

### 7.1 请求生命周期

```mermaid
sequenceDiagram
    participant U as DSP 工程师
    participant APIGW as API Gateway<br/>+ WAF
    participant AUTH as JWT Authorizer
    participant DDB_P as DynamoDB<br/>权限缓存
    participant SR as Scene Router
    participant HR as Haiku 4<br/>意图/复杂度
    participant VR as pgvector
    participant BR as OpenSearch
    participant GR as Neptune
    participant RRF as RRF 融合
    participant RK as Rerank 3.5
    participant LLM as Opus 4 / Sonnet 4
    participant GD as Guardrails
    participant CIT as Citation Extractor
    participant DDB_A as DynamoDB<br/>审计日志

    U->>APIGW: POST /v1/query<br/>{scene: "prod_support",<br/> query: "NullPointerException...",<br/> context: {env: "prod"}}
    APIGW->>AUTH: 验证 JWT (Entra ID)
    AUTH->>DDB_P: 获取 user → accessible_repos
    AUTH-->>APIGW: ✅ user=john.doe, repos=[payments, auth, ...]
    
    APIGW->>SR: 加载 prod_support.yaml 配置
    SR->>HR: 意图分类 + 实体抽取 + 复杂度评分
    HR-->>SR: {intent: stacktrace_analysis,<br/>entities: [NullPointerException,<br/>OrderService.processPayment],<br/>complexity: 0.85 → Opus 4}
    
    Note over SR: 构建元数据过滤器:<br/>repo IN accessible_repos<br/>AND is_prod_version = true
    
    par 三路并行召回
        SR->>VR: 向量检索 (Top 30)
        SR->>BR: BM25: "OrderService" "processPayment" "NullPointerException" (Top 30)
        SR->>GR: 图展开: processPayment ←[:CALLS*1..3]← + recent commits (Top 20)
    end
    
    VR-->>RRF: 30 chunks
    BR-->>RRF: 30 chunks
    GR-->>RRF: 20 chunks (含调用链上下文)
    
    RRF->>RK: 去重 + RRF 融合 → Top 20 → Rerank
    RK-->>SR: Top 8 高相关片段
    
    SR->>LLM: Opus 4 Prompt:<br/>System Prompt (场景模板)<br/>+ 8 chunks + 调用链图<br/>+ 引用强制规则
    LLM-->>GD: 原始回答
    GD->>GD: 1. 引用校验 (每个 [ref] 是否在 context 中)<br/>2. PII 扫描<br/>3. Grounding Check (阈值 0.7)
    GD-->>CIT: 校验后回答
    CIT->>CIT: 将 [repo/file:line@sha] 转为<br/>GitHub Enterprise 可点击链接
    CIT-->>APIGW: 最终响应
    
    APIGW->>DDB_A: 写入审计日志<br/>{user, query, scene, model,<br/>tokens, latency, chunks_used}
    APIGW-->>U: 带溯源链接的分析结果
```

### 7.2 智能 LLM 路由详细设计

```mermaid
flowchart TD
    Q[用户查询] --> H4[Haiku 4<br/>意图分类 + 复杂度评分<br/>~200ms, ~$0.001/次]
    
    H4 -->|complexity < 0.3<br/>纯元数据查询| DB[直接查数据库<br/>不调用 LLM<br/>$0/次]
    H4 -->|0.3 ≤ complexity < 0.7<br/>标准查询| SN4[Sonnet 4<br/>~$0.04/次<br/>约 60% 流量]
    H4 -->|complexity ≥ 0.7<br/>复杂推理/多跳分析| OP4[Opus 4<br/>~$0.20/次<br/>约 25% 流量]
    
    DB --> R[返回结果]
    SN4 --> R
    OP4 --> R
    
    style DB fill:#4CAF50,color:#fff
    style SN4 fill:#2196F3,color:#fff
    style OP4 fill:#FF5722,color:#fff
```

**复杂度评分示例（Haiku 4 Prompt）**：

```
你是查询复杂度评估器。评分 0-1：
- 0.0-0.2: 元数据查询 ("这个文件的作者是谁")
- 0.3-0.5: 单文件/单方法理解 ("这个方法做了什么")  
- 0.5-0.7: 跨文件分析 ("这两个服务如何交互")
- 0.7-0.9: 多跳推理 ("分析这个 stacktrace 的根因")
- 0.9-1.0: 跨领域综合 ("这个变更对安全合规有什么影响")

仅输出 JSON: {"intent": "...", "entities": [...], "complexity": 0.X}
```

### 7.3 Hybrid Retrieval — RRF 融合算法

```python
def reciprocal_rank_fusion(
    vector_results: List[Chunk],
    bm25_results: List[Chunk],
    graph_results: List[Chunk],
    k: int = 60,  # RRF 常数
    weights: dict = {"vector": 1.0, "bm25": 1.0, "graph": 1.5}  # 图谱加权
) -> List[Chunk]:
    """
    三路 RRF 融合。graph_results 权重更高，
    因为代码调用关系对 Stacktrace 分析更有价值。
    """
    scores = defaultdict(float)
    
    for source, results, weight in [
        ("vector", vector_results, weights["vector"]),
        ("bm25", bm25_results, weights["bm25"]),
        ("graph", graph_results, weights["graph"]),
    ]:
        for rank, chunk in enumerate(results):
            scores[chunk.chunk_id] += weight * (1.0 / (k + rank + 1))
    
    sorted_chunks = sorted(scores.items(), key=lambda x: x[1], reverse=True)
    return [get_chunk(cid) for cid, _ in sorted_chunks[:20]]  # Top 20 → Reranker
```

### 7.4 Knowledge API 设计

```yaml
# REST API v1
openapi: "3.0"
info:
  title: KIP - Knowledge Infrastructure Platform API
  version: "1.0"

paths:
  /v1/query:
    post:
      summary: 统一知识查询接口
      requestBody:
        content:
          application/json:
            schema:
              type: object
              required: [scene, query]
              properties:
                scene:
                  type: string
                  description: "场景 ID，对应 YAML 配置"
                  example: "production_support"
                query:
                  type: string
                  example: "java.lang.NullPointerException at com.hsbc.payment..."
                context:
                  type: object
                  properties:
                    environment:
                      type: string
                      enum: [prod, uat, sit]
                    version_filter:
                      type: string
                      enum: [prod_deployed, main_latest, specific_tag]
                    specific_tag:
                      type: string
                stream:
                  type: boolean
                  default: true
      responses:
        200:
          description: 流式响应（SSE）
          content:
            text/event-stream:
              schema:
                type: object
                properties:
                  answer:
                    type: string
                  citations:
                    type: array
                    items:
                      type: object
                      properties:
                        text: { type: string }
                        source: { type: string }
                        github_url: { type: string }
                        relevance_score: { type: number }
                  metadata:
                    type: object
                    properties:
                      model_used: { type: string }
                      total_tokens: { type: integer }
                      latency_ms: { type: integer }
                      retrieval_sources: { type: object }

  /v1/feedback:
    post:
      summary: 用户反馈接口
      requestBody:
        content:
          application/json:
            schema:
              type: object
              required: [query_id, rating]
              properties:
                query_id: { type: string, format: uuid }
                rating: { type: string, enum: [thumbs_up, thumbs_down] }
                comment: { type: string }

  /v1/scenes:
    get:
      summary: 列出所有可用场景
    
  /v1/scenes/{scene_id}:
    get:
      summary: 获取场景详情与配置

  /v1/admin/ingest:
    post:
      summary: 手动触发增量同步
      security: [admin_role]

  /v1/admin/stats:
    get:
      summary: 知识库统计（chunk 数、图谱节点数、查询量）
```

---

## 8. 知识库作为基础设施：多场景支撑架构

### 8.1 场景配置体系

```
kip-config/                          # Git 仓库
├── scenes/
│   ├── production_support.yaml      # P0
│   ├── code_discovery.yaml          # P0
│   ├── onboarding_assistant.yaml    # P1
│   ├── change_impact_analysis.yaml  # P1
│   ├── security_review.yaml         # P1
│   ├── incident_rca.yaml            # P2
│   ├── cicd_copilot.yaml            # P2
│   └── doc_generation.yaml          # P2
├── prompts/
│   ├── system/
│   │   ├── prod_support_system.md
│   │   ├── code_discovery_system.md
│   │   └── ...
│   └── templates/
│       ├── citation_instruction.md   # 通用引用规则
│       └── safety_preamble.md        # 通用安全前缀
├── guardrails/
│   ├── default_guardrails.yaml
│   └── security_review_guardrails.yaml  # 更严格
└── evaluation/
    ├── golden_sets/
    │   ├── prod_support_golden.json   # 200 题
    │   ├── code_discovery_golden.json # 100 题
    │   └── ...
    └── thresholds.yaml               # 各场景质量阈值
```

### 8.2 场景配置详细示例

```yaml
# scenes/change_impact_analysis.yaml
scene:
  name: "change_impact_analysis"
  display_name: "变更影响评估"
  description: "分析代码变更对上下游系统的潜在影响"
  icon: "🔄"
  
  retrieval_strategy:
    primary: "graph"
    graph_query: |
      MATCH (m:Method {name: $method_name})-[:CALLS*1..5]->(downstream:Method)
      MATCH (downstream)<-[:HAS_METHOD]-(cls:Class)
      OPTIONAL MATCH (cls)-[:READS_TABLE|WRITES_TABLE]->(t:Table)
      RETURN downstream, cls, collect(t) as affected_tables
    secondary: "vector"
    tertiary: "bm25"
    fusion: "reciprocal_rank"
    fusion_weights:
      graph: 2.0      # 图谱权重最高 — 影响分析的核心
      vector: 0.8
      bm25: 0.5
    rerank: true
    top_k: 10

  llm_routing:
    complexity_threshold: 0.5   # 变更分析通常较复杂，多用 Opus
    default_model: "sonnet-4"
    complex_model: "opus-4"

  version_context: "main_latest"  # 始终查最新代码

  prompt_template_ref: "prompts/system/change_impact_system.md"
  
  guardrails_ref: "guardrails/default_guardrails.yaml"
  
  output_format:
    sections:
      - name: "直接影响"
        description: "直接调用该方法的上游服务"
      - name: "间接影响"
        description: "2-5 跳内的传递依赖"
      - name: "数据层影响"
        description: "受影响的数据库表和存储过程"
      - name: "风险评估"
        description: "HIGH/MEDIUM/LOW + 建议的测试范围"
      - name: "建议"
        description: "建议的回归测试和通知对象"
```

### 8.3 新场景接入流程

```mermaid
flowchart LR
    A[1. 创建 scene YAML<br/>+ prompt template] --> B[2. 编写 golden set<br/>≥50 道评估题]
    B --> C[3. PR Review<br/>DevOps 团队审核]
    C --> D[4. CI 自动评估<br/>Ragas 跑分 ≥ 阈值]
    D --> E[5. 合并 → 自动部署<br/>新场景上线]
    E --> F[6. 灰度 10% 流量<br/>监控质量指标]
    F --> G[7. 全量发布]
```

> **接入时间**：一个新场景从提出到上线，预计 **2-3 个工作日**（主要时间花在编写 golden set）。

---

## 9. 安全、合规与治理

### 9.1 安全架构总览

```mermaid
graph TB
    subgraph "认证与授权"
        E_ID[Entra ID<br/>SSO / OIDC]
        JWT[JWT Token<br/>RS256 签名验证]
        RBAC[RBAC Engine<br/>GitHub repo-level]
        DDB_P[(DynamoDB<br/>权限缓存 TTL=24h)]
    end
    
    subgraph "数据保护"
        KMS[AWS KMS CMK<br/>自动轮转 365 天]
        TLS[TLS 1.3<br/>全链路传输加密]
        S3_ENC[S3 SSE-KMS<br/>+ Object Lock]
        PG_ENC[Aurora TDE<br/>+ SSL 强制]
        NEP_ENC[Neptune<br/>静态加密]
    end
    
    subgraph "网络隔离"
        VPC[Private VPC<br/>无 IGW]
        VPCEP[VPC Endpoints<br/>Bedrock/S3/DDB/...]
        SG[Security Groups<br/>最小权限]
        WAF_R[WAF Rules<br/>Rate Limit + IP 白名单]
    end
    
    subgraph "审计与合规"
        CT[CloudTrail<br/>Org Trail]
        AUDIT_LOG[(审计日志<br/>7 年保留)]
        MACIE[Amazon Macie<br/>S3 PII 扫描]
        GD_AWS[GuardDuty<br/>威胁检测]
    end
    
    E_ID --> JWT --> RBAC --> DDB_P
    KMS --> S3_ENC
    KMS --> PG_ENC
    KMS --> NEP_ENC
    VPC --> VPCEP
    VPC --> SG
    CT --> AUDIT_LOG
```

### 9.2 RBAC 权限同步机制

```mermaid
sequenceDiagram
    participant CRON as 每日权限同步<br/>Step Functions
    participant GH as GitHub Enterprise<br/>REST API
    participant DDB as DynamoDB<br/>user_permissions 表
    participant AUTH as JWT Authorizer

    CRON->>GH: GET /orgs/hsbc-dsp/teams
    GH-->>CRON: teams: [payments-team, auth-team, ...]
    
    loop 每个 team
        CRON->>GH: GET /teams/{id}/repos
        GH-->>CRON: repos: [payments-service, ...]
        CRON->>GH: GET /teams/{id}/members
        GH-->>CRON: members: [john.doe, jane.smith, ...]
    end
    
    CRON->>DDB: BatchWrite: user→repos 映射
    Note over DDB: john.doe → [payments-service,<br/>auth-service, shared-lib]<br/>TTL = 48h (安全余量)
    
    Note over AUTH: 查询时:<br/>1. JWT 解析 user_id<br/>2. DDB 查 accessible_repos<br/>3. 注入 MetadataFilter
```

### 9.3 审计日志 Schema

```json
{
  "audit_id": "uuid",
  "timestamp": "2026-04-19T18:30:00Z",
  "user_id": "john.doe@hsbc.com",
  "user_ip": "10.x.x.x",
  "action": "QUERY",
  "scene": "production_support",
  "query_text": "NullPointerException at OrderService...",
  "model_used": "claude-opus-4-7",
  "input_tokens": 12500,
  "output_tokens": 850,
  "latency_ms": 3200,
  "chunks_retrieved": 8,
  "repos_accessed": ["payments-service", "shared-lib"],
  "response_truncated_hash": "sha256:first500chars...",
  "feedback": null,
  "session_id": "uuid"
}
```

**保留策略**：

- DynamoDB TTL = 90 天（热数据，快速查询）
- 到期后自动归档到 S3 Glacier（7 年，SOX 合规）
- Athena 可查询归档数据

### 9.4 FCA/PRA 合规对照

|FCA/PRA 要求|KIP 实现|
|---|---|
|**SYSC 13 — 运营风险管理**|知识库本身提升了 Production Support 效率，降低 MTTR|
|**SYSC 3.2 — 职责分离**|管理员 ≠ 审核员 ≠ 用户，通过 IAM Role + Entra ID Group 实现|
|**FCA PRIN 11 — 与监管者关系**|审计日志可按需导出给 FCA/PRA 审计师|
|**PRA SS1/15 — 外包 & 第三方**|Bedrock/AWS 已在 HSBC 第三方风险管理框架内批准|
|**SOX Section 404**|完整审计追溯 + 变更控制（场景配置走 GitOps PR）|

### 9.5 SOX 职责分离矩阵

|角色|权限|Entra ID Group|
|---|---|---|
|**KIP Admin**|Terraform 部署、基础设施变更|`dsp-kip-admin` (≤3 人)|
|**Scene Author**|创建/修改场景 YAML + Prompt（需 PR Review）|`dsp-kip-scene-author`|
|**Scene Reviewer**|审批场景变更 PR（不能与 Author 同一人）|`dsp-kip-scene-reviewer`|
|**User**|查询知识库|`dsp-kip-user` (全部门 ~200 人)|
|**Auditor**|只读审计日志、评估报告|`dsp-kip-auditor`|

---

## 10. 可观测性与运维

### 10.1 可观测性三支柱

```mermaid
graph TB
    subgraph "Metrics (CloudWatch)"
        M1[查询延迟 P50/P90/P99]
        M2[模型使用量 by model]
        M3[检索命中率 by source]
        M4[Guardrails 拦截率]
        M5[DAU / 日查询量]
        M6[反馈 👍/👎 比率]
        M7[Token 成本/日]
    end
    
    subgraph "Logs (CloudWatch Logs)"
        L1[查询日志 → DynamoDB]
        L2[Ingestion 日志]
        L3[错误日志 + 告警]
    end
    
    subgraph "Traces (X-Ray via OpenTelemetry)"
        T1[完整请求链路<br/>API → Router → Retriever<br/>→ Reranker → LLM → Guardrails]
        T2[各阶段耗时分解]
    end
    
    subgraph "Dashboards"
        D1[运营仪表盘<br/>实时健康状态]
        D2[质量仪表盘<br/>每周 Ragas 评分趋势]
        D3[成本仪表盘<br/>按模型/场景拆分]
    end
    
    M1 & M2 & M3 --> D1
    M6 --> D2
    M7 --> D3
```

### 10.2 告警规则

|指标|阈值|告警级别|通知渠道|
|---|---|---|---|
|查询延迟 P99|> 15s|🟡 WARN|Slack|
|查询延迟 P99|> 30s|🔴 CRITICAL|Slack + PagerDuty|
|每日 Guardrails 拦截率|> 30%|🟡 WARN|Slack|
|反馈 👎 比率（7日滚动）|> 25%|🔴 CRITICAL|Slack + Email|
|Ragas 周评分|环比下降 > 5%|🟡 WARN|Slack|
|日成本|> $2,000|🟡 WARN|Slack|
|Ingestion 失败|任何仓库失败|🔴 CRITICAL|Slack|
|Neptune / pgvector 不可用|健康检查失败|🔴 CRITICAL|PagerDuty|

### 10.3 反馈闭环与持续改进

```mermaid
flowchart TD
    FB[用户反馈 👍/👎] --> DDB[(DynamoDB<br/>feedback 表)]
    DDB --> WEEKLY[每周 Step Functions]
    
    WEEKLY --> A1[统计分析<br/>按场景/模型/检索源]
    WEEKLY --> A2[👎 案例人工抽检<br/>每周 20 条]
    WEEKLY --> A3[Ragas 自动评估<br/>Golden Set 回归]
    
    A1 --> REPORT[周报 → Slack Channel]
    A2 --> IMPROVE{改进方向}
    A3 --> REPORT
    
    IMPROVE -->|Prompt 问题| PP[调整 Prompt Template<br/>PR → Review → 发布]
    IMPROVE -->|检索不相关| RP[调整 Retrieval 参数<br/>权重/Top-K/阈值]
    IMPROVE -->|知识缺失| KP[补充缺失的知识源<br/>新增 Confluence Space 等]
    IMPROVE -->|模型能力不足| MP[升级到更强模型<br/>或 Fine-tune]
```

---

## 11. 技术栈总览

|层|组件|技术选择|版本/规格|
|---|---|---|---|
|**身份认证**|SSO|Microsoft Entra ID (OIDC)|—|
|**入口**|API Gateway|AWS API Gateway (Regional) + WAF v2|REST + SSE 流式|
|**编排**|RAG Orchestrator|**LlamaIndex** on ECS Fargate|Python 3.12, LlamaIndex 0.12+|
|**工作流调度**|Pipeline|AWS Step Functions|Express Workflow|
|**主推理 LLM**|复杂任务|**Claude Opus 4** (`claude-opus-4-7`) on Bedrock|VPC Endpoint|
|**日常 LLM**|标准查询|**Claude Sonnet 4** (`claude-sonnet-4-6`) on Bedrock|VPC Endpoint|
|**路由 LLM**|意图分类|**Claude Haiku 4** (`claude-haiku-4-5`) on Bedrock|VPC Endpoint|
|**Embedding**|Text + Image|**Cohere Embed v4** (`cohere.embed-v4:0`) on Bedrock|1024 维|
|**Reranker**|重排|**Cohere Rerank 3.5** (`cohere.rerank-v3-5:0`) on Bedrock|—|
|**向量数据库**|语义检索|Aurora PostgreSQL Serverless v2 + pgvector (HNSW)|2-8 ACU|
|**图数据库**|知识图谱|Amazon Neptune Serverless (openCypher)|2-16 NCU|
|**全文检索**|BM25|OpenSearch Serverless|2 OCU|
|**对象存储**|原始内容|S3 + KMS CMK + Object Lock|—|
|**缓存/审计**|权限 + 日志 + 反馈|DynamoDB On-Demand|—|
|**密钥管理**|—|AWS Secrets Manager + KMS|自动轮转|
|**代码解析**|AST|Tree-sitter (多语言)|Java/Go/JS/Svelte|
|**可观测**|Metrics/Logs|CloudWatch + X-Ray (OpenTelemetry)|—|
|**IaC**|基础设施|Terraform|≥1.7|
|**CI/CD**|部署|GitHub Actions + OIDC → AWS|无长期密钥|
|**代码仓库**|源码|GitHub Enterprise|—|

---

## 12. 成本拆分

### 12.1 月度成本明细

**假设**：200 DAU × 10 查询/天 × 22 工作日 = ~44K 查询/月

| 类别                  | 项目                              | 规格                               | 月成本 (USD) |
| ------------------- | ------------------------------- | -------------------------------- | --------- |
| **LLM — 推理**        | Claude Opus 4                   | 11K 查询 (25%) × ~15K tok          | ~$6,600   |
|                     | Claude Sonnet 4                 | 26K 查询 (60%) × ~11K tok          | ~$1,700   |
|                     | Claude Haiku 4 (路由)             | 44K × 2K tok                     | ~$90      |
| **LLM — Ingestion** | Haiku 4 (图谱抽取)                  | ~200M tok/月                      | ~$200     |
| **Embedding**       | Cohere Embed v4                 | Ingestion ~500M tok + Query ~50M | ~$250     |
| **Reranker**        | Cohere Rerank 3.5               | 44K × 20 docs                    | ~$120     |
| **数据库**             | Aurora PostgreSQL Serverless v2 | 2-8 ACU, ~500GB                  | ~$1,800   |
|                     | Neptune Serverless              | 2-16 NCU (自动                     |           |
