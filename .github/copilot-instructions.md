# SVG Plan Annotation for Interactive Installations

## Purpose

This document describes a workflow for annotating floor plans and elevation drawings as SVG files, paired with a markdown context file. Together, these files provide a coding agent (like Claude) with the spatial and semantic information needed to write software for tracking-based interactive installations.

The goal is to give the agent everything it needs to:
- Understand the physical layout and coordinate system
- Map tracking data to spatial zones
- Address and control output devices (panels, lights, etc.)
- Implement interaction logic based on your design intent

---

## File Structure

Your annotation produces two files:

```
project-name.svg        # Annotated plan drawing
project-name-context.md # Documentation for coding agent
```

Always provide both files together when working with an agent.

---

## SVG Annotation Requirements

### 1. Element IDs

Every element the agent needs to reference must have a unique `id` attribute.

```xml
<rect id="zone-1" ... />
<rect id="panel-A1" ... />
<line id="mullion-left" ... />
```

**Naming conventions:**
- Use lowercase with hyphens: `zone-1`, `panel-north-3`
- Include type prefix when helpful: `zone-`, `panel-`, `column-`
- Be consistent within element types

### 2. Data Attributes

Use `data-*` attributes to add semantic meaning:

| Attribute | Purpose | Example Values |
|-----------|---------|----------------|
| `data-type` | Element category | `zone`, `panel`, `architecture`, `sensor`, `boundary` |
| `data-zone` | Zone assignment | `zone-1`, `entrance`, `hotspot-a` |
| `data-layer` | Drawing layer | `tracking`, `output`, `structure`, `reference` |
| `data-group` | Logical grouping | `north-wall-panels`, `window-1` |
| `data-address` | Control address | `1`, `DMX:1:001`, `0x0A` |
| `data-tags` | Comma-separated tags | `interactive,primary,dimmable` |

Example:
```xml
<rect id="panel-3" 
      data-type="panel" 
      data-zone="zone-1" 
      data-layer="output"
      data-address="3"
      data-tags="led,dimmable"
      x="100" y="50" width="30" height="200" />
```

### 3. Root SVG Attributes

Add these attributes to the root `<svg>` element:

```xml
<svg xmlns="http://www.w3.org/2000/svg"
     data-annotator-version="1.0"
     data-origin-x="0"
     data-origin-y="500"
     data-scale-px-per-meter="100"
     data-groups='[{"id":"g1","name":"Zone 1 Panels","elements":["panel-1","panel-2"]}]'>
```

| Attribute | Description |
|-----------|-------------|
| `data-origin-x`, `data-origin-y` | The (0,0) point in SVG coordinates |
| `data-scale-px-per-meter` | Pixels per meter (or per unit) |
| `data-groups` | JSON array of logical groupings |

### 4. Coordinate System

SVG uses a top-left origin with Y increasing downward. Your annotation should clarify how this maps to the physical space.

**In your context file, specify:**
- Where the origin is located physically (e.g., "left edge of storefront, floor level")
- Axis orientation (e.g., "+X runs right along the window, +Y extends toward the street")
- Any transforms needed between tracker coordinates and SVG coordinates

### 5. Scale Reference

Identify one element with a known real-world dimension. Mark it:

```xml
<line id="scale-reference" 
      data-scale-reference="true"
      x1="0" y1="0" x2="100" y2="0" />
<!-- This line represents 1 meter -->
```

Then set `data-scale-px-per-meter` on the root SVG accordingly.

---

## Context File Structure (Markdown)

The context file provides semantic information the SVG alone cannot convey. Use this template:

```markdown
# [Project Name]

## Overview

[Brief description of the installation, its purpose, and interaction goals]

## Spatial Configuration

**Origin:** [Physical location of (0,0)]
**Scale:** 1m = [X] px
**Orientation:** [How SVG axes map to physical space]

### Coordinate System

[Detailed description of coordinate mapping, especially between 
tracking system output and SVG coordinates. Include any transforms.]

## Tracking System

**Method:** [YOLO + Camera / LiDAR / mmWave / etc.]

[Details: camera position, field of view, resolution, update rate, 
coordinate output format, confidence thresholds]

## Element Types

### `zone`

[What zones represent, how they should be used in code]

### `panel`

[What panels are, control interface, addressing scheme]

### `architecture`

[Structural elements - how code should treat them (obstacles, boundaries, etc.)]

## Zones

### zone-1

[Purpose, expected behavior, interaction rules for this zone]

### zone-2

[...]

## Element Inventory

| ID | Type | data-type | data-zone | data-address | Notes |
|----|------|-----------|-----------|--------------|-------|
| panel-1 | rect | panel | zone-1 | 1 | North wall |
| ... | | | | | |

## Groups

### [Group Name]

**Elements:** [list]
**Purpose:** [why these are grouped]

## Interaction Rules

[Describe the expected behaviors:]
- Entry/exit triggers
- Proximity responses
- Multi-person handling
- Timing and animation expectations
- State machine if applicable
- Edge cases

## Data Structures

```javascript
// Tracking data format your system receives:
{
  timestamp: number,
  people: [{
    id: string,
    x: number,  // meters from origin
    y: number,
    velocity?: { x: number, y: number },
    confidence: number
  }]
}
```

## Panel Control Interface

**Protocol:** [DMX / Art-Net / sACN / 0-10V / MQTT / OSC / etc.]

[Address mapping, channel assignments, value ranges, 
universe info, timing constraints]

## Additional Notes

[Edge cases, performance constraints, existing code to integrate with,
physical constraints, testing/simulation notes]

---

*This documentation accompanies [filename].svg*
```

