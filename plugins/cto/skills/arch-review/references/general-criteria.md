# 通用架構審查標準

當專案沒有 CLAUDE.md 或指定標準文件時，使用以下通用框架。

---

## 系統設計

- 是否考慮了 scalability（水平 / 垂直擴展）
- 單點失敗（SPOF）是否有對應處理
- 是否有明確的 error handling 與 retry 策略
- API 設計是否符合 RESTful 或既有團隊規範
- 是否考慮了 idempotency（特別是金融、交易類操作）

## 資料庫

- Query 是否有對應的 index
- 是否有 N+1 query 問題
- 大表操作是否考慮分批或 partitioning
- 線上環境的 schema 變更是否使用 CONCURRENTLY
- Transaction 邊界是否正確，有無 race condition 風險

## 效能

- 時間複雜度是否合理
- 是否有不必要的同步阻塞操作
- Cache 策略是否考慮

## 維運

- 是否有足夠的 logging / observability
- 部署與回滾策略是否清楚
- 對現有系統的影響範圍是否評估過

## 安全性

- 輸入驗證是否完整
- 敏感資料是否有適當保護
- 權限控制是否正確

## 決策品質

- 是否評估過替代方案
- 決策理由是否充分
- 技術債是否有明確說明與計畫
