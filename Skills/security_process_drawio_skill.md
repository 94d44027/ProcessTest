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
- Use valid, well-formed XML (see **XML Well-formedness Checklist**).
- Open in diagrams.net / draw.io, including when pasted via **Extras > Edit Diagram**.
- Contain one diagram page unless the user requests multiple pages.
- Use a portrait page layout by default.
- Use uncompressed XML where possible so the result is reviewable and version-controllable.
- Use readable labels, consistent spacing, and non-overlapping shapes.

## Diagram Style
Use BPMN-like visual conventions, but do not claim strict BPMN 2.0 compliance unless explicitly requested. Use these visual mappings:
- Start event → circle (ellipse).
- End event → circle (ellipse) with a thicker border.
- Task or activity → rounded rectangle.
- Decision or gateway → diamond.
- Summing junction (merge of two or more sequence flows back into the main flow) → small circle containing an X (`mxgraph.flowchart.or`).
- Subprocess or referenced process → process shape (rectangle with double vertical bars).
- Document, ticket, evidence, or record → document shape.
- System, queue, or datastore → cylinder or database shape.
- Manual interaction → task shape with label beginning with an action verb.
- Escalation or high-risk path → connector with a distinct stroke (`strokeWidth=2`).
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
5. Incident Manager / EMT
6. L3 Response / Forensics (split into "L3 Response" and "L3 Analysis / Forensics" only when they are separate teams)
7. Risk / Compliance / Post-Incident Review

Incident Manager / EMT sits above L3 so that P1/P2 escalation flows downward into the response lanes instead of looping back up.

### Title and Swimlane Structure
- The process title is a **separate horizontal cell** at the top of the page (`parent="1"`), spanning the full lane width. It is not a pool label.
- Lanes are **flat, top-level swimlanes** (`parent="1"`), stacked top-to-bottom directly below the title. Do **not** wrap lanes in a pool and do **not** nest lanes inside one another.
- Lane names are **vertical, on the left**: `swimlane;horizontal=0;startSize=30;collapsible=0;html=1;`.
- All lanes share the **same `x` and the same `width`**.
- Lane order follows their `mxGeometry y` value from top to bottom, matching **Lane Ordering** above.
- Lane **heights vary** with content: lane height = number of grid rows in that lane × 120 (see **Placement Grid**). This deliberately deviates from the draw.io reference's fixed 150 px lanes, which assume one row of nodes per lane in a left-to-right flow.

### Placement Grid
Use this grid for every new diagram (page coordinates; lanes at `x=40`, `width=820`):

| Item | Value |
|------|-------|
| Escalation column (left) | `x=120` |
| Main path column (centre) | `x=380` |
| Exception column (right) | `x=640` |
| Column width | 140 (centre line at `x+70`) |
| Row pitch | 120 |
| Title bar | `x=40, y=20, width=820, height=50` |
| First lane | `y=90` |
| Node `y` | `lane_y + row*120 + (120 - node_height)/2` |

- Every node occupies exactly one `(column, row)` cell. Two nodes never share a cell.
- Shapes narrower than 140 are centred in their column: `x = column_x + (140 - node_width)/2`.
- Inserting a step = add one row to its lane, then shift every lane (and node) below it by +120.
- A fourth parallel branch requires an explicit, documented extra column; do not squeeze it between existing columns.
- Existing diagrams with hand-placed coordinates may keep them; re-flowing them onto the grid is a separate change.

