# AI Operations Platform

## Product Requirements Document

**Status:** Draft
**Version:** 1.0
**Target:** MVP + 12-week implementation roadmap

---

# 1. Product Summary

The AI Operations Platform is a unified operational system designed to automate and coordinate:

* Lead acquisition and qualification
* AI-assisted sales
* Follow-up and conversion
* Checkout and payment confirmation
* Student support
* Knowledge retrieval through RAG
* Customer Success
* Student engagement monitoring
* Financial operations
* Human support handoff
* Operational analytics
* AI evaluation and observability

The system must allow the business to scale revenue and customer volume without requiring equivalent growth in operational headcount.

The platform will initially focus on WhatsApp as the primary communication channel and progressively expand to Instagram and Facebook.

---

# 2. Product Vision

Create an AI-powered operating layer that connects communication channels, business processes, customer data, payment systems, knowledge, and human operators.

The platform should manage the lifecycle:

```text
LEAD
  │
  ▼
AI SALES
  │
  ▼
QUALIFICATION
  │
  ▼
OFFER
  │
  ▼
CHECKOUT
  │
  ▼
PAYMENT
  │
  ▼
ONBOARDING
  │
  ▼
STUDENT
  │
  ├─────────────┐
  ▼             ▼
SUPPORT     CUSTOMER SUCCESS
  │             │
  └──────┬──────┘
         ▼
     ENGAGEMENT
         │
         ▼
      RETENTION
```

Every important interaction should produce structured operational data.

---

# 3. Business Objectives

The platform must improve five primary outcomes.

## 3.1 Revenue

Increase conversion through:

* Faster response
* Automated qualification
* Objection handling
* Intelligent follow-up
* Checkout automation
* Payment confirmation

## 3.2 Operational efficiency

Reduce manual work in:

* Lead management
* Student support
* Follow-up
* Payment verification
* Customer Success
* Reporting

## 3.3 Customer experience

Provide:

* Faster support
* Personalized responses
* Accurate course information
* Proactive onboarding
* Proactive engagement

## 3.4 Scalability

Increase customer volume without proportionally increasing:

* Support staff
* Sales staff
* Administrative staff

## 3.5 Intelligence

Provide visibility into:

* Conversion
* Lead behavior
* Student engagement
* Support demand
* Payment behavior
* AI performance

---

# 4. Product Principles

The system will follow these architectural principles.

1. PostgreSQL is the primary business system of record.

2. Chatwoot is the conversation and human-service layer.

3. n8n orchestrates workflows but does not own critical business state.

4. Redis is used for n8n queue-mode execution when horizontal scaling becomes necessary.

5. RabbitMQ is optional and should only be introduced if durable cross-system event distribution becomes necessary.

6. AI performs interpretation, reasoning, classification, and response generation.

7. Deterministic services perform critical business operations.

8. Payment state is determined by the payment provider, not by the LLM.

9. AI evaluation begins before production deployment.

10. Security, observability, logging, and event capture begin in Foundation.

11. Events are captured from Day 1.

12. Analytics dashboards are built only after reliable data collection exists.

13. Lead scoring starts rule-based and should later be calibrated against real conversion data.

14. Customer Support and Customer Success are separate domains.

15. WhatsApp is the first communication channel.

---

# 5. High-Level Architecture

```text
                         CHANNELS
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
          WhatsApp     Instagram     Facebook
              │            │            │
              └────────────┼────────────┘
                           ▼
                      CHATWOOT
                Conversation Layer
                           │
                        Webhooks
                           │
                           ▼
                  ┌────────────────┐
                  │      n8n       │
                  │ Orchestration  │
                  └────────┬───────┘
                           │
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
      SALES AGENT     SUPPORT AGENT      CS ENGINE
           │               │               │
           │               ▼               │
           │              RAG              │
           │                               │
           └───────────────┼───────────────┘
                           ▼
                    BUSINESS SERVICES
              ┌────────────┼────────────┐
              ▼            ▼            ▼
          Customer      Payment      Messaging
           Service      Service       Service
              │            │            │
              └────────────┼────────────┘
                           ▼
                      PostgreSQL
                           │
                ┌──────────┴──────────┐
                ▼                     ▼
        Operational State           Events
                │                     │
                └──────────┬──────────┘
                           ▼
                  Analytics / BI
```

---

# 6. Infrastructure Architecture

```text
                       n8n Main
                          │
                        Redis
                    ┌─────┼─────┐
                    ▼     ▼     ▼
                 Worker Worker Worker

                          │
                          ▼
                     PostgreSQL
```

