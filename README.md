# Sales Drop Investigation

Business thinking practice: paano linawin ang isang vague na tanong ng VP, gumawa ng hypothesis, at i-verify gamit ang SQL bago magbigay ng recommendation.

*Phase 3 — Business Thinking, Lesson 1*

---

## Business Context

Ikaw ang Data Analyst ng isang retail company na may maraming branches. Bigla na lang nag-message ang VP of Sales: *"Hey, sales dropped last month. Can you look into why and get back to me today?"*

## Business Questions

Bago pumunta sa SQL, kinailangan munang linawin ang malabong request, dahil ang salitang "sales" ay pwedeng maraming ibig sabihin (revenue, units, orders), at "last month" ay pwedeng ikumpara sa iba't ibang paraan.

Ginamit ang Investigation Framework: **Metric → Timeframe → Scope → Magnitude → Hypothesize**

Clarifying message na ipinadala sa VP:

> "Hi ma'am, quick lang po, sa 'sales' po, ang assumption ko po ay total revenue, kumpara po sa nakaraang buwan (October vs November (MoM)). Tama po ba ito, o baka po kumpara sa parehong buwan noong nakaraang taon (YoY) ang tinutukoy niyo? At nangyari po ba ito sa lahat ng branches/products, o may specific branch/product lang na apektado? Magsisimula na po akong mag-check ng numbers habang hinihintay ko ang confirmation niyo."

Sagot ng VP:
- **Metric** — Revenue
- **Timeframe** — MoM (October vs November)
- **Scope** — Lahat ng branches, hindi lang isa

Ang Magnitude (gaano kalaki ang drop) hindi na kinailangang itanong sa VP, dahil kailangan itong kunin mismo sa data, hindi ito nasa utak/intensyon ng VP, factual number lang na kakalkulahin.

## Hypothesis

Dahil company-wide ang apektado (lahat ng branches), na-eliminate agad ang mga hypothesis na dapat sana isolated lang mangyari, gaya ng stockout o system error sa isang branch, kasi kung ganon ang dahilan, hindi dapat lahat ng branches sabay-sabay ang apektado.

Ang natirang malakas na hypothesis: **tumaas ang presyo → lumipat ang customers sa mas murang competitor → bumaba ang orders → bumaba ang revenue**

| Cause | Category |
|---|---|
| Nagpataas ng presyo ang company | Internal |
| Lumipat ang customers sa competitor | External |

Para ma-verify ito, kailangang kunin sa data:
1. Average price (Oct vs Nov)
2. Bilang ng orders (Oct vs Nov)
3. Total revenue (Oct vs Nov)

## Dataset

Table: `sales_transactions` (order_id, branch, order_date, customer_id, unit_price, quantity, revenue)

**Sample Data:**

| order_id | branch | order_date | customer_id | unit_price | quantity | revenue |
|---|---|---|---|---|---|---|
| 1001 | Branch A | 2025-10-05 | C001 | 500 | 2 | 1000 |
| 1002 | Branch B | 2025-10-12 | C002 | 500 | 1 | 500 |
| 1003 | Branch A | 2025-10-20 | C003 | 500 | 3 | 1500 |
| 1004 | Branch A | 2025-11-03 | C001 | 600 | 1 | 600 |
| 1005 | Branch B | 2025-11-10 | C004 | 600 | 1 | 600 |
| 1006 | Branch C | 2025-11-15 | C005 | 600 | 2 | 1200 |
| ... | ... | ... | ... | ... | ... | ... |

## SQL Query

```sql
SELECT
    DATE_FORMAT(order_date, '%Y-%m') AS order_month,
    SUM(revenue) AS total_revenue,
    AVG(unit_price) AS average_price,
    COUNT(order_id) AS order_count
FROM sales_transactions
GROUP BY 
    DATE_FORMAT(order_date, '%Y-%m');
```

Mga desisyon sa likod ng query:
- Ginamit ang `SUM(revenue)` para makuha ang total revenue per month, isang value per group (buwan), hindi per row.
- Ginamit ang `AVG(unit_price)` para makita kung tumaas ba talaga ang presyo per unit.
- Ginamit ang `COUNT(order_id)` para bilangin ang orders, order_id ang ginamit dahil unique identifier ito ng bawat transaction, hindi quantity na bilang ng piraso lang.

## Result

| order_month | total_revenue | average_price | order_count |
|---|---|---|---|
| 2025-10 | ₱5,000,000 | ₱500 | 10,000 |
| 2025-11 | ₱4,500,000 | ₱600 | 7,500 |

## Insight

Bumaba ang sales dahil sa pagtaas ng presyo, na nagreresulta ng paglipat ng customers sa ibang store, makikita ito sa 25% na pagbaba ng order count mula 10,000 papuntang 7,500, kasabay ng pagtaas ng average price mula ₱500 papuntang ₱600.

## Recommendation

Magbigay ng targeted discount promo (hal. 10% off sa next purchase) partikular sa mga customer na bumili noong October pero hindi na bumili noong November, sa halip na across-the-board discount sa lahat.

**Reason:** Sa ganitong paraan, mapapanatili ang normal na presyo (₱600) para sa mga regular customer na okay lang bumili sa bagong presyo, habang hinihikayat balik ang mga customer na malapit nang mawala.

**Expected Result:** Kung ma-re-engage kahit ilan sa mga churned customer, mababawasan ang epekto ng price increase sa overall revenue nang hindi na kailangang ibaba pabalik ang presyo para sa lahat.

Supporting SQL — paghahanap ng churned customers:

```sql
SELECT DISTINCT customer_id
FROM sales_transactions
WHERE order_month = '2025-10'
AND customer_id NOT IN (
    SELECT customer_id 
    FROM sales_transactions 
    WHERE order_month = '2025-11'
);
```

---

## Skills na Ginamit

- SQL: `SUM()`, `AVG()`, `COUNT()`, `GROUP BY`, `DATE_FORMAT()`, `NOT IN` subquery
- Business Thinking: Investigation Framework (Metric → Timeframe → Scope → Magnitude → Hypothesize) bago pumunta sa code
- Hypothesis categorization: Internal vs External na dahilan
- Business Communication: "Answer First" na format — Insight → Supporting Data → Recommendation