---

## Best Practices

### For SVG Creation

1. **Keep it simple** - Only include elements relevant to the interaction. Remove decorative elements or put them in a separate non-annotated layer.

2. **Use groups wisely** - SVG `<g>` elements can organize related items, but ensure individual elements still have their own IDs if they need individual control.

3. **Consistent units** - Pick a scale and stick to it. Document it clearly.

4. **Minimal styling** - Fill colors can indicate zones visually, but avoid complex gradients or effects that obscure the geometry.

5. **Validate coordinates** - Check that element positions make sense relative to your origin and scale.

### For Documentation

1. **Be specific about coordinates** - Ambiguity here causes bugs. State exactly how tracking coordinates map to your plan.

2. **Define all data-type values** - Every unique value used should have a description.

3. **Describe behaviors, not just layout** - The agent needs to know *what should happen*, not just *where things are*.

4. **Include edge cases** - What happens at zone boundaries? With multiple people? When tracking is lost?

5. **Specify timing** - Animation durations, fade times, update rates, debounce intervals.

### For Providing Files to an Agent

1. **Upload both files together** - The SVG and context markdown are meant to be read as a pair.

2. **Reference element IDs in your prompts** - "When someone enters zone-3, panels panel-5 through panel-8 should..."

3. **Iterate on the context file** - If the agent misunderstands something, clarify it in the documentation for next time.

---

## Example Workflow

1. **Create base SVG** - Export your floor plan or elevation from CAD/design software

2. **Clean up** - Remove unnecessary elements, flatten complex groups if needed

3. **Add IDs** - Name every interactive element

4. **Add data attributes** - Tag elements with type, zone, layer, address

5. **Set origin and scale** - Choose a reference point and measure a known distance

6. **Write context file** - Document coordinate system, element types, behaviors

7. **Test with agent** - Ask the agent to describe what it sees, verify understanding

8. **Refine** - Update annotations and documentation based on agent feedback

---

## Common Element Types

| Type | Typical Use |
|------|-------------|
| `zone` | Tracking/detection regions, trigger areas |
| `panel` | Light panels, LED strips, controllable outputs |
| `sensor` | Camera positions, radar units, input devices |
| `architecture` | Walls, columns, mullions (obstacles/boundaries) |
| `boundary` | Tracking area limits, exclusion zones |
| `reference` | Scale bars, alignment marks (not interactive) |
| `furniture` | Obstacles within the space |
| `path` | Expected movement corridors |
| `hotspot` | High-interest areas for special behavior |

---

## Troubleshooting

**Agent misreads coordinates:**
- Check origin position and axis orientation in documentation
- Verify scale calculation
- Confirm tracking system coordinate format matches documentation

**Agent can't find elements:**
- Ensure elements have IDs
- Check that IDs are unique
- Verify elements aren't nested inside groups without IDs

**Behaviors don't match intent:**
- Add more detail to Interaction Rules section
- Include specific examples
- Define state machine explicitly if complex

**Addressing problems:**
- Verify `data-address` values match hardware configuration
- Document address format explicitly (decimal, hex, universe:channel)

---

## Quick Reference: Data Attributes

```xml
data-type="zone|panel|sensor|architecture|boundary|reference"
data-zone="[zone-id]"
data-layer="tracking|output|structure|reference"
data-group="[group-id]"
data-address="[control-address]"
data-tags="[comma,separated,tags]"
data-scale-reference="true"
```

## Quick Reference: Root SVG Attributes

```xml
data-annotator-version="1.0"
data-origin-x="[number]"
data-origin-y="[number]"  
data-scale-px-per-meter="[number]"
data-groups="[json-array]"
```