Optional future event architecture:

```text
Applications
     │
     ▼
 RabbitMQ
     │
 ┌───┼─────────┐
 ▼   ▼         ▼
Sales CS    Analytics
```

RabbitMQ is not required for the MVP.

---

# 7. Core Domain Model

```text
Contact
 │
 ├── Lead
 │    │
 │    ├── Opportunity
 │    └── FollowUp
 │
 └── Customer
      │
      └── Student

Contact
 │
 └── Conversation
       │
       └── Message

Customer
 │
 ├── Order
 │    └── Payment
 │
 └── Subscription

Student
 │
 ├── Activity
 ├── EngagementScore
 └── SupportCase

AI
 │
 └── AIRun
      ├── PromptVersion
      ├── ToolCall
      ├── Retrieval
      └── Evaluation

All major domain entities
        │
        ▼
      Event
```

---

# 8. Delivery Streams

The product will be divided into six major Epics.

```text
EPIC 0 — PLATFORM FOUNDATION

EPIC 1 — REVENUE & AI SALES

EPIC 2 — STUDENT SUPPORT & RAG

EPIC 3 — CUSTOMER SUCCESS

EPIC 4 — FINANCIAL OPERATIONS

EPIC 5 — DATA, ANALYTICS & AI INTELLIGENCE
```

Cross-cutting concerns:

```text
Security
Testing
Observability
AI Evaluation
Event Collection
Documentation
Infrastructure
```

---

# 9. EPIC 0 — Platform Foundation

## Goal

Provide the technical foundation required by every other capability.

---

## Feature 0.1 — Environment Setup

### User Story

As an engineering team,
we need isolated environments,
so that development and testing do not impact production.

### Tasks

* Create DEV environment
* Create STAGING environment
* Create PROD environment
* Configure domain names
* Configure TLS
* Configure environment variables
* Implement secret management
* Define deployment strategy
* Define backup strategy

### Acceptance Criteria

* DEV, STAGING, and PROD are separated
* Credentials are not stored directly in workflows
* Production services use HTTPS
* Environment-specific configuration is documented

---

## Feature 0.2 — PostgreSQL

### User Story

As the platform,
I need a reliable system of record,
so that business state is independent from automation tools.

### Initial tables

```text
contacts
leads
customers
students
conversations
messages
opportunities
followups
orders
payments
subscriptions
student_activities
support_cases
ai_runs
events
```

### Tasks

* Deploy PostgreSQL
* Create schemas
* Create migrations
* Configure connection pooling
* Configure users
* Configure permissions
* Configure automated backups
* Perform restore test

### Acceptance Criteria

* Data persists independently from n8n
* Schema changes are version-controlled
* Backup restoration is tested successfully

---

## Feature 0.3 — Event Model

### User Story

As the platform,
I need all important business actions recorded as events,
so that analytics and operational history can be reconstructed.

### Event format

```json
{
  "event_id": "uuid",
  "event_type": "lead.created",
  "occurred_at": "timestamp",
  "entity_type": "lead",
  "entity_id": "uuid",
  "conversation_id": "uuid",
  "source": "whatsapp",
  "correlation_id": "uuid",
  "payload": {}
}
```

### Required events

```text
lead.created
lead.contacted
lead.qualified
lead.disqualified
lead.offer.created
lead.checkout.started
lead.abandoned
lead.converted

payment.created
payment.completed
payment.failed

student.created
student.activated
student.inactive

support.requested
support.resolved
support.escalated

ai.response.generated
ai.tool.executed
ai.handoff.requested
```

---

## Feature 0.4 — n8n Platform

### Tasks

* Deploy n8n
* Connect PostgreSQL
* Configure credentials
* Create error workflow
* Configure execution retention
* Define workflow naming convention
* Configure versioning
* Configure Redis if queue mode becomes necessary
* Configure worker infrastructure

---

## Feature 0.5 — Chatwoot

### Goal

Provide a unified conversation layer for customers and human agents.

### Tasks

* Deploy Chatwoot
* Configure organization
* Configure agents
* Configure teams
* Configure WhatsApp inbox
* Configure routing
* Configure tags
* Configure conversation assignment
* Configure webhook integration
* Configure AI → human handoff

---

# 10. EPIC 1 — Revenue & AI Sales

## Goal

Allow AI to handle most of the initial lead journey while ensuring critical actions remain deterministic.

---

