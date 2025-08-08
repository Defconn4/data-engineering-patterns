# Advanced Search User Guide

## 1. Introduction

The Advanced Search allows you to build powerful queries that filter submissions by various criteria. Whether you need to find submissions by name, classification, location (MGRS), or tabbed text content, this guide will walk you through each step.

## 2. Overview of Key Features

### Logical Operators

- **AND:** All conditions must be true.
- **OR:** At least one condition must be true.
- You can combine multiple conditions (and even multiple groups of conditions) with AND/OR.

### Metadata vs. Custom Components

- **Metadata:** Standard fields like submission name, creation date, etc.
- **Custom Components:** Specialized fields (e.g., MGRS coordinates, classification, tabbed text) stored in a separate fields table.

### Regex Matching (POSIX)

- Regex can be used with the “Matches” operator (`~`).
- Use advanced patterns (anchors, groups, etc.) if you’re comfortable with POSIX Basic Regular Expressions.

### Radius-Based MGRS Searches

- Specify a center MGRS coordinate and a radius in kilometers (converted to meters automatically).
- The system uses PostGIS to return submissions within that distance.

### Tabbed Text Editor (TTE) Content

- TTE content is broken into named tabs (e.g., “Summary”, “Report Text”).
- You can search across these tabs, and the system ensures no duplicate entries if the same text is repeated.

## 3. Constructing a Query

### 3.1 Selecting a Field

- Pick the field you want to filter on (e.g., “Name”, “ClassificationComponent”, “MgrsComponent”).
- Choose an operator (e.g., “equals”, “matches”, “greater_than”, etc.).
- **Tip:** Some operators don’t apply to certain field types (e.g., “greater_than” doesn’t make sense for text).

### 3.2 Combining Multiple Conditions

- Add more conditions within a single group. You can choose to connect them with AND or OR.
- If you want separate logical groupings, add another group. Each group can have its own AND/OR logic, and you can then choose how groups themselves are combined (AND/OR).

#### Example

- **Condition A:** “Name equals ‘Sample Submission’”
- **Condition B:** “ClassificationComponent equals ‘TS’”
- **Combined with AND:** “Find submissions named ‘Sample Submission’ **AND** with classification ‘TS’.”

## 4. Using Regex (Matches Operator)

- Enable the “Regex” toggle (if your interface provides one) or select the “matches” operator.
- Enter a POSIX-compatible regex pattern. For instance:
  - `^foo(bar|peep)$` matches exactly "foobar" or "foopeep".
  - `.*report.*` (with anchors like `^` or `$`) can help you find text containing “report.”
  - **Tip:** By default, the "Regex" toggle performs a "fuzzy" search.

**Common Pitfalls:**

- Ensure your pattern is correct. A small typo can yield no results.
- Keep patterns within a reasonable length; overly complex patterns might slow the search.

#### Example

- **Regex:** `^gamer moment.*`  
  This finds any submission name beginning with “gamer moment”.

## 5. Geo Searches with MGRS

- Select the `MgrsComponent` field.
- Enter an MGRS coordinate (e.g., `17K KS 0123 4567`) in the input box.
- Specify a radius in kilometers. The system converts km to meters and uses `ST_DWithin` to find all points within that distance.

**Be Mindful of Precision:**

- If your MGRS coordinate is coarse (e.g., 10km squares), a 1km radius might miss many points.
- If you want to capture multiple squares, increase your radius accordingly.

#### Example

- **MGRS:** `17K KS 8231 2222`, **Radius:** `2 km`  
  This finds all submissions with MGRS data within 2 kilometers of that coordinate.

## 6. Searching Tabbed Text Editor (TTE) Fields

- Tabbed Text is stored in separate tabs labeled with `<h3>TabName</h3>`.
- The system automatically splits each tab’s content, so you can filter by text inside specific tabs (e.g., “Summary”, “Report Text”).
- **No Duplication:** If the same text is saved repeatedly, the system prevents duplicates from being re-saved, but your search can still match it.

#### Example

- **Condition:** `tabbedTextEditorComponent matches 'Context Statement.*secret'`  
  Finds any TTE tab containing “Context Statement” followed by “secret” (within reason).

## 7. Array Fields and Multi-Value

- Some fields (e.g., “tags”, “chips”) can contain multiple values.
- When searching arrays, you can specify an array operator or use “matches” for partial matching.

#### Example

- Searching tags that contain “urgent” or “hello” might look like:  
  `tags matches '(urgent|hello)'` (Regex approach).

## 8. Common Pitfalls and Tips

### Regex Complexity

- Use anchors (`^`, `$`) if you need exact matches.
- If you’re unfamiliar with POSIX syntax, keep patterns simple.

### MGRS Radius

- Double-check your radius if you expect certain results but see none. Possibly use a bigger radius or a more precise MGRS.

### AND vs. OR

- If your results are too broad, consider switching some ORs to AND.
- If your results are too narrow, try an OR approach.

### Tabbed Text Editor

- If you have multiple TTE fields in one submission, each is uniquely named. Keep track of which TTE you’re querying.

### Duplicate Components

- Some forms may have multiple components of the same type, named with incremental digits. The system handles them by normalizing the name or storing them separately in the database. Just be aware that searching for “tabbedTextEditorComponent” will also match “tabbedTextEditorComponent1” and so on.

## 9. Putting It All Together: Example Search

### Scenario:

You want all submissions where:

- The name starts with “gamer moment”
- The classification is “U”
- They have a TTE “Report Text” containing “urgent”

### Steps:

1. **Add Condition:** `Name matches ^gamer moment` (Regex).
2. **Add Condition:** `ClassificationComponent equals U`.
3. **Add Condition:** `tabbedTextEditorComponent matches Report Text.*urgent`.
4. **Combine conditions with AND.**

You’ll see submissions that match all three conditions. Adjust as needed for OR logic if you want a broader set.