### Node Parenting
- Nodes may live **inside a lane** (`parent="<lane_id>"`, coordinates relative to the lane's top-left corner, so `x_rel = page_x - 40`, `y_rel = row*120 + (120 - node_height)/2`) or **at the page level** (`parent="1"`, page coordinates).
- **Prefer inside-lane parenting for new diagrams** — nodes move with their lane when lanes are resized or reordered. Page-level parenting remains valid for existing files.
- Do not mix both styles within one lane.

### Flow Direction
Place the process top-to-bottom:
- Start events on the very top.
- End events on the very bottom.
- Main path should be vertical, in the centre column.
- Exceptions should branch right (exception column).
- Escalations should branch left (escalation column).
- Rework loops should return backward with clearly labeled connectors.
- When two or more diverged paths re-converge into the main flow, insert a **summing junction** at the convergence point and route all incoming edges into it, then continue with a single outgoing edge.
- Avoid crossing connectors where possible: first try re-placing nodes on the grid. If connectors must still cross, use line jumps (see **Edges**).

## Labeling Rules
Use concise, audit-friendly labels. Every label except swimlane labels, line labels (e.g. Yes/No), summing junctions (which carry no number), evidence/annotation objects and the process title at the top should have a preceding two-digit number, like `01. Start`, `02. Raise ticket`.

Numbering rules:
1. Every numbered label must have `0d` format, followed by `.` and a space, then the actual label content.
2. Numbering follows the **flow order**: start at the start event and follow the sequence flow. When a gateway splits, number one branch completely up to its merge point, then the next branch, then continue after the merge.
3. When steps are inserted, renumber all subsequent steps; there is no need to preserve old numbers. Internal cell `id` values do not have to change — only labels are renumbered.
4. Summing junctions are unnumbered and unlabeled (they are pure routing nodes).

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

Edge labels:
- Keep edge labels to **one to three words** that state the condition. Move explanations into node text or a legend.
- Connector labels are not numbered. Connectors entering or leaving a summing junction do not need labels unless they carry a condition.

Language:
- Write all labels, the title and annotations in the user's language.

### Label Encoding
- Always include `html=1` in every cell style.
- Line breaks: use `&#xa;` (works with or without `html=1`) or escaped `&lt;br&gt;` (requires `html=1`). Never use `\n` — draw.io renders it literally.
- Escape special characters in `value` attributes: `&` → `&amp;`, `<` → `&lt;`, `>` → `&gt;`, `"` → `&quot;`. One bare `&` makes the file unopenable.
- Whole-label bold/italic/underline: use `fontStyle` (`1` bold, `2` italic, `4` underline, combinable, e.g. `3`). Use escaped HTML tags (`&lt;b&gt;…&lt;/b&gt;`) only to format **part** of a label. Never combine both for the same effect.

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

When the process maps to NIST CSF 2.0, append the category code to the task label in parentheses, e.g. `13. Analyse incident&#xa;&amp; collect evidence&#xa;(RS.AN)`.

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
  <diagram name="Process Diagram" id="page-1">
    <mxGraphModel grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="900" pageHeight="2000" math="0" shadow="0">
      <root>
        <mxCell id="0" />
        <mxCell id="1" parent="0" />
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

Place the title bar, lanes, shapes and connectors as `mxCell` elements inside `<root>`, after cell `1`. Set `pageHeight` to at least the bottom of the last lane plus 40.

Every shape must have:
- A unique `id`.
- A `value` (label, may be empty for summing junctions).
- A valid `style` that includes `html=1`.
- `vertex="1"`.
- A `parent`.
- An `mxGeometry` element with `x`, `y`, `width`, and `height`.

Every connector must have:
- A unique `id`.
- `edge="1"`.
- A `source` and a `target`.
- A `parent` (see **Edges**).
- A connector style with an arrow.
- A label if it exits a decision gateway.
- An expanded `<mxGeometry relative="1" as="geometry" />` child element.

### Geometry vs. Style constraints
- `width=` and `height=` MUST NOT appear inside any `style` attribute (title, lane, or any other shape).
- All dimensional values live exclusively inside `<mxGeometry .../>`.

### Edges
- **Every edge is written in expanded form** with a `<mxGeometry relative="1" as="geometry" />` child. Self-closing edge cells (`<mxCell ... edge="1" ... />`) are invalid and may not render.
- **One edge style for the whole diagram:** `edgeStyle=orthogonalEdgeStyle;rounded=1;html=1;endArrow=classic;jumpStyle=arc;jumpSize=14;`. Do not mix in `elbowEdgeStyle` or `curved=1`.
- **Forward flow:** declare only `source` and `target`. Do not add waypoints or `exitX`/`exitY`/`entryX`/`entryY` overrides.
- **Allowed manual routing** (waypoints via `<Array as="points">` and/or exit/entry overrides) only for:
  - rework loops that return to an earlier step (route around the boxes on the right),
  - escalations leaving a gateway to the left,
  - exceptions leaving a gateway to the right,
  - edges entering a summing junction from the side.
- **Line jumps:** `jumpStyle=arc;jumpSize=14;` is on every edge; draw.io shows the 14 px arch only where two edges actually cross. The edge **later in XML order** draws the jump, so list a crossing edge after the edge it jumps over.
- **Parent of an edge:** the innermost container holding **both** endpoints — `parent="<lane_id>"` when both ends are in the same lane, `parent="1"` when the edge crosses lanes or either end is at page level.
- **Line semantics:** solid = sequence flow; `strokeWidth=2` = escalation / high-risk path; `dashed=1` = informational relation (evidence, register update). Give each style exactly one meaning; if more meanings are needed, add a small legend node.

### Optional: Tags, Metadata and Dark Mode
- **Tags** (filter via Edit > Tags) require wrapping the cell in `<object>`; the label moves from `value` to `label`:
  ```xml
  <object id="n13" label="13. Analyse incident&#xa;&amp; collect evidence&#xa;(RS.AN)" tags="RS" csf="RS.AN" owner="L3 Response">
    <mxCell style="rounded=1;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;" vertex="1" parent="laneL3">
      <mxGeometry x="340" y="30" width="140" height="60" as="geometry" />
    </mxCell>
  </object>
  ```
  For CSF 2.0 processes, tag steps with their function: `DE`, `RS`, `RC` (and `GV`, `ID`, `PR` if present).
- **Metadata:** extra attributes on `<object>` (e.g. `csf`, `owner`) store structured data; add `placeholders="1"` to show them in the label via `%csf%`.
- **Dark mode:** add `adaptiveColors="auto"` to `mxGraphModel` to let draw.io invert explicit colours in dark theme. Use `light-dark(#light,#dark)` only when the automatic inversion is unsatisfactory.

### XML Well-formedness Checklist
Before handing over the file, verify:
- [ ] No XML comments (`<!-- -->`) anywhere in the output.
- [ ] Every `id` is unique.
- [ ] Every `source`/`target`/`parent` references an existing `id`.
- [ ] All `&`, `<`, `>`, `"` inside attribute values are escaped.
- [ ] No `\n` in labels; line breaks use `&#xa;`.
- [ ] Every style contains `html=1`.
- [ ] No `width=`/`height=` in any `style`.
- [ ] Every edge has an expanded `<mxGeometry relative="1" as="geometry" />` child.
- [ ] The file parses (e.g. `python3 -c "import xml.etree.ElementTree as ET; ET.parse('file.drawio')"`).

## Shape Style Guide
Use these styles unless the user provides a house style. Sizes follow the **Placement Grid**.

| Element | Style | Size (w×h) |
|---------|-------|------------|
| Title bar | `rounded=0;whiteSpace=wrap;html=1;fillColor=#e1e7ef;strokeColor=#6c8ebf;fontSize=15;fontStyle=1;align=center;verticalAlign=middle;` | 820×50 |
| Lane (shaded header) | `swimlane;horizontal=0;startSize=30;collapsible=0;html=1;fillColor=#f5f5f5;swimlaneFillColor=#ffffff;fontStyle=1;` | 820×(rows×120) |
| Lane (plain header) | `swimlane;horizontal=0;startSize=30;collapsible=0;html=1;swimlaneFillColor=#ffffff;fontStyle=1;` | 820×(rows×120) |
| Start event | `ellipse;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;` | 60×60 (or 120×60 for longer labels) |
| End event | `ellipse;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;strokeWidth=3;` | 60×60 |
| End event (exception, e.g. false positive) | `ellipse;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;strokeWidth=2;` | 60×60 |
| Task | `rounded=1;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;` | 140×60 |
| Escalation task | `rounded=1;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;strokeWidth=2;` | 140×60 |
| Exception / closure task | `rounded=1;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;` | 140×60 |
| Decision | `rhombus;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;` | 140×80 |
| Subprocess | `shape=process;whiteSpace=wrap;html=1;fillColor=#e1d5e7;strokeColor=#9673a6;` | 140×60 |
| Document / ticket / record | `shape=document;whiteSpace=wrap;html=1;boundedLbl=1;fillColor=#fff2cc;strokeColor=#d6b656;` | 120×80 |
| Evidence (informational) | `shape=document;whiteSpace=wrap;html=1;boundedLbl=1;fillColor=#f5f5f5;strokeColor=#666666;` | 120×80 |
| System / datastore | `shape=cylinder3;whiteSpace=wrap;html=1;boundedLbl=1;fillColor=#f5f5f5;strokeColor=#666666;` | 100×70 |
| Summing junction | `shape=mxgraph.flowchart.or;html=1;backgroundOutline=1;` | 30×30 |
| Legend / note | `text;html=1;align=left;verticalAlign=top;whiteSpace=wrap;` | as needed |
| Sequence flow | `edgeStyle=orthogonalEdgeStyle;rounded=1;html=1;endArrow=classic;jumpStyle=arc;jumpSize=14;` | — |
| Escalation flow | sequence flow + `strokeWidth=2;` | — |
| Informational relation | sequence flow + `dashed=1;` | — |

### Example: title, lane, task, decision, edges

```xml
<mxCell id="titleBar" value="Security Incident Monitoring Process&#xa;(NIST CSF 2.0)" style="rounded=0;whiteSpace=wrap;html=1;fillColor=#e1e7ef;strokeColor=#6c8ebf;fontSize=15;fontStyle=1;align=center;verticalAlign=middle;" vertex="1" parent="1">
  <mxGeometry x="40" y="20" width="820" height="50" as="geometry" />
</mxCell>
<mxCell id="laneL2" value="L2 Triage" style="swimlane;horizontal=0;startSize=30;collapsible=0;html=1;swimlaneFillColor=#ffffff;fontStyle=1;" vertex="1" parent="1">
  <mxGeometry x="40" y="90" width="820" height="240" as="geometry" />
</mxCell>
<mxCell id="n08" value="08. Classify severity&#xa;&amp; assign SLA" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;" vertex="1" parent="laneL2">
  <mxGeometry x="340" y="30" width="140" height="60" as="geometry" />
</mxCell>
<mxCell id="n09" value="09. P1/P2&#xa;critical?" style="rhombus;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;" vertex="1" parent="laneL2">
  <mxGeometry x="340" y="140" width="140" height="80" as="geometry" />
</mxCell>
<mxCell id="e_08_09" edge="1" parent="laneL2" source="n08" target="n09" style="edgeStyle=orthogonalEdgeStyle;rounded=1;html=1;endArrow=classic;jumpStyle=arc;jumpSize=14;">
  <mxGeometry relative="1" as="geometry" />
</mxCell>
```

## References
- draw.io XML reference: https://github.com/jgraph/drawio-mcp/blob/main/shared/xml-reference.md
- draw.io style reference: https://github.com/jgraph/drawio-mcp/blob/main/shared/style-reference.md
- mxfile XML Schema (XSD): https://github.com/jgraph/drawio-mcp/blob/main/shared/mxfile.xsd

Not adopted from the reference, by design: fixed 150 px single-row lanes (this skill uses top-to-bottom flow with multi-row lanes), automatic `libavoid` / `elk` passes (available only through the draw.io MCP tool, not via Extras > Edit Diagram), and cross-functional table layouts.