# Feature 1.1 — Lead Ingestion

### Story

As the platform,
I want incoming WhatsApp contacts to become structured leads,
so that every sales interaction can be tracked.

### Tasks

* Receive Chatwoot webhook
* Identify contact
* Create contact if missing
* Create lead if missing
* Deduplicate contact
* Store source
* Store campaign if available
* Associate conversation
* Publish `lead.created`

### Acceptance Criteria

* Duplicate messages do not create duplicate leads
* Every conversation has an associated contact
* Every new lead generates an event

---

# Feature 1.2 — Lead State Machine

Lead state must be deterministic.

```text
NEW
 │
 ▼
CONTACTED
 │
 ▼
QUALIFYING
 │
 ├──────► DISQUALIFIED
 │
 ▼
QUALIFIED
 │
 ▼
OFFERED
 │
 ▼
CHECKOUT_STARTED
 │
 ├──────► ABANDONED
 │
 ▼
PAID
```

### Tasks

* Implement lead stage model
* Define allowed transitions
* Create transition service
* Log transitions
* Reject invalid transitions

---

# Feature 1.3 — AI Sales Agent

### Story

As a lead,
I want relevant answers during my purchase journey,
so that I can make a purchase decision without waiting for a human.

### Agent responsibilities

* Understand intent
* Ask qualification questions
* Retrieve product knowledge
* Handle approved objections
* Recommend offers
* Trigger allowed tools
* Request human assistance

### Agent tools

```text
get_contact()

get_lead()

update_lead()

search_knowledge()

get_product()

get_offer()

create_checkout()

schedule_followup()

handoff_to_human()
```

### Tasks

* Create system prompt
* Create sales playbook
* Create objection library
* Create product knowledge
* Implement conversation memory
* Implement tool calling
* Implement structured responses
* Implement fallback rules
* Implement human handoff

---

# Feature 1.4 — Qualification

### Story

As the sales organization,
I want leads qualified automatically,
so that effort is concentrated on viable opportunities.

### Tasks

* Define qualification questions
* Define qualification rules
* Store answers
* Calculate qualification
* Update lead stage
* Emit qualification event

---

# Feature 1.5 — Objection Handling

Initial objection categories:

```text
Price
Trust
Timing
Product fit
Expected results
Payment
Technical questions
```

### Tasks

* Detect objection category
* Search approved response
* Generate contextual response
* Track objection
* Track resolution
* Track subsequent conversion

---

# Feature 1.6 — Checkout Integration

### Story

As a qualified lead,
I want to receive a checkout when I decide to buy,
so that I can complete the purchase immediately.

### Tasks

* Integrate payment gateway
* Generate checkout
* Associate checkout with lead
* Store order
* Handle gateway webhook
* Update payment
* Update lead status

---

# Feature 1.7 — Follow-Up Engine

Follow-ups must be state-driven.

### Data model

```text
followup

id
lead_id
conversation_id
due_at
type
status
attempt
created_at
completed_at
cancelled_at
```

### Initial schedule

```text
30 minutes
1 hour
1 day
```

### Tasks

* Detect abandoned conversation
* Create pending follow-up
* Create scheduler
* Check lead eligibility
* Send follow-up
* Increment attempt
* Stop on conversion
* Stop on opt-out
* Cancel pending follow-ups

---

# Feature 1.8 — Human Handoff

### Triggers

* Uncertain answer
* Unsupported request
* Customer requests human
* Sensitive issue
* Payment dispute
* Repeated AI failure

### Tasks

* Assign conversation
* Add Chatwoot tag
* Notify agent
* Persist handoff reason
* Pause AI
* Resume AI only through controlled rule

---

# 11. EPIC 2 — Student Support & RAG

## Goal

Reduce repetitive human support while ensuring answers are grounded in approved training content.

---

# Feature 2.1 — Knowledge Ingestion

Supported sources:

* Course transcripts
* PDFs
* Training materials
* FAQs
* Product documentation

Pipeline:

```text
Source
  │
  ▼
Extraction
  │
  ▼
Cleaning
  │
  ▼
Chunking
  │
  ▼
Metadata
  │
  ▼
Embedding
  │
  ▼
Vector Store
```

### Metadata example

```json
{
  "course": "nutrition",
  "module": "module_03",
  "lesson": "lesson_05",
  "content_type": "transcript",
  "version": 2
}
```

### Tasks

* Source collection
* Text extraction
* Text cleanup
* Chunk strategy
* Metadata strategy
* Embedding generation
* Index creation
* Re-index mechanism

