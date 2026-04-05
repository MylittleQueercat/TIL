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

⚠️ 字符串里有单引号时，用两个单引号转义：
```sql
WHERE winner = 'EUGENE O''NEILL'
```

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

### 子查询用于比较（标量子查询）
```sql
-- 人均GDP高于英国的欧洲国家
SELECT name FROM world
WHERE continent = 'Europe'
AND gdp / population > (SELECT gdp / population FROM world
                        WHERE name = 'United Kingdom')
```
子查询返回单个值，可以直接用 `>` `<` `=` 比较。

### ALL — 与子查询结果的所有值比较
```sql
-- GDP高于欧洲所有国家
SELECT name FROM world
WHERE gdp > ALL(SELECT gdp FROM world
                WHERE continent = 'Europe'
                AND gdp IS NOT NULL)  -- ⚠️ 有NULL时必须过滤，否则比较失败
```

### 子查询用于IN（动态列表）
```sql
-- 找出包含Argentina或Australia的洲，再列出这些洲的所有国家
SELECT name, continent FROM world
WHERE continent IN (SELECT continent FROM world
                    WHERE name IN ('Argentina', 'Australia'))
ORDER BY name
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
WHERE name LIKE 'Y%'      -- 以Y开头
WHERE name LIKE '%land'   -- 以land结尾
WHERE name LIKE '%anz%'   -- 包含anz
WHERE name NOT LIKE '% %' -- 不含空格
```
类比C：`strstr()` / Python `startswith`

---
## 8. CONCAT — 字符串拼接
```sql
-- 拼接字符串
SELECT CONCAT(name, ' City') FROM world;

-- 在WHERE条件里动态拼接
WHERE capital = CONCAT(name, ' City')

-- 配合LIKE，动态生成匹配模式
WHERE capital LIKE CONCAT('%', name, '%')

-- 拼接百分号（显示百分比）
SELECT name, CONCAT(ROUND(population / (SELECT population FROM world
                           WHERE name = 'Germany') * 100, 0), '%')
FROM world
WHERE continent = 'Europe'
```
类比C：`strcat()` / Python `+`

---
## 9. REPLACE — 字符串替换/删除
```sql
-- REPLACE(原字符串, 要找的部分, 替换成什么)
REPLACE(capital, name, '')   -- 把name从capital里删掉，''=空字符串=删除

-- 用途：提取扩展部分（Monaco-Ville → -Ville）
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
## 15. 相关子查询（Correlated Subquery）
普通子查询只执行一次；相关子查询**每行执行一次**，内层查询依赖外层当前行。
```sql
-- 找每个洲面积最大的国家
SELECT continent, name, area FROM world x
WHERE area >= ALL(SELECT area FROM world y
                  WHERE x.continent = y.continent
                  AND area IS NOT NULL)
```

- `x` = 外层当前行
- `y` = 内层用来比较的所有行
- `x.continent = y.continent` = 只和同洲的行比较

类比C：
```c
for (int i = 0; i < n; i++) {           // x：外层每一行
    int max_area = 0;
    for (int j = 0; j < n; j++) {       // y：内层扫全表
        if (x[i].continent == y[j].continent)
            max_area = MAX(max_area, y[j].area);
    }
    if (x[i].area >= max_area)
        print(x[i]);
}
```

### 常见用法模式
```sql
-- 每组最大值（面积、人口等）
WHERE col >= ALL(SELECT col FROM world y
                 WHERE x.group = y.group
                 AND col IS NOT NULL)

-- 每组字母顺序第一
WHERE name <= ALL(SELECT name FROM world y
                  WHERE x.continent = y.continent)

-- 整组都满足条件（所有国家人口<=2500万的洲）
WHERE 25000000 >= ALL(SELECT population FROM world y
                      WHERE x.continent = y.continent
                      AND population IS NOT NULL)

-- 比同组所有其他成员都大N倍
WHERE population > ALL(SELECT population * 3 FROM world y
                       WHERE x.continent = y.continent
                       AND y.name <> x.name)  -- 排除自己
```

---
## 16. 聚合函数
```sql
SUM(gdp)        -- 求和
COUNT(*)        -- 计数（所有行）
COUNT(name)     -- 计数（非NULL的行）
AVG(population) -- 平均值
MAX(area)       -- 最大值
MIN(area)       -- 最小值
```
```sql
-- 非洲GDP总和
SELECT SUM(gdp) FROM world
WHERE continent = 'Africa'

-- 面积>=100万的国家数量
SELECT COUNT(*) FROM world
WHERE area >= 1000000

-- 每个洲的国家数量
SELECT continent, COUNT(*) FROM world
GROUP BY continent
```

---
## 17. DISTINCT — 去重
```sql
-- 每个洲只显示一次
SELECT DISTINCT continent FROM world
```
类比C：用 `set` 去重。

---
## 18. HAVING — 对分组结果过滤
```sql
-- 总人口超过1亿的洲
SELECT continent FROM world
GROUP BY continent
HAVING SUM(population) >= 100000000
```

**WHERE vs HAVING：**
- `WHERE` → 分组**前**过滤行
- `HAVING` → 分组**后**过滤组，可以用聚合函数
```sql
-- 组合使用：先过滤行，再过滤组
SELECT continent, COUNT(*) FROM world
WHERE population >= 10000000   -- 先筛掉人口<1000万的国家
GROUP BY continent
HAVING COUNT(*) >= 3           -- 再筛掉国家数<3的洲
```

---
## 不等于
```sql
<>  -- 标准写法
!=  -- 也支持，两者等价
```

## NULL处理
```sql
-- NULL不能用=比较，必须用IS
WHERE gdp IS NOT NULL
WHERE gdp IS NULL
```
⚠️ `ALL` 与子查询比较时，如果子查询结果含NULL，整个比较会失败，必须在子查询里过滤NULL。

---
## 查表结构（忘记字段名时用）
```sql
SELECT column_name FROM information_schema.columns
WHERE table_name = 'jobs';
```
---
## 大小写规范
SQL对大小写不敏感，但约定：
- **关键字大写**：SELECT、FROM、WHERE、JOIN…
- **表名、字段名小写**：jobs、company、status…
