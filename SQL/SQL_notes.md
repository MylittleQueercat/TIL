# SQL 笔记

## 核心概念

**SQL表 = C里的结构体数组**
```c
struct Job { int id; char company[100]; char status[20]; };
struct Job jobs[1000];
```
```sql
SELECT id, company, status FROM jobs;
```
---
## 1. SELECT — 查数据
```sql
-- 查所有字段
SELECT * FROM jobs;
-- 查指定字段
SELECT company, position, status FROM jobs;
```
---
## 2. WHERE — 过滤
```sql
SELECT company, position FROM jobs
WHERE status = 'applied';
```
类比C：`if (job.status == "applied")`

⚠️ 字符串必须用**单引号** `'`，双引号 `"` 在SQL里是标识符（表名/列名），PostgreSQL会报错。

---
## 3. GROUP BY + COUNT — 统计
```sql
SELECT status, COUNT(*) as total
FROM jobs
GROUP BY status;
```
类比C：遍历数组，用map统计每个status出现次数。
---
## 4. JOIN / LEFT JOIN — 关联两张表
```sql
-- JOIN：两边都有才显示
SELECT jobs.company, interviews.interview_type, interviews.date
FROM jobs
JOIN interviews ON interviews.job_id = jobs.id;
-- LEFT JOIN：左边全显示，右边没有填null
SELECT jobs.company, interviews.interview_type, interviews.date
FROM jobs
LEFT JOIN interviews ON interviews.job_id = jobs.id;
```
类比C：
```c
for (int i = 0; i < jobs_count; i++)
  for (int j = 0; j < interviews_count; j++)
    if (interviews[j].job_id == jobs[i].id)
      // 输出这一行
```
---
## 5. 子查询 — 查询里套查询
```sql
-- 有面试记录的job
SELECT company, position FROM jobs
WHERE id IN (SELECT job_id FROM interviews);
-- 没有面试记录的job
SELECT company, position FROM jobs
WHERE id NOT IN (SELECT job_id FROM interviews);
```
---
## 6. INSERT / UPDATE / DELETE — 增改删
```sql
-- 插入
INSERT INTO interviews (job_id, round, interview_type, date, notes)
VALUES (1, 1, 'phone', '2026-04-10', 'HR初筛');
-- 修改
UPDATE jobs SET status = 'interview'
WHERE company = 'Google';
-- 删除
DELETE FROM jobs WHERE company = 'Google';
```
---
## 7. LIKE — 模糊匹配
```sql
-- % 匹配任意字符（包括空）
WHERE name LIKE 'Y%'    -- 以Y开头
WHERE name LIKE '%land' -- 以land结尾
WHERE name LIKE '%anz%' -- 包含anz
```
类比C：`strstr()` / `startswith`

---
## 8. CONCAT — 字符串拼接
```sql
-- 拼接字符串
SELECT CONCAT(name, ' City') FROM world;

-- 在WHERE条件里动态拼接
WHERE capital = CONCAT(name, ' City')

-- 配合LIKE，动态生成匹配模式
WHERE capital LIKE CONCAT('%', name, '%')
```
类比C：`strcat()` / Python `+`

---
## 9. REPLACE — 字符串替换/删除
```sql
-- REPLACE(原字符串, 要找的部分, 替换成什么)
REPLACE(capital, name, '')   -- 把name从capital里删掉，''=空字符串=删除

-- 用途：提取扩展部分
-- Monaco-Ville → -Ville
SELECT name, REPLACE(capital, name, '') AS extension
FROM world
WHERE capital LIKE CONCAT(name, '%')
AND capital <> name;
```
类比Python：`capital.replace(name, '')`

---
## 10. IN — 多值匹配
```sql
-- 替代多个OR
WHERE name IN ('France', 'Germany', 'Italy')

-- 等价于：
WHERE name = 'France' OR name = 'Germany' OR name = 'Italy'
```

---
## 11. ROUND — 四舍五入
```sql
ROUND(population / 1000000, 2)    -- 保留2位小数
ROUND(gdp / population, -3)       -- 四舍五入到最近的1000
```
第二个参数：
- 正数 → 小数点后几位
- 负数 → 小数点左边（`-1`=十位，`-2`=百位，`-3`=千位）

---
## 12. LENGTH / LEFT — 字符串操作
```sql
LENGTH(name)       -- 字符串长度
LEFT(name, 1)      -- 取最左边N个字符（常用于取首字母）
```
```sql
-- 名字和首都长度相同
WHERE LENGTH(name) = LENGTH(capital)

-- 名字和首都首字母相同，但不是同一个词
WHERE LEFT(name, 1) = LEFT(capital, 1)
AND name <> capital
```

---
## 13. ORDER BY — 排序
```sql
-- 单列排序
ORDER BY yr DESC      -- 从大到小
ORDER BY winner       -- 字母顺序（默认ASC）

-- 多列排序（逗号分隔，依次生效）
ORDER BY yr DESC, winner

-- 用IN表达式控制特定值排到最后（利用IN返回0/1的特性）
ORDER BY subject IN ('chemistry', 'physics'), subject, winner
-- 非chemistry/physics返回0排前面，chemistry/physics返回1排后面
```

---
## 14. OR / AND 逻辑 — XOR模式
```sql
-- 满足A或B，但不能同时满足（异或）
WHERE (area > 3000000 OR population > 250000000)
AND NOT (area > 3000000 AND population > 250000000)
```
类比C：`(a || b) && !(a && b)`

---

## 查表结构（忘记字段名时用）
```sql
SELECT column_name FROM information_schema.columns
WHERE table_name = 'jobs';
```
---
## 大小写规范
SQL对大小写没有硬性要求，也可以遵循规范：
- **关键字大写**：SELECT、FROM、WHERE、JOIN…
- **表名、字段名小写**：jobs、company、status…