---

# Feature 2.2 — Retrieval

### Tasks

* Semantic search
* Metadata filtering
* Course filtering
* Lesson filtering
* Top-K retrieval
* Relevance threshold
* Optional reranking
* Retrieval logging

---

# Feature 2.3 — Student Support Agent

### Story

As a student,
I want to ask questions through WhatsApp,
so that I receive immediate support about the course.

### Tasks

* Identify student
* Detect student question
* Retrieve relevant content
* Generate grounded answer
* Reference course/module
* Support text responses
* Support audio responses
* Save support interaction
* Detect uncertainty
* Escalate to human

---

# Feature 2.4 — Support Analytics

Track:

```text
AI resolution rate
Human escalation rate
Response time
Repeated questions
Top support topics
Retrieval confidence
Cost per support interaction
```

---

# 12. EPIC 3 — Customer Success

## Goal

Proactively engage students and reduce inactivity.

---

# Feature 3.1 — Student Onboarding

### Story

As a new student,
I want structured onboarding,
so that I successfully start using the product.

### Tasks

* Detect purchase
* Create student
* Send welcome message
* Confirm login
* Confirm group membership
* Confirm application access
* Confirm first training activity

---

# Feature 3.2 — Student Activity Integration

This depends on the training/application data source.

### Discovery Tasks

* Determine whether application exposes API
* Document available endpoints
* Determine authentication
* Determine activity granularity
* Determine historical access

Fallback:

```text
Application
     │
     ▼
Manual Export
     │
     ▼
Google Sheet / Import
     │
     ▼
Platform
```

---

# Feature 3.3 — Engagement Events

Track:

```text
student.login
student.training.started
student.training.completed
student.group.joined
student.group.left
student.message.sent
student.app.inactive
```

---

# Feature 3.4 — Inactivity Engine

Initial rules:

```text
7 days inactive
15 days inactive
30 days inactive
```

Exact rules must be defined by the business.

Flow:

```text
Student Activity
       │
       ▼
Inactivity Rule
       │
       ▼
CS Action
       │
       ▼
Personalized Message
       │
       ▼
Response?
   ┌───┴───┐
   ▼       ▼
  Yes      No
   │        │
Close    Retry
           │
           ▼
         Human
```

---

# Feature 3.5 — Student Engagement Score

Phase 1 uses simple rules.

Signals:

* Application usage
* Training activity
* Group activity
* Support activity
* Payment status

Categories:

```text
Highly Engaged
Engaged
At Risk
Critical
```

Scores should later be calibrated against real retention data.

---

# 13. EPIC 4 — Financial Operations

## Goal

Automate payment-related operational work.

---

# Feature 4.1 — Payment Status

Required for MVP.

### Tasks

* Payment gateway authentication
* Create payment
* Payment lookup
* Webhook processing
* Payment/customer correlation
* Update order
* Update lead/customer

---

# Feature 4.2 — Proof of Payment Verification

### Flow

```text
Proof Received
      │
      ▼
Payment Lookup
      │
      ▼
Match?
 ┌────┴────┐
 ▼         ▼
YES        NO
 │          │
Approve   Human Review
```

### Tasks

* Receive proof
* Identify customer
* Query gateway
* Detect matching payment
* Store evidence
* Create human review when necessary
* Maintain audit trail

---

# Feature 4.3 — Collections

### Story

As the business,
I want overdue customers automatically contacted,
so that outstanding payments can be recovered.

### Tasks

* Detect overdue payment
* Create collection action
* Send WhatsApp template
* Schedule retry
* Stop when paid
* Escalate when necessary
* Track recovery

---

# Feature 4.4 — Reconciliation

### Tasks

* Import gateway transactions
* Match orders
* Match customers
* Detect unmatched payments
* Flag exceptions
* Produce reconciliation report

---

# 14. EPIC 5 — Data, Analytics & AI Intelligence

## Goal

Transform operational activity into measurable business intelligence.

---

# Feature 5.1 — Sales Funnel

Model:

```text
Lead
 ↓
Contacted
 ↓
Qualified
 ↓
Offer
 ↓
Checkout
 ↓
Paid
```

KPIs:

* Lead volume
* Qualification rate
* Checkout rate
* Conversion rate
* Revenue
* AI-assisted revenue
* Follow-up recovery
* Average time to conversion
* Objection frequency

---

# Feature 5.2 — Student Analytics

KPIs:

