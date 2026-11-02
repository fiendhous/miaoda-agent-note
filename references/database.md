## 数据库

`server/database/schema.ts` 由 `npm run gen:db-schema` 生成。手改会在下次生成时消失，也写不出条件索引和表达式索引。真实 DDL 放生成器碰不到的 SQL 文件里。

每张业务表、每一个字段都要有中文注释，含 `_created_at`、`_created_by`、`_updated_at`、`_updated_by`。这四个系统字段可以一次批量补，不必逐表重写。用 PostgreSQL 的 `COMMENT ON TABLE` 和 `COMMENT ON COLUMN`，和本次 DDL 一起执行。不要用 MySQL 那种写在列定义里的 `COMMENT`。注释写业务含义，不要只重复英文表名或字段名。

```sql
COMMENT ON TABLE example_order IS '订单，一门店一个营业日一行';
COMMENT ON COLUMN example_order.store_name IS '门店名称';
COMMENT ON COLUMN example_order.payload IS '@type { sku: string; qty: number }[] 下单明细';
```

JSONB 字段保留平台要求的 `@type { ... }`，中文说明写在同一条 `COMMENT` 里，不能用中文换掉 `@type`。

SQL 编辑器一次只跑一张表的注释。整文件一次贴进去会超时，注释里的斜杠还可能被当成语句分隔。不要给系统表 `pg_audit` 写注释。

执行顺序：`lark-cli apps +db-execute`，然后 `gen:db-schema`，重新读过 `schema.ts` 再写业务代码。

先看这个应用有没有拆出 dev：

- 拆了 dev：DDL 打在 dev，不要打 online。发布会把库结构一起带上。
- 只有一套库：DDL 打在这套库上。`--environment` 可以传 `online`，也可以不传；不传时单库落到 online，多环境落到 dev。不要传 `dev`，也不要执行 `+db-env-create`，那一步不可逆。`+env-pull` 若报 `Invalid DB Branch: dev`，就是没拆环境，不要因此去建 dev。

公共连接禁止 `WITH` 和多语句，DDL 自动提交，不能用 `BEGIN … ROLLBACK` 试探，也不能用 `pg_advisory_lock`。应用进程里已经跑通的 drizzle SQL 不受这条限制。人员字段只比较 `(字段).user_id`，没有 `.name`。不要把 JS 的 `Date` 绑进 SQL。UUID 数组写成 `ARRAY[$1::uuid, ...]::uuid[]`。`SELECT` 也要 `--yes`，见 `SKILL.md`「本机 lark-cli」。结果大约到 1000 行会截断，按人汇总用 `GROUP BY`。

平台的 postgres.js 会把 void 结果当 JSON 解析。`INSERT`、`UPDATE`、`DELETE` 不带 `RETURNING`，或查出 `pg_advisory_xact_lock` 这种 void 列，会抛 `Unexpected end of JSON input`，再变成 500。写语句带 `RETURNING`。要的值用 `json_build_object` 包出来。

SQL 要在真库跑一次，并确认结果不是全 0。全 0 和「条件恒不成立」长得一样。对 SQL 文本做 `toContain` 只锁住字符串，锁不住它能执行。drizzle 拼出来的是表名不是别名，外层再起别名会报 `42P01`。

本地查询脚本有 DML、没有 DDL。`--file` 的路径规则见 `SKILL.md`「本机 lark-cli」。

`schema.ts` 表达不了表达式索引和条件索引，生成时还会丢掉 `WHERE`，读起来像全表唯一，库里却是部分唯一。这两类索引用 SQL 文件维护。索引对账放 `tools/db-index-audit.js`，不要放进平台托管的 `scripts/`。

还有这些库结构事实：

- `EXTRACT` 跟随会话时区。写进 CHECK 要锚定 UTC。
- 条件唯一索引会在「先插新行再删旧行」的中间态撞上。
- `ON CONFLICT (id)` 挡不住业务唯一索引。
- `CREATE TABLE IF NOT EXISTS` 改不了已经存在的列默认值。
- `estimated_row_count` 是旧统计，不能用来判断表空不空。
- 清空数据前先查 `NO ACTION` 外键。
- 按一个所有行都相同的列排序，等于没有排序。
- 并存的两条唯一索引口径不同时，新写入会 500。先列出这张表的全部唯一索引。drizzle 把驱动错误包在 `cause` 上。
- 两个环境的 DDL 直接 `diff` 噪声很大，先把空白和标识符归一化。
- 服务端 `Promise.all` 不会让 drizzle 查询重叠。耗时约等于次数乘以往返。要快就减少查询次数。
- 应用角色没有 `BYPASSRLS`。表只开 RLS、策略默认拒绝时，正常读写也会失败。RLS 管不住 `TRUNCATE`。
## 系统自带的 pg_audit

`pg_audit` 是平台自带的变更日志，不是业务表。不要 `CREATE`、不要 `INSERT`、不要 `COMMENT ON`。Nest 运行时也不要建它。

业务表要在数据表的「记录日志」里打开「记录变更日志追溯」，平台才会往 `pg_audit` 写。开关只负责写入。

SQL 编辑器里 `SELECT * FROM pg_audit` 能看到行，应用直查经常是空的，这是行级权限，不是没数据。应用里要看操作日志时，在该应用的 SQL 编辑器手工建 `query_audit_logs`，函数用 `SECURITY DEFINER`，建在本应用自己的 schema 里。不要写成 `public.query_audit_logs`，`public` 会被拒。不要抄另一个应用的 schema 名。应用只调用这个函数，不直接读 `pg_audit`。

这个项目不做操作日志时，不要建函数，也不要在页面上留入口。把「不做」写进 `AGENTS.md`。

操作日志页的枚举要自己维护。`query_audit_logs` 返回英文表名和字段名，新增表或字段时一起补映射；查不到时回退原始值，不要留空白。`status` 这类值必须按表区分：订单的 `pending` 是待发货，报修明细的是待派单。用一份表清单的源码断言守住，漏一张表就挂测试。

平台 RBAC 授予容易、撤销难，不要把它当唯一安全边界。每个写接口自己校验。
