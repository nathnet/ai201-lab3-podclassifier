# Evaluation Spec — Pod Classifier

Complete this spec **before** writing any code for Milestone 3.

Use Plan or Ask mode to think through each blank field. When you're done,
your answers here become the blueprint for `compute_accuracy()` and
`compute_per_class_accuracy()` in `evaluate.py`.

---

## Background: What is evaluation?

After building a classifier, we need to know how well it works. Evaluation answers:
- **Overall:** What fraction of episodes did we classify correctly?
- **Per-class:** Are we better at some labels than others?

Both functions take the same inputs: a list of predicted labels and a list of
ground-truth labels, in the same order.

---

## compute_accuracy(predictions, ground_truth)

### What it does
Returns the fraction of predictions that exactly match the ground truth.

### Inputs

| Parameter | Type | Description |
|---|---|---|
| `predictions` | `list[str]` | Labels predicted by `classify_episode()`, one per episode. |
| `ground_truth` | `list[str]` | The correct labels, in the same order as `predictions`. |

### Output

| Return value | Type | Description |
|---|---|---|
| accuracy | `float` | A value between 0.0 and 1.0. |

---

### Spec fields — fill these in before writing code

**Formula:**

```

Accuracy = number of correct predictions / total number of predictions
where correct predictions are the predictions that match the corresponding ground truth.
```

---

**Step-by-step logic:**

```
1. Check if total number of predictions is above 0
2. Count how many predictions[i] == ground_truth[i]
3. Divide the correct predictions / total number of predictions
4. Return the result.
```

---

**Edge case — what if both lists are empty?**

```
Return 0.0 to prevent crash from division by zero.
0 is returned because having no predictions means there's nothing to report.
```

---

**Worked example:**

```
predictions  = ["interview", "solo", "panel", "interview"]
ground_truth = ["interview", "solo", "solo",  "narrative"]

i = 1: interview == interview
i = 2: solo == solo
i = 3: panel != solo
i = 4: interview != narrative

The correct prediction count is 2, while the total is 4.
Accuracy would be 2/4 or 0.5.
```

---

## compute_per_class_accuracy(predictions, ground_truth)

### What it does
Returns accuracy broken down by each label. For each label in `VALID_LABELS`,
reports how many episodes with that ground-truth label were classified correctly.

### Inputs

| Parameter | Type | Description |
|---|---|---|
| `predictions` | `list[str]` | Labels predicted by `classify_episode()`. |
| `ground_truth` | `list[str]` | Correct labels, in the same order. |

### Output

A `dict` keyed by label. Each value is a dict with three keys:

```python
{
    "interview": {"correct": int, "total": int, "accuracy": float},
    "solo":      {"correct": int, "total": int, "accuracy": float},
    "panel":     {"correct": int, "total": int, "accuracy": float},
    "narrative": {"correct": int, "total": int, "accuracy": float},
}
```

---

### Spec fields — fill these in before writing code

**What does "correct" mean for a given class?**

```
[blank — be precise. When does an episode count as correctly classified
 for the "interview" class, for example?]
An episode count as correctly classified for each label when ground_truth[i] == label AND 
predictions[i] == label.
```

---

**What does "total" mean for a given class?**

```
The total number of predictions per class is defined by the number of ground_truth for that label.
```

---

**Step-by-step logic:**

```
1. Initialize a dict for each label in VALID_LABELS with correct=0, total=0
2. Loop over zip(predictions, ground_truth)
3. For each pair (predicted, truth): increment total for the truth label; 
   if predicted == truth, also increment correct for that label.
4. After the loop, compute accuracy = correct / total for each label.
5. Return the dict
```

---

**Edge case — what if a class has no examples in ground_truth (total == 0)?**

```
Set accuracy for that label to 0.0.
```

---

**Worked example:**

```
predictions  = ["interview", "interview", "solo", "panel", "panel"]
ground_truth = ["interview", "solo",      "solo", "panel", "narrative"]

label       correct  total  accuracy
----------  -------  -----  --------
interview      1       1      1.0
solo           1       2      0.5
panel          1       1      1.0
narrative      0       1      0.0
```

---

**Evaluation Results:**
This is pasted from a ran evaluation.

```
Evaluation Results
Overall accuracy: 100.0% (20/20)

Per-class accuracy: interview ██████████ 100% (5/5) solo ██████████ 100% (5/5) panel ██████████ 100% (5/5) narrative ██████████ 100% (5/5)

No misclassifications — perfect score!
```

---

## Reflection questions (discuss at the checkpoint)

1. Your overall accuracy might be decent even if one class has very low accuracy.
   Why is per-class accuracy a more informative metric than overall accuracy alone?
   - The per-class accuracy mith suggest the training data might be skewed towards a specific class.
     The model does not receive enough training data to distinguish other classes. It could also
     indicate an unclear boundary for each class. A high overall accuracy might be hiding which
     classes are failing since the test data could be from a single class.

2. If `panel` episodes consistently get misclassified as `interview`, what does
   that tell you about your training labels or your prompt?
   - The label taxonomy may be unclear as to where `interview` ends and `panel` begins. 
     The training data may also be lacking diversity towards `panel`. The model might classify `panel` as `interview` 
     more because it sees multiple people talking, but not distinction between everyone talking about equally 
     and one person leading the discussion.

3. You labeled 20 training episodes and evaluated on 20 test episodes (5 per class).
   How might the evaluation results change if you had labeled 100 training episodes?
   What if you had 200 test episodes?
   - 100 training examples might give the LLM more specific clues and examples about each class. 
     200 test episodes will just give more data to evaluate the accuracy, 
     making the results more statistically reliable.