* Active students
* Inactive students
* Engagement rate
* At-risk students
* Support usage
* Support resolution rate
* Reactivation rate
* Retention

---

# Feature 5.3 — Group Analytics

Track:

* Members entering
* Members leaving
* Message activity
* Most active members
* Least active members
* Link interactions
* Common questions
* Common objections

---

# Feature 5.4 — Lead Scoring

Do not initially use arbitrary numeric scoring.

### Phase 1

Collect behavioral signals.

```text
message_received
message_read
question_asked
price_question
checkout_clicked
checkout_started
purchase
```

### Phase 2

Use rules:

```text
HOT

checkout_started

WARM

price_question
multiple interactions

COLD

low interaction
```

### Phase 3

Calibrate against historical conversion.

---

# Feature 5.5 — AI Observability

Every AI run must capture:

```text
ai_run_id
agent
model
prompt_version
input
output
retrieval_context
tools_called
latency
tokens
cost
conversation_id
evaluation_result
```

---

# Feature 5.6 — AI Evaluation

Every AI agent requires a regression suite.

Examples:

```text
Happy path

Price objection

Trust objection

Unknown question

Unsupported question

Payment question

Repeated message

Human escalation

Missing data

Incorrect retrieval
```

Metrics:

* Correctness
* Retrieval relevance
* Hallucination rate
* Tool-call accuracy
* Handoff accuracy
* Policy adherence
* Response latency
* Cost

---

# 15. Non-Functional Requirements

## Reliability

* Idempotent external actions
* Retry policies
* Timeout strategies
* Workflow failure handling
* Dead-letter handling where necessary
* Recovery procedures

## Security

* Secrets management
* HTTPS
* RBAC
* Audit logs
* Data retention rules
* Sensitive-data handling
* LGPD review

## Performance

Initial target:

```text
< 5 seconds normal AI response
< 15 seconds RAG-supported response
```

Exact SLAs should be measured during staging.

## Scalability

The architecture must support:

* Increasing n8n workers
* Increasing database capacity
* Redis-based queue mode
* Separate services where load requires it

---

# 16. MVP Definition

## MVP Goal

Prove that AI can receive a WhatsApp lead, conduct a controlled sales conversation, generate checkout, detect purchase, and escalate when necessary.

### MVP Architecture

```text
WhatsApp
    │
    ▼
Chatwoot
    │
    ▼
n8n
    │
    ▼
AI SDR
 │ │ │
 │ │ └──── Human
 │ │
 │ └──── Knowledge
 │
 └──── Checkout
          │
          ▼
    Payment Gateway
          │
          ▼
     PostgreSQL
          │
          ▼
        Events
```

---

# 17. MVP User Stories

## Story 1 — Receive Lead

As the platform,
I want incoming WhatsApp messages converted into structured leads.

### Acceptance Criteria

* Contact created or matched
* Lead created or matched
* Conversation correlated
* Event recorded

---

## Story 2 — AI Conversation

As a lead,
I want the AI to understand my questions and respond appropriately.

### Acceptance Criteria

* AI understands supported product questions
* Unsupported questions trigger fallback
* Conversation is stored

---

## Story 3 — Qualification

As the sales team,
I want AI to qualify leads.

### Acceptance Criteria

* Required questions captured
* Qualification persisted
* Lead state updated

---

## Story 4 — Objection Handling

As a lead,
I want answers to common purchase objections.

### Acceptance Criteria

* Objection classified
* Approved response strategy used
* Interaction tracked

---

## Story 5 — Checkout

As a qualified lead,
I want to receive checkout automatically.

### Acceptance Criteria

* Checkout generated
* Order created
* Lead associated with checkout

---

## Story 6 — Payment Confirmation

As the platform,
I want payment events to update customer state automatically.

### Acceptance Criteria

* Gateway webhook received
* Payment stored
* Order updated
* Lead converted

---

## Story 7 — Follow-Up

As the sales team,
I want abandoned leads automatically followed up.

### Acceptance Criteria

* Follow-up created
* Message sent when eligible
* Conversion cancels pending follow-up

---

## Story 8 — Human Handoff

As a customer,
I want access to a human when AI cannot resolve my request.

### Acceptance Criteria

* Conversation assigned
* AI pauses
* Handoff reason stored

---

## Story 9 — Telemetry

As the business,
I want every significant sales action measured.

### Acceptance Criteria

Events exist for:

* Lead creation
* Qualification
* Offer
* Checkout
* Payment
* Follow-up
* Handoff

