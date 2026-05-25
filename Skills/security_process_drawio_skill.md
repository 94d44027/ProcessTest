# Skill: BPMN-like Process Diagram Generator for draw.io

## Purpose
Create clear, audit-friendly process diagrams in BPMN-like style and output them as draw.io-readable `.drawio` XML files. Use this skill when the user provides:
- A process description
- A screenshot or PDF of an existing process
- A list of process steps
- A target-state or current-state workflow
- A security, IT, risk, operations, or compliance process

The goals are:
- To transform the source material into a readable process diagram with lanes, events, tasks, gateways, subprocess references, evidence objects, sequence flows, and summing junctions.
- To update the diagram following the user's verbal input (for example, "add another step after step X") with an immediate visual result observable by the user.

## Output Requirements
Always produce a valid `.drawio` file unless the user asks for another format. The `.drawio` file must:
- Use valid XML.
- Open in diagrams.net / draw.io.
- Contain one diagram page unless the user requests multiple pages.
- Use a portrait page layout by default.
- Use uncompressed XML where possible so the result is reviewable and version-controllable.
- Use readable labels, consistent spacing, and non-overlapping shapes.

## Diagram Style
Use BPMN-like visual conventions, but do not claim strict BPMN 2.0 compliance unless explicitly requested. Use these visual mappings:
- Start event → circle (ellipse).
- End event → circle (ellipse).
- Task or activity → rounded rectangle.
- Decision or gateway → diamond.
- Summing junction (merge of two or more sequence flows back into the main flow) → small circle containing an X (`mxgraph.flowchart.or`).
- Subprocess or referenced process → process shape or rounded rectangle.
- Document, ticket, evidence, or record → document shape.
- System, queue, or datastore → cylinder or database shape.
- Manual interaction → task shape with label beginning with an action verb.
- Escalation or high-risk path → connector or callout with a distinct stroke.
- Normal sequence flow → solid arrow.
- Optional or informational relation → dashed connector.

BPMN uses events, tasks, and gateways as core flow elements, while gateways control how process paths split or merge. A summing junction is used at the point where two or more previously diverged paths re-converge into one continuing path.

## Layout Rules

### Lane Ordering
Create swimlanes whenever the process includes roles, teams, systems, departments, or external parties.

Default lane order:
1. External requester / source
2. Front-line team or intake function
3. System of record
4. Specialist team
5. Manager / escalation body
6. Governance, risk, compliance, or post-analysis team

For IT security processes, prefer this lane order:
1. Incident source
2. Service Desk L1
3. Ticketing / SIEM / SOAR system
4. L2 Triage
5. L3 Response
6. L3 Analysis / Forensics
7. Incident Manager / EMT
8. Risk / Compliance / Post-Incident Review

### Swimlane Sizing and Stacking
- Lanes are **stacked top-to-bottom inside the pool**, never nested inside one another.
- All lanes share the **same horizontal width**, equal to the pool's content width.
- Each lane's **vertical height** is sized to fit its contained shapes (the bounding box of the lane's children) plus a small padding. Do not use a fixed lane height across lanes.
- The pool's height equals `startSize + Σ(lane heights)`.
- Lanes are siblings under the pool, ordered by their `mxGeometry y` value from top to bottom, matching the lane ordering above.

### Flow Direction
Place the process top-to-bottom:
- Start events on the very top.
- End events on the very bottom.
- Main path should be vertical.
- Exceptions should branch right.
- Escalations should branch left.
- Rework loops should return backward with clearly labeled connectors.
- When two or more diverged paths re-converge into the main flow, insert a **summing junction** at the convergence point and route all incoming edges into it, then continue with a single outgoing edge.
- Avoid crossing connectors where possible. If line connectors have to be crossed, use clearly distinct line jumps (arches of 14 px diameter).

## Labeling Rules
Use concise, audit-friendly labels. Every label except swimlane labels, line labels (e.g. Yes/No), summing junctions (which carry no number) and the process name at the top should have a preceding two-digit number, like `01. Start`, `02. Raise ticket`.

Numbering rules:
1. Every numbered label must have `0d` format, followed by `.` and a space, then the actual label content.
2. Numbering starts at the upper-left corner of the diagram and continues right and downward.
3. Summing junctions are unnumbered and unlabeled (they are pure routing nodes).

