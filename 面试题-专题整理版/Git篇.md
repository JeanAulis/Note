[toc]

---

## 1. 你们公司怎么管理分支的？平常是怎么解决冲突的？

我们公司的分支管理是基于Git Flow，有下面几个分支

**master/main分支**：生产环境代码，只包含稳定版本

**develop分支**：开发主分支，包含最新的开发进度

**feature分支**：功能开发分支，从develop分出，完成后合并回develop

**release分支**：发版分支，从develop分出，用于发版前的最后调试

**hotfix分支**：紧急修复分支，从master分出，修复后同时合并到master和develop



### 解决冲突

优先进行沟通，从远端拉取代码，查看冲突标记，解决冲突部分，删除不需要的部分，保留需要的部分，测试好了之后commit代码，再推送到远端合代码

接受别人的，然后再慢慢合自己的





> [!important]
>
> ## Git 提交规范
>
> Angular 团队规范
>
> ```bash
> <type>(<scope>): <subject>
> 
> <body>
> 
> <footer>
> ```
>
> ```tex
> feat(module): 新增功能  
> fix(module): 修复bug  
> docs(module): 文档修改  
> refactor(module): 重构  
> style(module): 代码格式调整  
> perf(module): 性能优化  
> test(module): 单元测试变动  
> chore(module): 构建或依赖变更  
> build(module): 构建系统改动  
> ci(module): CI/CD流程调整
> ```
>
> | 类型         | 说明                                         |
> | ------------ | -------------------------------------------- |
> | **feat**     | 新功能（feature）                            |
> | **fix**      | 修复bug                                      |
> | **docs**     | 文档修改（如README）                         |
> | **style**    | 代码格式调整（不影响逻辑，如空格、缩进）     |
> | **refactor** | 重构（既不是新功能也不是修复）               |
> | **perf**     | 性能优化                                     |
> | **test**     | 增加或修改测试                               |
> | **chore**    | 构建过程或辅助工具变动（如依赖管理、CI配置） |
> | **revert**   | 回滚到某个提交                               |
> | **build**    | 项目构建、依赖变更                           |
> | **ci**       | CI/CD相关改动（如GitHub Actions、Jenkins）   |



> [!tip]
>
> - **PR:** Pull Request. 拉取请求，给其他项目提交代码
> - **LGTM:** Looks Good To Me. 朕知道了 代码已经过 review，可以合并
> - **SGTM:** Sounds Good To Me. 和上面那句意思差不多，也是已经通过了 review 的意思
> - **WIP:** Work In Progress. 传说中提 PR 的最佳实践是，如果你有个改动很大的 PR，可以在写了一部分的情况下先提交，但是在标题里写上 WIP，以告诉项目维护者这个功能还未完成，方便维护者提前 review 部分提交的代码。
> - **PTAL:** Please Take A Look. 你来瞅瞅？用来提示别人来看一下
> - **TBR:** To Be Reviewed. 提示维护者进行 review
> - **TL;DR:** Too Long; Didn't Read. 太长懒得看。也有很多文档在做简略描述之前会写这么一句
> - **TBD:** To Be Done (or Defined/Discussed/Decided/Determined). 根据语境不同意义有所区别，但一般都是还没搞定的意思