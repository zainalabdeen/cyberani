X-Agent Integration with Odoo Meeting Minutes & Project Plan Phase 1:
Odoo AI Assistant \| Phase 2: Extended X-Agent Integration

# Email / MOM Summary

Subject: MOM - X-Agent Integration with Odoo Dear Team, Following our
discussion regarding the X-Agent integration with Odoo, below is the
agreed high-level plan. The detailed implementation notes and team
responsibilities are included in the next section of this document. \##
Phase 1 - Odoo AI Assistant - Integrate the Odoo AI chat interface with
X-Agent. - Allow users to ask ERP-related questions and perform data
analysis from inside Odoo. - Keep Phase 1 read-only. X-Agent can view
and analyze data, but it will not create, update, or delete ERP
records. - All data access must follow the logged-in user's Odoo
permissions, record rules, and company access. - ERP answers should be
based on Odoo data/tools and approved Knowledge Base sources. - ERP Team
will evaluate the Odoo 19 AI modules and adapt the required capabilities
for our Odoo 18 environment. - ERP Team will prepare the application and
database environment, including required Python libraries and pgvector
if needed. - X-Agent Team will provide the required model/service
integration details and tool-calling interface. - Where possible, reuse
X-Agent's existing embedding and retrieval capabilities for the
Knowledge Base instead of creating a separate RAG stack in Odoo. - ERP
Team will build controlled Odoo tools that X-Agent can call for ERP data
and analysis. \## Initial POC The first POC should validate the full
flow: Odoo User -\> Odoo AI Chat -\> X-Agent -\> Odoo Tool -\> X-Agent
Analysis -\> Odoo Chat. The POC should also confirm user access control,
Odoo record context, grounded answers, tool calling, and auditability.
\## Phase 2 - Actions & X-Agent to Odoo - Add controlled AI actions such
as creating requests/tasks and sending emails. - Add confirmation or
approval for sensitive actions. - Allow users to interact with Odoo from
the existing X-Agent UI. - Use X-Agent's wider capabilities such as chat
history, memory, RAG, vision, and other approved tools together with
Odoo. The final target is to support both directions: Odoo UI -\>
X-Agent -\> Odoo, and X-Agent UI -\> Odoo. Please review the detailed
implementation plan below and share any comments or required changes.
Best regards, Zainalabdeen

# Detailed Project Plan

## 1. Objective

The goal is to bring X-Agent into Odoo without building a second
independent AI platform. X-Agent will remain the AI reasoning and chat
layer, while Odoo remains responsible for ERP data, business rules,
permissions, and controlled tools. \## 2. Phase 1 Scope - Odoo users
access X-Agent from the Odoo AI/chat interface. - The integration is
read-only for ERP data. - X-Agent can request Odoo data only through
approved tools. - Odoo executes requests under the current user's access
rights and record rules. - The current Odoo context can be passed to
X-Agent, such as active model, active record, company, and selected
records. - Answers should be grounded in Odoo tool results or approved
Knowledge Base content. - External/general answers should be restricted
in the initial ERP mode to reduce hallucination risk. \## 3. ERP Team
Responsibilities - Review Odoo 19 Enterprise AI modules and identify the
parts required for our Odoo 18 environment. - Adapt/backport the
required Odoo AI integration components to Odoo 18. - Prepare the Odoo
AI/chat interface and connection with X-Agent. - Prepare the ERP
environment and install the required Python dependencies. - Verify
PostgreSQL compatibility and install/configure pgvector if it is
required by the selected Odoo AI components. - Build the Odoo tools and
business services used by X-Agent. - Ensure tool execution follows Odoo
ACLs, record rules, multi-company rules, and business logic. - Provide
logging/auditing for X-Agent calls to Odoo tools. \## 4. X-Agent Team
Responsibilities / Required Information The X-Agent team will provide
the integration details required for Odoo to use the existing X-Agent
services.

  -----------------------------------------------------------------------
  Area                    Current Service         Required from X-Agent
                                                  Team
  ----------------------- ----------------------- -----------------------
  Chat / Reasoning        GPT-OSS / Groq          Base URL,
                                                  authentication/API key,
                                                  request/response
                                                  format, streaming
                                                  support, structured
                                                  output support

  Vision                  Qwen / Groq             Base URL,
                                                  authentication,
                                                  supported file/image
                                                  input and response
                                                  format

  Embeddings              mxbai-embed-large /     Base URL,
                          Ollama                  authentication if
                                                  applicable, embedding
                                                  API details

  Tool Calling            X-Agent agent runtime   How tools are defined,
                                                  registered, authorized,
                                                  called, and how tool
                                                  results/errors are
                                                  returned

  Web Search              Google Custom Search    API/configuration
                                                  details if we decide to
                                                  enable web search in
                                                  ERP mode

  Chat Context            X-Agent                 How Odoo can pass ERP
                                                  mode, user context,
                                                  active record context,
                                                  and
                                                  conversation/session
                                                  information
  -----------------------------------------------------------------------

