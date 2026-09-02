# 2026H2W9 作业：把一门得到课程接入「得到大脑」GetNote

- **提交人**：Wendy（wendyzhou0915-arch）
- **提交日期**：2026-09-02
- **作业周期**：2026-08-31 ~ 2026-09-06

---

## 一、作业主题

练一次 Skill 串联工作流，把「浏览器发现已发布正文 URL → 本地清单去重预检 → GetNote 提取全文 → 异步状态核验」这条链路跑通，而不是复制课程内容本身。

## 二、实际执行的串联链路

| 步骤 | 动作 | 用的能力 | 结果 |
|---|---|---|---|
| ① 浏览器发现 URL | 在已登录得到的课程目录页，逐篇复制已发布正文链接（用户配合手动提供，未触碰 Cookie/登录态） | 浏览器 + 用户协作 | ✅ 拿到 15 篇链接 |
| ② 本地清单整理 | 将链接整理为 `course-articles.json`（抽取 enid、规范为 `https://www.dedao.cn/course/article?id=<enid>`、去重校验） | Python 脚本 | ✅ 15 篇，0 重复 |
| ③ 预检去重（dry-run） | 调用 `sync_getnote.py --dry-run`，联网扫描 GetNote 现有笔记做去重 | getnote + 技能脚本 | ✅ GetNote 现有 0 重复，15 篇全部 pending |
| ④ 同步 GetNote | 执行 `getnote save` / `sync_getnote.py` 提取全文 | getnote CLI | ❌ 15 篇全部失败 |
| ⑤ 异步核验 | 调用 `getnote notes` / `getnote search` 核验同步结果 | getnote CLI | ⚠️ 无新增内容（印证步骤④失败） |

## 三、关键数据（已脱敏）

- 课程：得到《薛兆丰的经济学课之成本解读》（用户已有权访问的付费课）
- 用户提供已发布正文链接数：**15 篇**
- 清单内重复：**0**
- GetNote 现有重复（dry-run）：**0**
- 待新增（dry-run）：**15**
- 同步成功：**0**
- 同步失败：**15**（全部 `status=failed` / "生成笔记失败"）

> 脱敏说明：本报告不含完整 URL 清单、enid 以 `zl12vGeN…` 形式局部展示，不含 Cookie、密钥、登录响应、GetNote 笔记全文。

## 四、卡点与根因（如实记录）

`getnote` CLI 对 dedao 课程正文抓取**全部失败**。过程中测试了 6 种 dedao URL 形态（`article?id=` / `articleid=` / `id=` / `detail?id=` 及其组合），统一返回 `生成笔记失败`；而通用网页 `example.com` 抓取**成功**（返回完整笔记），证明 getnote 服务端抓取能力本身正常。

即便已在 GetNote 网页登录并「连接得到账号」，dedao 课程正文仍返回失败。原 skill 设计依赖「浏览器插件复用登录态」完成 dedao 正文提取——本环境没有该插件，且作业红线禁止导出 Cookie，因此纯 CLI 路径与 GetNote 网页导入入口（仅支持音视频文件拖拽，无课程/URL 导入）均无法把 dedao 正文入库。

**结论**：工作流前半段（发现 → 清单 → 预检）已完整跑通并验证；全文提取这步因外部工具对 dedao 的授权/抓取缺口而未完成，非操作失误。

## 五、安全与边界

- 全程未导出 Cookie、登录响应、付费墙后正文、GetNote 笔记全文；
- PR 不含完整 URL 清单、enid 已脱敏；
- 探路产生的 3 条 dedao 失败空笔记已清理至 GetNote 回收站（可恢复）。

## 六、本次收获

- 掌握了 `getnote` CLI 在 Windows 的可靠调用方式：需用 **PowerShell 原生 shell** 绕过 Git Bash 路径污染，且 `.cmd` 子进程需 `shell=True` 启动；
- 识别出 dedao 链接多种形态，并确认 `sync_getnote.py` 的 `canonical_dedao_url` 仅认 `article?id=`；
- 建立起对 Skill 工作流「能力可串联、但受外部授权约束」的边界认知——这也是本作业最值得记下的点。
