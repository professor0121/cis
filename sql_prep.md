1. Basics
    dSQL aur NoSQL me kya difference hai?
    dDDL, DML, DCL, TCL, DQL kya hote hain? Examples do.
    dDELETE, TRUNCATE, aur DROP me difference batao.
    sWHERE aur HAVING me kya fark hai?
    dCHAR vs VARCHAR vs TEXT?
    dUNION vs UNION ALL?
    LIKE, IN, BETWEEN, EXISTS kab use karte hain?
    COUNT(*), COUNT(col), COUNT(DISTINCT col) me difference?
    ORDER BY ke saath LIMIT/OFFSET kaise kaam karta hai?
    Alias kya hota hai aur kab zaroori hai?
2. NULL Handling
    NULL = NULL false kyun hota hai?
    COALESCE, IFNULL, NULLIF ka use batao.
    NOT IN me NULL aaye toh kya problem hoti hai?
    Aggregate functions NULL ko kaise treat karte hain?
    Aise users nikalo jinka email NULL ya empty string dono ho sakta hai.
3. Joins
    INNER, LEFT, RIGHT, FULL OUTER, CROSS, SELF join explain karo.
    Aise users nikalo jinhone kabhi order nahi kiya.
    Aise products nikalo jo kabhi sell nahi hue.
    Har employee ke saath uske manager ka naam dikhao.
    Un departments ki list do jisme koi employee nahi hai.
    JOIN vs WHERE me condition rakhne ka LEFT JOIN me kya effect padta hai?
    Natural join kyun avoid karna chahiye?
    3 se zyada tables join karte waqt performance kaise sochoge?
4. Aggregation & Grouping
        Har city ke total users aur total revenue nikalo.
    Un categories ko dikhao jinka average price 500 se zyada hai.
    Month-wise total sales nikalo.
    Top 5 customers by total spending.
    Un users ko nikalo jinhone 3 se zyada orders diye.
    GROUP BY me non-aggregated column select karne se kya hota hai?
    ROLLUP, CUBE, GROUPING SETS kya hain?
    Conditional aggregation: ek query me delivered, cancelled, pending orders ka count alag columns me.
