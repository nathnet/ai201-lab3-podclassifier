# Classifier Spec — Pod Classifier

Complete this spec **before** writing any code for Milestone 2.

Use Plan or Ask mode to think through each blank field. When you're done,
your answers here become the blueprint for `build_few_shot_prompt()` and
`classify_episode()` in `classifier.py`.

---

## build_few_shot_prompt(labeled_examples, description)

### What it does
Constructs a prompt string for the LLM that includes the task instructions,
all labeled training examples, and the new episode description to classify.

### Inputs

| Parameter | Type | Description |
|---|---|---|
| `labeled_examples` | `list[dict]` | Each dict has `"title"`, `"description"`, `"label"` (and others). These are the examples you labeled in Milestone 1. |
| `description` | `str` | The episode description to classify. |

### Output

| Return value | Type | Description |
|---|---|---|
| prompt | `str` | A complete prompt string ready to send to the LLM. |

---

### Spec fields — fill these in before writing code

**Task instruction (what should the LLM know about the task?):**

```
You are classifying podcast episodes by their format. Classify the episode
into exactly one of these four labels:

- interview: a conversation between a host and one or more guests
- solo: a single host speaking from memory, experience, or opinion — no guests,
  no assembled external sources
- panel: multiple guests with roughly equal speaking time, often debating or
  discussing a topic together
- narrative: a story assembled from external sources — interviews, archival
  audio, reporting — with a clear narrative arc

Return only the label and your reasoning. Do not explain the taxonomy.
```

---

**How should labeled examples be formatted in the prompt?**

```
Each example should include the episode title, a brief excerpt or the full
description, and the correct label. Separate examples with a blank line or
a delimiter like "---". Include all fields that help the model see why the
label was applied — title and description are both useful; other fields
(like episode ID) are not needed.
```

---

**Example block sketch (write one concrete example):**

```
Title: {title}
Description: {description}
Label: {label}
```

---

**How should the new episode (to be classified) be presented?**

```
Present it in the same format as the labeled examples, but omit the Label
line and replace it with an instruction to classify. For example:

Title: {title}
Description: {description}
Label: ?

Then add a line like: "Classify the episode above. Return your answer in
the format below:" followed by the output format you chose.
```

---

**What output format should you request from the LLM?**

```
Label: X
Reasoning: Y
```

---

**Edge cases to handle in the prompt:**

```
Edge cases will be handled upstream in classify_episode() before this function is called.
If the labeled_examples is empty or description is empty, classify_episode() returns early
with "unknown" and never calls build_few_shot_prompt(). For short description, the LLM
will just have to work with less signal. No changes will be made to the prompt.
```

---

## classify_episode(description, labeled_examples)

### What it does
Classifies a single podcast episode description using the few-shot LLM classifier.
Returns a dict with a label and reasoning.

### Inputs

| Parameter | Type | Description |
|---|---|---|
| `description` | `str` | The episode description to classify. |
| `labeled_examples` | `list[dict]` | Labeled training examples from `load_labeled_examples()`. |

### Output

| Return value | Type | Description |
|---|---|---|
| result | `dict` | Must have keys `"label"` and `"reasoning"`. `"label"` must be one of `VALID_LABELS` or `"unknown"`. |

---

### Spec fields — fill these in before writing code

**Step 1 — Build the prompt:**

```
Call build_few_shot_prompt(labeled_examples, description) and store the
returned string in a variable (e.g., prompt). Pass through both arguments
exactly as received — no modification needed before calling.
```

---

**Step 2 — Send to the LLM:**

```
Call _client.chat.completions.create() with:
  - model: the model name from config (LLM_MODEL)
  - messages: a list with one dict — {"role": "user", "content": prompt}
    (system-design.md shows an optional system message too — either shape works)
  - max_tokens: a reasonable limit (e.g., 200–300) to keep responses concise

Extract the response text from:
  response.choices[0].message.content
```

---

**Step 3 — Parse the response:**

```
response_text = response.choices[0].message.conent
label = response_text.split("Label:")[-1].split("\n")[0].strip().lower()
reasoning = response_text.split("reasoning:")[-1].strip()
```

---

**Step 4 — Validate the label:**

```
if label not in VALID_LABELS:
  label = "unknown"
```

---

**Step 5 — Handle errors gracefully:**

```
except Exception as e:
  return {"label": "unknown", "reasoning": f"Error: {e}"}
```

---

### Return value structure

```python
{
    "label": str,      # one of VALID_LABELS, or "unknown" if invalid/error
    "reasoning": str,  # brief explanation from the LLM
}
```

---

## Notes on label quality

The classifier is only as good as your labels. If your training examples have
inconsistent or ambiguous labels, the LLM will learn the wrong pattern.

Before implementing the classifier, re-read `data/taxonomy.md` and double-check
any labels you're unsure about. Annotation quality is part of the lab.

---

## Implementation Notes

*Fill this in after implementing and testing both functions.*

**Test: what does the raw LLM response look like for one episode?**

```
Episode tested: solo
Raw response text: The episode features a single host speaking from their opinion and experience, with no guests or external sources, discussing their thoughts on the four-day workweek.
```

**How did you parse the label out of the response?**

```
[describe the string operations — strip, split, lower, etc.]
The response string is split using the word "Label:", and what comes last in the split array should be the response.
Then I took the last element in the split array and applied another split on the new line and choose the first element 
since the new line split would leave "<label>" and "Reasoning: <brief reasoning>".
Then I applied .lower() and .strip() to ensure the label is in lowercase and has no leading/trailing spaces.
```

**Did any episodes return `"unknown"`? If so, why?**

```
no, all the episodes returned their corresponding label:  

The Case for Four-Day Workweeks: solo  
Dr. Priya Nair on Adolescent Mental Health After the Pandemic: interview  
The Aral Sea: A Disaster in Four Acts: narrative  
Five Writers on What It Means to Write for the Internet Now: panel  
```

**One thing about the output format that surprised you:**

```
N/A, the LLM was very precise in returning correct output format
```
