
Related to [[Sundt]]
#sundt_project_ideas

### Construction Concept Overview


#### Ai Response
This is exactly the right question. If you understand where change orders originate, you’ll understand the architecture of the software much better.

Most change orders are not caused by a single document. They are caused by mismatches between documents, stakeholders, or project phases.

Think of the construction pipeline like this:

Owner Requirements

        ↓

Architect/Engineer Design

        ↓

Drawings + Specifications

        ↓

Estimator Bid

        ↓

Contract

        ↓

Construction

        ↓

Field Conditions

A change order often appears when information changes or conflicts somewhere along that chain.

1. Drawings vs Specifications

This is one of the biggest sources.

Example:

Drawing

Panel A = 400A

Specification

Panel A = 600A

Which is correct?

Someone eventually discovers the discrepancy.

The contractor priced one version.

The owner wants the other.

Result:

Additional material cost

Additional labor

Potential change order

A future AI system could compare drawing-derived information against specification-derived information and flag mismatches.

  

2. Bid Estimate vs Actual Design

Estimator receives:

60% design drawings

Project gets awarded.

Later:

90% drawings released

New scope appears:

additional conduit

additional concrete

additional steel

The estimate did not include this work.

Result:

Potential Change Order (PCO)

Your system could compare:

Bid Documents

vs

Current Documents

and identify additions.

  

3. Contract Scope vs Field Requests

Contract:

Install 20 cameras

Owner later says:

Can we add 8 more cameras?

That’s not in the contract.

Result:

Owner Change Order

The software might analyze meeting notes or emails and detect requests that exceed contractual scope.

  

4. RFIs vs Design Documents

This is where things get interesting.

Example:

RFI says:

Drawing A4.2 conflicts with E5.3.

Please clarify.

Engineer responds:

Proceed per revised detail.

That revised detail adds work.

Now the RFI has effectively created new scope.

Your system should watch for:

RFI response

        ↓

Design revision

        ↓

Added scope

        ↓

Potential change order

  

5. Existing Conditions vs Drawings

Huge source of risk.

Drawings say:

Existing wall is 8 inches thick.

Field discovers:

Existing wall is 18 inches thick.

Now:

different anchors

different drilling

additional labor

Result:

Change order request

This is especially common in renovations.

  

6. Meeting Minutes vs Contract

Meeting notes might contain statements like:

Architect requested revised ceiling layout.

or

Owner requested alternate finish.

Nobody formally issued a change order yet.

But an experienced PM immediately sees risk.

A good AI system would flag:

Requested work

without

approved change order

  

7. Schedule vs Procurement

Example:

Submittal approved late.

Equipment now has:

18 week lead time

Project only has:

10 weeks remaining

Now acceleration is required.

Result:

Overtime

expediting

additional cost

Potential change order.

  

What your internship project may actually analyze

If I were designing a first-generation system for a construction company, I’d focus on:

RFIs

Meeting Minutes

Change Order Logs

Submittal Logs

Drawing Revisions

because those are usually available in digital form.

The pipeline would look something like:

PDFs

Emails

Meeting Minutes

RFI Logs

Drawing Revision Logs

        ↓

Document Parser

        ↓

Entity Extraction

        ↓

Risk Rules

        ↓

Risk Database

        ↓

Dashboard

  

Example risk detection rule

Imagine this appears in meeting minutes:

Owner requested relocation of electrical panels.

The AI extracts:

{

  "request_type": "scope_change",

  "discipline": "electrical",

  "source": "meeting_minutes_05_12"

}

Then it checks:

Has a change order been issued?

No.

Risk score increases.

Risk: Unpriced Owner Request

Score: 82/100

This is the type of system that provides immediate value because it doesn’t require sophisticated machine learning initially. It’s mostly about connecting information across documents and identifying where reality is diverging from the original contract, estimate, or design. That’s where most change-order risk lives.

### MVP goal

#### AI Response

For an internship project, the biggest mistake would be trying to build a full AI platform.

A good MVP should be:

- Solvable in 8–12 weeks
- Useful on day one
- Understandable by construction personnel
- Extensible later into a larger system

What I would build

MVP: Change Order Risk Scanner

Input:

- RFI log (CSV/Excel)
- Meeting minutes (PDF/text)
- Change order log (CSV/Excel)

Output:

Project: XYZ Hospital

  

Top Risks:

  

1. Electrical Scope Clarification

   Risk Score: 85

   Sources:

      - RFI-104

      - Meeting Minutes 5/12

  

   Reason:

      Open for 24 days.

      Architect clarification pending.

      Electrical work scheduled in 7 days.

  

2. Structural Revision

   Risk Score: 78

   Sources:

      - RFI-122

      - Drawing Revision 6

  

   Reason:

      New detail adds steel not included in original estimate.

This immediately provides value.

  

Architecture

Step 1 — Ingest Documents

Python reads:

RFI Log.xlsx

MeetingMinutes.pdf

ChangeOrders.xlsx

Libraries:

pandas

pdfplumber

pydantic

  

Step 2 — Extract Structured Data

Convert documents into common objects:

class RFI:

    id: str

    status: str

    days_open: int

    discipline: str

class RiskFinding:

    source: str

    score: int

    reason: str

  

Step 3 — Rule Engine

Initially, no machine learning.

Example:

if rfi.days_open > 14:

    score += 20

  

if rfi.status == "Open":

    score += 20

  

if discipline == "Electrical":

    score += 10

Construction people trust explicit rules much more than a mysterious model.

  

Step 4 — LLM Summarization

After rules identify risks:

RFI-104

Open 24 days

Electrical

Work starts next week

Send to LLM:

Explain why this may become a change-order risk.

Now you get readable output.

  

Step 5 — Dashboard

Simple web UI.

Stack:

- FastAPI
- HTML
- TypeScript (if the team uses it)
- SQLite or PostgreSQL

Display:

|   |   |   |
|---|---|---|
|Risk|Score|Source|
|Electrical Scope|85|RFI-104|
|Structural Revision|78|RFI-122|

Clicking opens supporting documents.

  

What NOT to build

Avoid:

Drawing comparison AI

PDF drawings

CAD files

BIM models

Extremely difficult.

  

Predictive machine learning

Predict future change orders

Requires years of historical data.

Likely unavailable.

  

Agent swarm architectures

A common AI-engineer trap:

RFI Agent

Meeting Agent

Estimator Agent

Schedule Agent

Looks impressive.

Usually unnecessary for an MVP.

  

What would impress your manager

Not:

Look at my fancy LLM architecture.

Instead:

I processed 500 RFIs,

found 37 unresolved risks,

and reduced review time from 4 hours to 15 minutes.

Construction companies care about measurable operational value.

  

If I were your mentor

I would define the internship phases like this:

Phase 1

Learn:

- Python
- Pandas
- Pydantic
- FastAPI
- Construction workflow

Deliverable:

Read RFI logs

Generate risk report

  

Phase 2

Add:

- Meeting minutes parsing
- PDF extraction
- Risk scoring

Deliverable:

Cross-reference RFIs and meeting notes

  

Phase 3

Add:

- LLM explanations
- Search capability
- Web dashboard

Deliverable:

Interactive risk review tool

  

Phase 4 (stretch goal)

Add retrieval over project documents:

"Why is RFI-104 considered high risk?"

The system retrieves the supporting meeting minutes, RFI text, and prior change-order history and explains the reasoning.

  

That is a realistic internship-scale project that teaches Python, AI systems, document processing, APIs, databases, and construction operations without requiring a huge research effort.