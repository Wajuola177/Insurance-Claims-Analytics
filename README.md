-- =============================================
-- PROJECT 1: INSURANCE CLAIMS ANALYTICS 
-- =============================================

-- 1. Total claims and total incurred cost overview
SELECT 
    COUNT(claim_id) AS total_claims,
    SUM(total_incurred) AS total_incurred_cost_gbp,
    AVG(total_incurred) AS average_claim_value
FROM insurance_claims;

-- 2. Count & cost by claim status (open/closed/pending)
SELECT 
    claim_status,
    COUNT(claim_id) AS claim_count,
    SUM(total_incurred) AS total_cost,
    ROUND(AVG(total_incurred), 2) AS avg_claim_value
FROM insurance_claims
GROUP BY claim_status
ORDER BY total_cost DESC;

-- 3. Monthly claim volume and cost trend
SELECT 
    DATE_FORMAT(incurred_date, '%Y-%m') AS claim_month,
    COUNT(claim_id) AS monthly_claims,
    SUM(total_incurred) AS monthly_total_cost,
    ROUND(AVG(total_incurred), 2) AS monthly_avg_cost
FROM insurance_claims
GROUP BY claim_month
ORDER BY claim_month;

-- 4. Top 10 highest-cost individual claims
SELECT 
    claim_id, policy_number, claim_type, total_incurred, 
    incident_date, settlement_date
FROM insurance_claims
ORDER BY total_incurred DESC
LIMIT 10;

-- 5. Claim volume and cost by type (Accident/Health/Property etc.)
SELECT 
    claim_type,
    COUNT(claim_id) AS total_claims,
    SUM(total_incurred) AS total_cost,
    ROUND(AVG(total_incurred), 2) AS avg_cost_per_claim
FROM insurance_claims
GROUP BY claim_type
ORDER BY total_cost DESC;

-- 6. Claims exceeding £10,000 — high-value segment
SELECT 
    claim_id, policy_number, claim_type, total_incurred, region
FROM insurance_claims
WHERE total_incurred > 10000
ORDER BY total_incurred DESC;

-- 7. Regional claim distribution & cost concentration
SELECT 
    region,
    COUNT(claim_id) AS region_claim_count,
    SUM(total_incurred) AS region_total_cost,
    ROUND(AVG(total_incurred), 2) AS region_avg_claim_value
FROM insurance_claims
GROUP BY region
ORDER BY region_total_cost DESC;

-- 8. Average settlement time by claim type (days)
SELECT 
    claim_type,
    COUNT(claim_id) AS closed_claims,
    ROUND(AVG(DATEDIFF(settlement_date, incident_date)), 1) AS avg_settlement_days
FROM insurance_claims
WHERE claim_status = 'Closed'
GROUP BY claim_type;

-- 9. Policies with 2+ repeat claims — identify frequent claimants
SELECT 
    policy_number,
    COUNT(claim_id) AS claim_frequency,
    SUM(total_incurred) AS total_payout
FROM insurance_claims
GROUP BY policy_number
HAVING COUNT(claim_id) >= 2
ORDER BY claim_frequency DESC;

-- 10. Year-on-year claim growth comparison
SELECT 
    YEAR(incurred_date) AS claim_year,
    COUNT(claim_id) AS annual_claims,
    SUM(total_incurred) AS annual_cost,
    LAG(SUM(total_incurred)) OVER (ORDER BY YEAR(incurred_date)) AS prior_year_cost,
    ROUND(
        (SUM(total_incurred) - LAG(SUM(total_incurred)) OVER (ORDER BY YEAR(incurred_date))) 
        / LAG(SUM(total_incurred)) OVER (ORDER BY YEAR(incurred_date)) * 100, 
    2) AS yoy_cost_percentage_change
FROM insurance_claims
GROUP BY claim_year;

-- 11. Cost breakdown by severity tier
SELECT 
    CASE 
        WHEN total_incurred < 1000 THEN 'Low (<£1k)'
        WHEN total_incurred BETWEEN 1000 AND 5000 THEN 'Medium (£1k–£5k)'
        WHEN total_incurred BETWEEN 5001 AND 10000 THEN 'High (£5k–£10k)'
        ELSE 'Very High (>£10k)'
    END AS cost_severity,
    COUNT(claim_id) AS claim_count,
    SUM(total_incurred) AS segment_total_cost
FROM insurance_claims
GROUP BY cost_severity
ORDER BY segment_total_cost DESC;

-- 12. Claims with missing critical fields — data quality check
SELECT claim_id, policy_number, claim_type
FROM insurance_claims
WHERE 
    total_incurred IS NULL 
    OR incident_date IS NULL 
    OR policy_number IS NULL;

-- 13. Quarter-over-quarter performance summary
SELECT 
    CONCAT(YEAR(incurred_date), '-Q', QUARTER(incurred_date)) AS claim_quarter,
    COUNT(claim_id) AS quarterly_claims,
    SUM(total_incurred) AS quarterly_total,
    ROUND(AVG(total_incurred), 2) AS quarterly_average
FROM insurance_claims
GROUP BY claim_quarter
ORDER BY claim_quarter;

-- 14. Correlation: policy tenure vs average claim value
SELECT 
    CASE 
        WHEN policy_tenure_years < 1 THEN 'New (<1yr)'
        WHEN policy_tenure_years BETWEEN 1 AND 3 THEN 'Established (1–3yr)'
        WHEN policy_tenure_years BETWEEN 4 AND 7 THEN 'Long-term (4–7yr)'
        ELSE 'Veteran (8+yr)'
    END AS policy_tenure_band,
    COUNT(claim_id) AS claim_count,
    ROUND(AVG(total_incurred), 2) AS avg_claim_value
FROM insurance_claims
GROUP BY policy_tenure_band;

-- 15. Top 5 regions by average claim cost
SELECT 
    region,
    COUNT(claim_id) AS total_claims,
    ROUND(AVG(total_incurred), 2) AS avg_claim_value
FROM insurance_claims
GROUP BY region
ORDER BY avg_claim_value DESC
LIMIT 5;
