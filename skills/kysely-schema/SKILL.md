---
name: kysely-schema
description: 根据 MySQL DDL 生成数据库表类型与配套插入/更新方法的工作流
---

# 数据库表结构到代码生成工作流

## 适用场景

当拿到一份 `CREATE TABLE` 语句，需要在本仓库里补齐表结构类型，或者继续补齐对应的插入/更新方法时使用。

## 必守规则

- 新增表结构时，优先写显式的 `XxxDB`、`Selectable`、`Insertable`、`Updateable`，不要用 `ToGenerated` 偷懒。
- DDL 里只要出现 `default`，字段类型就不要再标 `| null`。
- 只有数据库明确允许 `NULL`，且没有默认值时，才写成 `type | null`。
- 自增主键、默认值字段、默认时间字段都要写成 `Generated<>`。
- 字段名用 camelCase，SQL 字段名和表名用 snake_case。
- 导出的类型、导出的函数、类的 `static` 方法都要补 JSDoc。
- 类型定义只放在 `src/types/database.d.ts`，增删改方法放在对应的 `*-modify.ts`，查询方法放在对应的 `*-query.ts`。
- 同一路径的 `import` / `import type` 只保留一条，别拆成多条。

## 第一步 添加表类型到 DB 接口

在 `src/types/database.d.ts` 的 `DB` 类型接口中添加新表，并把它放到对应业务分组的末尾。

```typescript
export type DB = {
  // monitor
  m_monitor_evaluation: MonitorEvaluationDB;
  m_monitor_evaluation_list: MonitorEvaluationListDB;
  m_monitor_evaluation_dimension: MonitorEvaluationDimensionDB;
};
```

## 第二步 创建表结构类型

表结构类型直接写成显式 Kysely 风格，不要再包 `ToGenerated`。

字段判断规则按下面处理。

- 自增主键写 `Generated<number>`
- 有默认值的字段写 `Generated<type>`
- 没有默认值、也不允许为空的字段写 `type`
- 允许为空、且没有默认值的字段才写 `type | null`

一个完整示例如下。

```typescript
/**
 * m_monitor_evaluation_list 课堂评价明细
 */
export type MonitorEvaluationListDB = {
  id: Generated<number>;
  monitorId: number; // 三率主记录ID
  dimensionId: number; // 维度ID
  score: string; // 课堂评价分数
  avgScore: string | null; // 平均分数
  type: Generated<number>; // 0 接口获取 1自己算
};
export type MonitorEvaluationList = Selectable<MonitorEvaluationListDB>;
export type InsertMonitorEvaluationListDB = Insertable<MonitorEvaluationListDB>;
export type UpdateMonitorEvaluationListDB = Updateable<MonitorEvaluationListDB>;
```

如果当前任务只要求补类型，到这里就可以停，不需要硬补 `modify` 方法。

## 第三步 在 modify 文件里导入新类型

在对应的 `src/service/**/modify.ts` 文件里，导入这个表对应的新增类型。

```typescript
import type { InsertMonitorEvaluationListDB, UpdateMonitorEvaluationListDB } from "#types/database.d.ts";
```

## 第四步 创建插入或更新方法

方法放在对应业务类里，写法跟现有项目保持一致。

```typescript
/**
 * 插入或更新课堂评价明细
 */
static async dbInsertMonitorEvaluationLists<T extends InsertMonitorEvaluationListDB>(
  params: DBInsertParams<T>,
) {
  const { dataList, tx = db } = params;
  if (isFalsy(dataList)) return [];

  const first = dataList[0];
  const updateField = {
    id: isValidValue(first.id),
    monitorId: isValidValue(first.monitorId),
    dimensionId: isValidValue(first.dimensionId),
    score: isValidValue(first.score),
    avgScore: isValidValue(first.avgScore),
    type: isValidValue(first.type),
  };

  const values = dataList.map((item) => ({
    id: item.id,
    monitorId: item.monitorId,
    dimensionId: item.dimensionId,
    score: item.score,
    avgScore: item.avgScore,
    type: item.type,
  }));

  const result = await DBUtil.upsert(
    tx.insertInto("m_monitor_evaluation_list").values(values),
    {
      id: updateField.id ? sql<number>`VALUES(id)` : undefined,
      monitorId: updateField.monitorId ? sql<number>`VALUES(monitor_id)` : undefined,
      dimensionId: updateField.dimensionId ? sql<number>`VALUES(dimension_id)` : undefined,
      score: updateField.score ? sql<string>`VALUES(score)` : undefined,
      avgScore: updateField.avgScore ? sql<string | null>`VALUES(avg_score)` : undefined,
      type: updateField.type ? sql<number>`VALUES(type)` : undefined,
    },
    {
      id: updateField.id ? sql.ref<number>("EXCLUDED.id") : undefined,
      monitorId: updateField.monitorId ? sql.ref<number>("EXCLUDED.monitor_id") : undefined,
      dimensionId: updateField.dimensionId ? sql.ref<number>("EXCLUDED.dimension_id") : undefined,
      score: updateField.score ? sql.ref<string>("EXCLUDED.score") : undefined,
      avgScore: updateField.avgScore ? sql.ref<string | null>("EXCLUDED.avg_score") : undefined,
      type: updateField.type ? sql.ref<number>("EXCLUDED.type") : undefined,
    },
  )
    .$call(debugSQL(false))
    .executeTakeFirst();

  return DBUtil.setInsertId(dataList, Number(result.insertId));
}
```

## 常见错误

- 新表直接套 `ToGenerated`。
- DDL 有 `default`，字段类型还写成 `| null`。
- `updateField`、`values`、`upsert` 参数漏字段。
- camelCase 和 snake_case 混着写。
- 导出函数和 `static` 方法没写 JSDoc。

## 验证清单

- DB 接口里已经加了新表类型
- 表结构类型是显式 `XxxDB` 写法
- 导出了 `Selectable`、`Insertable`、`Updateable`
- 有默认值的字段已经用了 `Generated<>`
- `default` 字段没有误标 `| null`
- `modify` 文件已经导入新类型
- 插入或更新方法字段映射完整
- 命名符合 camelCase / snake_case 约定
- JSDoc 已补齐
