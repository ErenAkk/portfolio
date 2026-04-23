-- =========================================
-- BRANCH PERFORMANCE & SLA DASHBOARD
-- SQL QUERY PACK
-- =========================================

-- 1. Monthly Branch Credit Volume
SELECT 
    b.BranchName,
    b.Region,
    COUNT(c.CreditID) AS TotalCreditCount,
    SUM(c.Amount) AS TotalCreditVolume_TRY,
    m.TargetVolume_TRY,
    (SUM(c.Amount) / m.TargetVolume_TRY) * 100 AS AchievementPercentage
FROM Credits c
JOIN Branches b ON c.BranchID = b.BranchID
JOIN MonthlyTargets m ON b.BranchID = m.BranchID
                    AND MONTH(c.IssueDate) = m.TargetMonth
WHERE YEAR(c.IssueDate) = 2026
  AND MONTH(c.IssueDate) = 3
GROUP BY b.BranchName, b.Region, m.TargetVolume_TRY
ORDER BY AchievementPercentage DESC;

-- 2. SLA Breach Ratio by Branch
SELECT
    BranchID,
    COUNT(*) AS TotalTransactions,
    SUM(CASE WHEN WaitingTimeMinutes > 5 THEN 1 ELSE 0 END) AS BreachCount,
    CAST(SUM(CASE WHEN WaitingTimeMinutes > 5 THEN 1 ELSE 0 END) * 100.0 / COUNT(*) AS DECIMAL(5,2)) AS BreachPercentage
FROM BranchQueueMetrics
WHERE TransactionDate BETWEEN '2026-03-01' AND '2026-03-31'
GROUP BY BranchID
ORDER BY BreachPercentage DESC;

-- 3. Net Customer Acquisition by Channel
SELECT
    BranchID,
    Channel,
    COUNT(CustomerID) AS NewCustomers
FROM CustomerAcquisition
WHERE AcquisitionDate BETWEEN '2026-03-01' AND '2026-03-31'
GROUP BY BranchID, Channel
ORDER BY BranchID, NewCustomers DESC;

-- 4. NPS & Complaint Ratio by Branch
SELECT
    BranchID,
    AVG(NPSScore) AS AvgNPS,
    SUM(CASE WHEN ComplaintFlag = 1 THEN 1 ELSE 0 END) AS ComplaintCount,
    COUNT(*) AS SurveyCount
FROM CustomerFeedback
WHERE FeedbackDate BETWEEN '2026-03-01' AND '2026-03-31'
GROUP BY BranchID
ORDER BY AvgNPS ASC;

-- 5. Staff Productivity by Branch
SELECT
    BranchID,
    COUNT(TransactionID) AS TotalTransactions,
    COUNT(DISTINCT EmployeeID) AS ActiveEmployees,
    CAST(COUNT(TransactionID) * 1.0 / COUNT(DISTINCT EmployeeID) AS DECIMAL(10,2)) AS ProductivityPerEmployee
FROM BranchTransactions
WHERE TransactionDate BETWEEN '2026-03-01' AND '2026-03-31'
GROUP BY BranchID
ORDER BY ProductivityPerEmployee DESC;