## 5. Knowledge Base / RAG

The Knowledge Base will be hosted and managed in Odoo using PostgreSQL
pgvector. Odoo will manage the approved documents/URLs, content chunks,
embeddings, and stored vectors. The main architecture point to agree
with the X-Agent team is where the RAG process should be implemented.
X-Agent already provides RAG capabilities, but it does not provide the
Knowledge Base that Odoo will use. We therefore need to decide which
side will manage retrieval and RAG orchestration while keeping the
Knowledge Base and vector storage in Odoo. \### Proposed Knowledge Base
Flow

``` mermaid
flowchart TD
    A((Document / URL)) --> B[Odoo Knowledge Base]
    B --> C[Content Processing]
    C --> D[Embedding Generation<br/>Ollama + mxbai-embed-large]
    D --> E[Odoo PostgreSQL / pgvector<br/>Chunks and vectors stored in Odoo]
```

### Proposed Query-Time RAG Flow

``` mermaid
flowchart TD
    A((User Question)) --> B[Question Embedding]
    B --> C[Odoo Vector Search<br/>Retrieve the most relevant permitted content]
    C --> D[RAG Context<br/>Relevant Knowledge Base content]
    D --> E[X-Agent / GPT-OSS<br/>Generate grounded answer]
```

### RAG Ownership - Decision Required

The Knowledge Base and pgvector storage will remain in Odoo. The open
point is which component should manage the retrieval flow and prepare
the RAG context for the model. - Option 1 - Odoo-managed RAG: Odoo
performs the vector search, retrieves the permitted relevant content,
and sends the question together with the RAG context to X-Agent for
answer generation. - Option 2 - X-Agent-managed RAG: X-Agent manages the
RAG orchestration and requests relevant Knowledge Base content from Odoo
through a controlled retrieval interface. Odoo still owns the Knowledge
Base, pgvector storage, and access control. - The final option should be
selected based on security, access control, reuse of X-Agent
capabilities, maintainability, performance, and avoiding duplicated RAG
logic. \### Points to Confirm with X-Agent Team - Can X-Agent manage RAG
when the Knowledge Base and vectors are stored in Odoo? - What interface
does X-Agent need to request and receive relevant chunks from Odoo? -
Should Odoo send the retrieved RAG context to X-Agent, or should X-Agent
control the retrieval sequence? - How will Knowledge Base source
references be returned with the final answer? - For Phase 1, do we keep
external Google Search disabled and depend only on Odoo data and the
Odoo Knowledge Base? \## 6. Odoo Tools Odoo tools will be the controlled
interface between X-Agent and ERP data. X-Agent decides which tool is
needed; Odoo executes the business operation and returns structured
data. - Get current/selected record information - Search permitted Odoo
records - Get sales summary and breakdown - Get customer summary - Get
project information and profitability - Get receivables aging - Get
approved finance/revenue metrics Phase 1 tool rules: - Read-only. - Use
Odoo ORM/business logic; no unrestricted SQL access. - Respect the
requesting user's Odoo permissions. - Return structured and limited data
suitable for AI analysis. - Log tool calls for troubleshooting and
security review. \## 7. Hallucination / Grounding Controls For the
initial rollout, the Odoo chat should run X-Agent in a dedicated
ERP/Odoo mode. The main rule is: no ERP factual answer should be
presented as verified unless it is supported by an Odoo tool result or
an approved ERP Knowledge Base source. - Disable or restrict general web
search initially. - Do not use general model knowledge to invent ERP
values or business facts. - If the required information cannot be
retrieved, X-Agent should clearly state that it cannot verify the answer
from Odoo. - Pass active Odoo record context as temporary conversation
context, not permanent memory. \## 8. Phase 1 POC / Acceptance
Criteria 1. User opens the AI/X-Agent chat inside Odoo. 2. Odoo sends
the user question together with the relevant ERP context. 3. X-Agent
identifies the required Odoo tool. 4. X-Agent calls the tool using the
agreed integration mechanism. 5. Odoo executes the tool under the user's
access rights. 6. Odoo returns structured data to X-Agent. 7. X-Agent
analyzes the result and returns a grounded answer to the Odoo UI. 8. The
request and tool execution can be traced for troubleshooting/audit
purposes. \## 9. Phase 2 Scope - Introduce controlled write/action tools
such as creating requests, tasks/activities, and sending emails. -
Require confirmation/approval for sensitive or high-impact operations. -
Enable Odoo access from the existing X-Agent UI. - Combine Odoo tools
with X-Agent memory/history, RAG, vision, and other approved
capabilities. - Expand into dashboards, proactive analysis, and
additional ERP business areas after Phase 1 is stable. \## 10. Target
Architecture Odoo UI -\> X-Agent -\> Odoo Tools / ERP Data X-Agent UI
-\> Odoo Tools / ERP Data (Phase 2) X-Agent remains the AI reasoning,
conversation, and orchestration layer. Odoo remains the ERP source of
truth, security authority, business-logic layer, and provider of
controlled tools.