---

# 18. MVP Exit Criteria

The MVP is complete when:

```text
□ WhatsApp is connected
□ Chatwoot is operational
□ Leads are persisted
□ AI SDR works
□ Qualification works
□ Objection handling works
□ Human handoff works
□ Checkout works
□ Payment webhook works
□ Follow-up works
□ Events are recorded
□ AI evaluation suite passes
□ Production monitoring exists
```

---

# 19. Delivery Roadmap

## Weeks 1–2

### Foundation

* Business discovery
* Architecture
* PostgreSQL
* Chatwoot
* n8n
* Event model
* Security baseline
* WhatsApp integration

---

## Weeks 3–4

### Revenue MVP

* Lead ingestion
* Lead state machine
* AI SDR
* Qualification
* Objections
* Checkout
* Payment confirmation
* Follow-up
* Human handoff
* AI evaluation

### Milestone

**MVP GO LIVE**

---

## Weeks 5–6

### Student Support

* Knowledge ingestion
* Vector store
* RAG
* Student Support AI
* Human escalation
* Support analytics

---

## Weeks 7–8

### Customer Success

* Student onboarding
* Application integration
* Engagement events
* Inactivity rules
* Re-engagement

---

## Weeks 9–10

### Financial Operations

* Payment proof verification
* Collections
* Reconciliation
* Human approval

---

## Weeks 11–12

### Intelligence & Hardening

* Sales dashboard
* Student dashboard
* Group analytics
* Lead scoring
* Student scoring
* Performance testing
* Failure testing
* Backup testing
* Operational runbooks

---

# 20. Definition of Done

Every story must satisfy:

```text
□ Requirement documented
□ Acceptance criteria defined
□ Architecture impact reviewed
□ Code/workflow reviewed
□ DEV validation complete
□ Integration test complete
□ Failure scenario tested
□ Logging implemented
□ Events implemented
□ Documentation updated
□ STAGING validated
□ Business validation complete
□ Production deployed
```

For AI stories:

```text
□ Prompt versioned
□ Evaluation dataset exists
□ Regression evaluation passed
□ Tool calls tested
□ Fallback tested
□ Human handoff tested
□ Cost measured
□ Latency measured
```

---

# 21. Suggested Azure Boards Structure

```text
EPIC

AI Revenue

   ↓

FEATURE

Automated Sales Follow-Up

   ↓

USER STORY

Recover abandoned leads

   ↓

TASKS

Create follow-up schema

Create scheduler

Create eligibility rule

Create WhatsApp workflow

Implement cancellation

Implement retries

Implement telemetry

Create tests
```

Suggested statuses:

```text
Backlog
   ↓
Ready
   ↓
In Progress
   ↓
Review
   ↓
Testing
   ↓
Staging
   ↓
Business Validation
   ↓
Done
```

Suggested tags:

```text
MVP

Post-MVP

Production-Critical

Technical-Debt

AI

Data

Integration

Infrastructure
```

---

# 22. Primary Success Metrics

## Sales

* AI handled leads
* AI-assisted conversion
* Conversion rate
* Checkout rate
* Follow-up recovery rate
* Revenue influenced by AI
* Human handoff rate

## Support

* AI resolution rate
* Human escalation rate
* Average response time
* Repeated question rate
* Support cost per interaction

## Customer Success

* Onboarding completion
* Active students
* Inactive students
* Reactivation rate
* At-risk students

## Payments

* Automated payment confirmation
* Reconciliation success rate
* Collection recovery rate
* Manual review rate

## AI

* Hallucination rate
* Tool-call accuracy
* Retrieval relevance
* Cost per conversation
* Latency
* Handoff accuracy

---

# 23. End State

The final system should behave as an integrated operational platform rather than a collection of disconnected automations.

```text
                    AI OPERATIONS PLATFORM
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
       REVENUE            EXPERIENCE        OPERATIONS
          │                  │                  │
       AI SDR          AI Support          Payments
       Follow-Up           RAG             Collections
       Checkout       Customer Success     Human Ops
          │                  │                  │
          └──────────────────┼──────────────────┘
                             │
                      SHARED PLATFORM
                             │
        Chatwoot / n8n / PostgreSQL / Redis
                             │
                             ▼
                        EVENT LAYER
                             │
                             ▼
                 ANALYTICS + AI EVALUATION
```

The platform is successful when the business can measure not only how many workflows exist, but how much revenue, operational efficiency, customer engagement, and service quality those workflows produce.
