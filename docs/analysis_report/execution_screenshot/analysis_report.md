# Customer Feedback Sentiment & Opinion Mining — Live Azure Run

This document summarizes results from a live analysis run against the **Azure Cognitive Services Language Service**, covering sentiment classification and aspect-level opinion extraction across a set of customer-submitted text samples.

---

## 🎯 Summary at a Glance

A total of **7 text samples** were submitted for analysis, covering product reviews, delivery feedback, support interactions, and dining experiences. Every request was authenticated, batch-submitted, and processed end-to-end by Azure's pre-trained NLP engine without errors.

**Overall sentiment split:**

| Sentiment | Count | Share |
|---|---|---|
| Negative | 3 | 42.9% |
| Mixed | 2 | 28.6% |
| Positive | 2 | 28.6% |
| Neutral | 0 | 0.0% |

**Mean confidence values across all documents:**

| Polarity | Avg. Confidence |
|---|---|
| Positive | 0.4700 |
| Negative | 0.4957 |
| Neutral | 0.0343 |

---

## 📋 Per-Document Results

Each row reflects the label and confidence scores Azure assigned to a given input.

| Doc ID | Summary of Input | Label Assigned | Pos. Score | Neu. Score | Neg. Score |
|---|---|---|---|---|---|
| 1 | Praise for a laptop's battery and screen | Positive | 1.0000 | 0.0000 | 0.0000 |
| 2 | Complaint about a multi-hour support wait | Negative | 0.0000 | 0.1200 | 0.8800 |
| 3 | On-time delivery, product as described | Mixed | 0.7600 | 0.1200 | 0.1200 |
| 4 | Good camera/performance, slow charging, high price | Negative | 0.0300 | 0.0000 | 0.9700 |
| 5 | High praise for espresso quality | Positive | 1.0000 | 0.0000 | 0.0000 |
| om1 | Tasty food, but slow and noisy service | Negative | 0.0000 | 0.0000 | 1.0000 |
| om2 | Strong battery life, weak low-light camera | Mixed | 0.5000 | 0.0000 | 0.5000 |

---

## 🧩 Aspect-Level Findings

Rather than relying solely on an overall document score, opinion mining isolates individual product or service features and tags each one with its own sentiment and supporting keyword.

### Document 1 — Laptop Review
- **laptop** → Positive (cue word: "love")
- **battery life** → Positive (cue word: "outstanding")
- **display** → Positive (cue word: "crystal clear")
- **purchase** → Positive (cue word: "Best")

### Document 2 — Support Experience
- **customer service** → Negative (cue word: "terrible")
- **product quality** → Negative (cue word: "below average")

### Document 3 — Delivery & Product
- **product** → Positive (cue word: "works")

### Document 4 — Electronics Review
- **camera** → Positive (cue word: "Great")
- **performance** → Positive (cue word: "smooth")
- **charging speed** → Negative (cue word: "slow")
- **price** → Negative (cue word: "too high")

### Document 5 — Coffee Maker
- **espresso** → Positive (cue word: "amazing")
- **build quality** → Positive (cue word: "premium")

### Document om1 — Restaurant Visit
- **food** → Positive (cue word: "delicious")
- **portions** → Positive (cue word: "generous")
- **service** → Negative (cue word: "slow")
- **restaurant** → Negative (cue word: "noisy")

### Document om2 — Smartphone Review
- **camera quality** → Negative (cue word: "disappointing")

---

## 🖨️ Raw Console Output

The text below reflects the exact summary written to `output/summary_report.txt` when the script was executed:

```text
======================================================================
          SENTIMENT ANALYSIS RESULTS — AZURE COGNITIVE SERVICES          
======================================================================
Run Type             : LIVE (Azure Cognitive Services)
Documents Processed  : 7
Time Taken           : 1.526 seconds
----------------------------------------------------------------------
SENTIMENT BREAKDOWN:
  * POSITIVE :   2 ( 28.6%)
  * MIXED    :   2 ( 28.6%)
  * NEUTRAL  :   0 (  0.0%)
  * NEGATIVE :   3 ( 42.9%)
----------------------------------------------------------------------
MEAN CONFIDENCE LEVELS:
  * Positive: 0.4700
  * Neutral : 0.0343
  * Negative: 0.4957
----------------------------------------------------------------------
ASPECT-LEVEL SENTIMENT (SELECTED EXAMPLES):
  - 'laptop' → POSITIVE (keyword: love)
  - 'battery life' → POSITIVE (keyword: outstanding)
  - 'display' → POSITIVE (keyword: crystal clear)
  - 'purchase' → POSITIVE (keyword: Best)
  - 'customer service' → NEGATIVE (keyword: terrible)
  - 'product quality' → NEGATIVE (keyword: below average)
  - 'product' → POSITIVE (keyword: works)
  - 'camera' → POSITIVE (keyword: Great)
  - 'performance' → POSITIVE (keyword: smooth)
  - 'charging speed' → NEGATIVE (keyword: slow)
  - 'price' → NEGATIVE (keyword: too high)
  - 'espresso' → POSITIVE (keyword: amazing)
  ... plus 6 additional aspects detected.
======================================================================
```

---

## 🖼️ Screenshot Reference

A capture confirming the live request and the resulting aspect-level extraction is available here:

![Live Azure Execution Screenshot](file:///c:/Users/duduy/OneDrive/Documents/Assignment_12/docs/execution_screenshot.png)
