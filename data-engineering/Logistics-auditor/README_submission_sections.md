# Veridi Logistics — Delivery Performance Audit

*Submission by [Your Name] — [Month Year]*

---

## A. Executive Summary

Late deliveries are a **regional problem, not a nationwide one — and they are overwhelmingly the carrier's fault, not the sellers'.** Across ~96.5k delivered orders, 8.1% missed the promised delivery date, but the failures concentrate in the Northeast (Alagoas 23.9%, Maranhão 19.7% vs. São Paulo ~6%), while Rio de Janeiro contributes the largest single pile of late orders through sheer volume. The cost is measurable: on-time orders average **4.29 stars**, while orders more than 5 days late collapse to **1.78** — and worryingly, these "Super Late" failures (4.2% of orders) outnumber mildly late ones, meaning when we miss, we miss badly. Decomposing each late delivery into its seller leg and carrier leg shows **72.8% of late orders were dispatched on time by the seller and lost in transit**, with the carrier leg ballooning from 7.4 days (on-time orders) to 32.6 days (super-late). Recommended focus: renegotiate carrier lanes serving the Northeast, recalibrate delivery estimates in that region, and add seller dispatch enforcement specifically in Rio de Janeiro.

## B. Project Links

- **Notebook (Google Colab):** [PASTE COLAB SHARE LINK — set to "Anyone with the link: Viewer"]
- **Dashboard (Tableau Public):** [PASTE TABLEAU PUBLIC LINK]
- **Presentation (slides):** [PASTE SLIDES/PDF LINK — "Anyone with the link can view"]
- **Video walkthrough (optional):** [PASTE YOUTUBE LINK or remove this line]

*The `.ipynb` notebook and an HTML/PDF export with all rendered charts are included in this repository.*

## C. Technical Explanation

### Data cleaning

- **Type handling:** all date columns parsed to datetimes at load time, since the core analysis is date arithmetic.
- **Duplicate reviews:** 551 orders carried more than one review; a naive join would have duplicated those orders and skewed every downstream percentage. I deduplicated to the **most recent review per order** (the customer's final verdict) and guarded the master table with asserts enforcing exactly one row per order after every merge.
- **Missing values:** 768 orders have no review — kept via left joins so their delivery data still counts, excluded only from sentiment analysis. 2,965 orders were never delivered (canceled, unavailable, in transit); a delay is undefined for them, so they are flagged **"Not Delivered"** and excluded from delay statistics rather than silently dropped. Eight rows marked "delivered" but missing a delivery date (data entry gaps) are routed to the same flag by the classifier.
- **Outliers kept deliberately:** some deliveries ran 100+ days late and some estimates were beaten by 100+ days. These are real events, not data errors — removing them would delete exactly the failures this audit exists to find. They are retained and discussed in the notebook.
- **Category translation (bonus story):** Portuguese categories translated via the dataset's official mapping file. Since orders can contain multiple items, I attach one category per order (the first item's) to preserve the one-row-per-order guarantee; ~2.2k orders lack a category (mostly canceled orders with no item records) and are excluded from category-level views only.

### Candidate's Choice: the Seller-vs-Carrier blame split

Knowing *where* deliveries are late doesn't tell Veridi *what to fix*. Every delivery has two legs — seller (purchase → carrier handoff) and carrier (handoff → customer's door) — and the items table provides `shipping_limit_date`, the contractual deadline for the seller's handoff. For each late order I assign responsibility: **Seller** if the handoff missed that deadline, **Carrier** if the seller dispatched on time yet the order still arrived late.

**Why it matters to the business:** the two verdicts trigger different, expensive actions — seller SLAs and penalties on one side, renegotiating carrier contracts on specific routes on the other. Without the split, Veridi risks spending money fixing the wrong half of the chain. The result — 72.8% carrier-caused, with the Northeast almost purely a carrier/routing problem while Rio shows a meaningful seller share on top — turns the audit from a diagnosis into a differentiated action plan per region.

---

*The original project brief follows below.*
