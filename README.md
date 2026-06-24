# EVM Ecosystem: 12-Month DEX Performance Overview

## 📌 Project Objective

This project provides a comparative data analysis of decentralized exchange (DEX) activity across the top five EVM networks over a 12-month period. By processing over $1 Trillion in cumulative volume and analyzing 5.5 million unique user addresses, the objective is to uncover structural differences in liquidity trends and retail engagement between Ethereum L1 and the emerging L2 scaling solutions.

## 🛠️ Tools Used

* **SQL (Dune Analytics):** Data extraction, aggregation, and window functions to isolate top-performing protocols.
* **Tableau:** Data visualization and dashboard design.

## 📊 The Dashboard

[**View the Interactive Tableau Dashboard Here**](https://public.tableau.com/views/TheEVMSuperchainReportVolumeTradesandUsers/TheEVMSuperchainReportVolumeTradesandUsers?:language=en-US&publish=yes&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)

## 💡 Key Insights

1. **The Layer 1 Capital Stronghold:** Ethereum remains the undisputed king of liquidity, processing over $654B in volume, primarily driven by large institutional trades.
2. **The Layer 2 Retail Engine:** Base (Aerodrome) processed more than double the number of individual trades (225M) compared to Ethereum L1 (107M), proving that sub-cent gas fees drive high-frequency micro-trading.
3. **The OP Stack MetaDEX Trend:** While Uniswap dominates Ethereum, Arbitrum, and Polygon, the native OP Stack networks (Optimism and Base) are dominated by their native MetaDEX architecture (Velodrome and Aerodrome).

## 💻 SQL Methodology

To optimize database scanning and prevent timeout errors across millions of rows, I utilized a Common Table Expression (CTE) and the `ROW_NUMBER()` window function to dynamically rank and pull only the #1 DEX per network in a single query pass.

```sql
WITH ranked_dex_volume AS (
    SELECT 
        blockchain,
        project,
        COUNT(*) AS total_trades,
        COUNT(DISTINCT taker) AS unique_users,
        SUM(CAST(amount_usd AS DOUBLE)) AS total_volume_usd,
        ROW_NUMBER() OVER (
            PARTITION BY blockchain 
            ORDER BY SUM(CAST(amount_usd AS DOUBLE)) DESC
        ) AS volume_rank
    FROM dex.trades
    WHERE 
        blockchain IN ('ethereum', 'base', 'arbitrum', 'optimism', 'polygon')
        AND block_time > NOW() - INTERVAL '365' day
    GROUP BY 
        blockchain,
        project
)
SELECT 
    blockchain,
    project AS top_dex,
    total_trades,
    unique_users,
    total_volume_usd
FROM ranked_dex_volume
WHERE volume_rank = 1
ORDER BY total_volume_usd DESC;