5. Subqueries
Correlated vs non-correlated subquery?
Second highest salary kitne tareekon se nikal sakte ho?
Nth highest salary nikalo.
Un employees ko nikalo jinki salary apne department ki average se zyada hai.
IN vs EXISTS me performance difference?
Subquery vs JOIN kab kya choose karoge?
Un products ko nikalo jinka price overall average se kam hai.
6. Window Functions
ROW_NUMBER, RANK, DENSE_RANK me difference?
Har department me top 3 salary wale employees.
Running total of daily sales.
Month-over-month growth % nikalo (LAG).
Har user ka pehla aur aakhri order dikhao.
Moving average (last 7 days).
PARTITION BY vs GROUP BY?
NTILE se customers ko 4 buckets me divide karo.
FIRST_VALUE, LAST_VALUE ka frame clause issue kya hai?
7. CTE & Recursive Queries
CTE vs subquery vs temp table?
Recursive CTE se employee hierarchy (manager → reportees) nikalo.
1 se 100 tak numbers generate karo bina table ke.
Category tree (parent-child) flatten karo.
Multiple CTEs chain karke monthly cohort report banao.
8. Date & String Functions
Pichle 30 din ke orders nikalo.
Har month ka pehla aur aakhri din nikalo.
Age calculate karo DOB se.
Un users ko nikalo jinka signup weekend pe hua.
Email se domain extract karo.
Name ka first letter capital karo.
Comma-separated string ko rows me todo.
Duplicate spaces hatao string se.
9. Duplicates & Data Cleaning
Duplicate emails dhundo.
Duplicate rows delete karo, par ek rakho.
Table me kisi bhi column ka duplicate count nikalo.
Consecutive duplicate values detect karo.
Missing IDs (gaps in sequence) dhundo.
10. DML & DDL
INSERT ... SELECT kab use karte ho?
UPDATE with JOIN kaise karte hain?
UPSERT (ON DUPLICATE KEY / ON CONFLICT) explain karo.
ALTER TABLE se column add, rename, drop, type change.
MERGE statement kya hai?
Bulk insert ke best practices?
Large table me ADD COLUMN production me safely kaise karoge?
11. Keys, Constraints, Normalization
Primary, Foreign, Unique, Composite, Candidate, Super key?
ON DELETE CASCADE, SET NULL, RESTRICT?
CHECK, DEFAULT, NOT NULL constraints?
1NF, 2NF, 3NF, BCNF with example.
Denormalization kab sahi hai?
Surrogate key vs natural key?
UUID vs auto-increment ID: pros/cons?
12. Indexes & Performance
Index kya hai aur kaise kaam karta hai (B-Tree)?
Clustered vs non-clustered index?
Composite index me column order kyun matter karta hai?
Covering index kya hota hai?
Index hone ke baad bhi query slow kyun ho sakti hai?
EXPLAIN / EXPLAIN ANALYZE kaise padhte ho?
LIKE '%abc' pe index kyun nahi lagta?
Function lagane se (WHERE YEAR(date)=2025) index kyun skip hota hai?
N+1 query problem kya hai?
Pagination: OFFSET vs keyset pagination?
Slow query debug karne ka step-by-step approach batao.
Too many indexes ka downside?
13. Transactions & Concurrency
ACID properties example ke saath.
Isolation levels aur unke problems (dirty read, non-repeatable read, phantom read).
Deadlock kya hai aur kaise avoid karoge?
Optimistic vs pessimistic locking?
SELECT ... FOR UPDATE kab use karte ho?
Savepoint, commit, rollback?
Do users ek saath last stock item kharidne ki koshish karein, kya hoga?
14. Views, Procedures, Triggers, Functions
View vs Materialized View?
Stored procedure vs function?
Trigger ka use case aur downside?
Updatable views ke rules?
Audit log trigger se kaise banaoge?
15. Scenario-Based (E-commerce)
Last 7 din me jin users ne cart me add kiya par order nahi kiya, unki list.
Har product category ka top-selling product.
Repeat customers (2+ orders) ka percentage.
Har user ka average order value aur lifetime value.
Un orders ko nikalo jinka payment fail hua par order status “confirmed” hai.
Month-wise new vs returning customers.
Revenue by city, jisme sirf delivered orders count hon.
Aise users jinhone pehle order ke 30 din ke andar dusra order kiya.
Product ke saath aksar kaunse products saath kharide jaate hain (market basket)?
Stock negative ho gaya, kaise detect aur prevent karoge?
Refund hone par revenue report me kaise adjust karoge?
Coupon abuse detect karo: ek user ne same coupon 3+ baar use kiya.
16. Scenario-Based (HR / Company)
Un managers ko nikalo jinke under 5 se zyada reportees hain.
Har department me highest paid employee (ties handle karte hue).
Employees jo apne manager se zyada kama rahe hain.
Pichle 6 mahine me hire hue employees ka dept-wise count.
Salary ka median nikalo.
Company hierarchy me kisi employee ke upar ke saare managers (CEO tak).
17. Scenario-Based (Analytics / Product)
DAU, WAU, MAU nikalo login table se.
Retention: day-1, day-7, day-30 cohort analysis.
Funnel: signup → add to cart → checkout → payment, har step ka drop-off.
Session nikalo: 30 min inactivity ke baad naya session.
Consecutive 3 din login karne wale users.
Longest login streak per user.
Peak hour of orders.
Un users ko dhundo jinki activity pichle 30 din me zero hai (churned).
18. Debugging & Optimization Scenarios
Ek dashboard query 30 sec le rahi hai, kya-kya check karoge?
Production me table lock ho gayi, kaise investigate karoge?
Report query me wrong totals aa rahe hain, join se rows duplicate toh nahi ho rahe? Kaise verify karoge?
10 crore rows ki table me data archive kaise karoge?
Ek query ko bina index add kiye kaise fast karoge?
Read-heavy vs write-heavy system ke liye DB design me kya change karoge?
Ek table me 500 columns hain, kya problem hai aur kaise fix karoge?
19. Database Design Scenarios
Instagram-style follow/follower system ka schema.
Many-to-many relationship (students–courses) ka design.
Multi-tenant SaaS ke liye schema strategies.
Soft delete vs hard delete?
Order history me price change hone par purana price kaise preserve karoge?
Hierarchical data (comments, categories) store karne ke tareeke: adjacency list, nested set, materialized path.
Notification system ka schema (read/unread, types).
Inventory + warehouse + stock movement ka design.
20. MySQL vs PostgreSQL (tumhare stack ke liye)
AUTO_INCREMENT vs SERIAL / IDENTITY?
JSON/JSONB columns: query aur indexing.
Postgres me RETURNING clause ka use?
MySQL InnoDB vs MyISAM?
Full-text search dono me kaise hota hai?
Postgres partial index aur expression index?
Partitioning (range, list, hash) kab use karoge?
Connection pooling kyun zaroori hai?
21. Rapid-Fire Theory
Cardinality vs selectivity?
OLTP vs OLAP?
Sharding vs replication vs partitioning?
CAP theorem?
SQL injection kya hai aur kaise rokte ho?
Prepared statements ke fayde?
Temp table vs table variable?
DISTINCT vs GROUP BY?
Self-referencing foreign key?
Orphan records kya hote hain?