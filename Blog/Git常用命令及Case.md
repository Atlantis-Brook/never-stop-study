# Git 常用命令
revert

reset

## fast-forward

## rebase

## cherry-pick

# 经典Case
## rebase搭配ff使用

团队里有一个 `main` 分支作为主线开发，`feature/login` 分支是你新建的功能分支。

在你开发期间，其他同事已经往 `main` 提交了新的 commit。  
如果你直接合并，会出现一个 **merge commit**。为了保持 **提交历史简洁**，你希望能用 **rebase + fast-forward**。
## 初始情况

```bash
# main 分支提交历史
A -- B -- C    (main)

# 你的 feature 分支提交历史
A -- B -- C
          \
           D -- E   (feature/login)

```
- `C` 是最新的公共 commit
    
- `D`、`E` 是你在 `feature/login` 上的提交
    

此时团队已经在 `main` 上新增了一个提交 `F`：
``` bash
A -- B -- C -- F   (main)
          \
           D -- E   (feature/login)
```
---
## 操作步骤

### 1. 在本地更新 `main`

`git checkout main git pull origin main`

现在 `main` 是：

`A -- B -- C -- F   (main)`

---

### 2. 在功能分支上 rebase

`git checkout feature/login git rebase main`

**效果图**：

`A -- B -- C -- F   (main)                   \                    D' -- E'   (feature/login)`

- `D'`、`E'` 是 rebase 后“重新应用”的 commit（旧的 D、E 会被替换掉）。
    
- 这样 `feature/login` 就基于最新的 `main` 了。
    

---

### 3. 合并时用 fast-forward

切回主分支，然后执行 FF 合并：

`git checkout main git merge --ff-only feature/login`

最终结果：

`A -- B -- C -- F -- D' -- E'   (main, feature/login)`

- 没有多余的 **merge commit**
    
- 历史线非常干净，像是 **一条直线**
    

---

## 对比如果不用 rebase，直接 merge

`git checkout main git merge feature/login`

会变成：

`A -- B -- C -- F -----------------M   (main)           \                       /            D -- E   (feature/login)`

这里 `M` 是 **merge commit**，提交历史会多出一条分叉，看起来不如 rebase + FF 简洁。
### 推荐实践

1. **个人开发时**：在合并前先 rebase 到 `main`，再用 FF 合并，历史干净。
2. **团队协作时**：如果需要保留分支历史（审计、review），则用 `merge`，否则用 `rebase + FF` 保持简洁。