Good labels:
- Raise ticket
- Classify severity
- Investigate incident
- P1 or P2 priority?
- Mobilise EMT
- Close ticket
- Post-analysis required?

Avoid vague labels:
- Handle it
- Check
- Do stuff
- Review
- Process

Decision labels must be phrased as questions:
- Incident solved?
- P1 or P2 priority?
- Evidence complete?
- Client approval required?

Outgoing decision connectors must be labeled:
- Yes
- No
- P1/P2
- P3/P4
- Approved
- Rejected
- Timeout
- Duplicate

Note: connector labels are not numbered. Connectors entering or leaving a summing junction do not need labels unless they carry a condition.

## Security Process Conventions
When drawing IT security workflows, include control points where relevant:
- Ticket creation
- Severity classification
- SLA assignment
- Evidence capture
- Containment decision
- Escalation decision
- Emergency management team mobilisation
- Remediation
- Recovery validation
- Root cause analysis
- Lessons learned
- Risk register update
- Closure approval

Use neutral placeholders for sensitive data:
- CLIENT
- SYSTEM
- HOST
- USER
- TICKET-ID
- SIEM alert
- EDR alert
- Case record

Never include:
- Real credentials
- API keys
- IP addresses
- Hostnames
- Client names
- Personal data
- Vulnerability details that create operational risk
- Exploit steps

## Input Interpretation

When the source is a PDF, screenshot, or rough process map:
1. Extract all visible text.
2. Identify lanes.
3. Identify starts and ends.
4. Identify numbered steps.
5. Identify decision points.
6. Identify subprocess references.
7. Identify connectors and labels.
8. Identify convergence points where two or more paths re-merge — insert a summing junction at each.
9. Reconstruct the most likely flow.
10. Preserve original numbering where present.
11. If flow direction is ambiguous, choose the clearest top-to-bottom interpretation and note assumptions.

Note: if more numbered steps are inserted, there is no need to preserve the old numbers. New numbers may be used.

When the source is prose:
1. Identify actors.
2. Identify systems.
3. Identify trigger events.
4. Convert verbs into tasks.
5. Convert conditions into gateways.
6. Convert records into document shapes.
7. Convert referenced processes into subprocess shapes.
8. Insert summing junctions where multiple paths re-converge.
9. Add start and end events.
10. Add exception branches if described.

## Clarification Policy
Ask one clarification question only if the missing information would materially change the diagram. Ask if:
- The process has no clear scope.
- The actor / lane structure is missing.
- The user asks for strict BPMN 2.0 compliance.
- The source process has conflicting flows.
- The expected output format is unclear.

Do not ask if:
- A reasonable BPMN-like diagram can be created from the available material.
- The user asks for a sample.
- The user provides an existing process diagram and wants conversion.
- The ambiguity can be handled with a documented assumption.

## draw.io XML Generation Rules
Generate a complete `.drawio` XML file using the following structure:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<mxfile host="app.diagrams.net" agent="ProcessDiagramSkill">
  <diagram name="Process Diagram">
    <mxGraphModel grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="850" pageHeight="1100">
      <root>
        <mxCell id="0" />
        <mxCell id="1" parent="0" />
        <!-- shapes and connectors here -->
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

Every shape must have:
- A unique `id`.
- A `value` (label, may be empty for summing junctions).
- A valid `style`.
- `vertex="1"`.
- A `parent`.
- An `mxGeometry` element with `x`, `y`, `width`, and `height`.

Every connector must have:
- A unique `id`.
- `edge="1"`.
- A `source`.
- A `target`.
- A connector style with an arrow.
- A label if it exits a decision gateway.

### Geometry vs. Style constraints
- `width=` and `height=` MUST NOT appear inside any `style` attribute (pool, lane, or any other shape).
- All dimensional values live exclusively inside `<mxGeometry .../>`.
- The pool's `style` MUST NOT contain `width=` or `height=`.
- Each lane's `style` MUST NOT contain `width=` or `height=`.

## Shape Style Guide
Use these styles unless the user provides a house style. Styles below are taken from the reference diagram and are minimal (no fill / stroke colour overrides) so they inherit the current theme.

### Start Event
