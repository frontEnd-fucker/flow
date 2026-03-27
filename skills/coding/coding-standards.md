# implementer subagent提示词模板

在调度implementer subagent时使用此模板。

```
Task tool (general-purpose):
    description: "执行 Task N: [task name]"
    prompt: ｜
    ## 开始前：
    向用户宣告："正在使用subagent执行任务-[任务名称]，模块类型：[通用模块/业务模块]，依赖任务：[依赖的任务内容]"

    ## 一、 规范

    ### 1. 头部注释
    所有新建或修改的源代码文件（`.ts`, `.tsx`, `.js` 等），**必须**在文件最顶部添加描述性注释。
    ```typescript
    /**
    * @file UserCard.tsx
    * @description 用户信息展示卡片，包含头像、昵称及关注按钮。支持骨架屏加载状态。
    */
    ```

    ### 2. 原型页面注释
    若文件为页面组件（路由组件），**必须**包含所使用的原型html和figma信息，如果没有则标明没有使用原型html或figma
    ```typescript
    /**
     * @file DetailPage.tsx
     * @description 详情页面
     * @原型 prototype/detail.html
     * @figma https://www.figma.com/design/cf1NMdWsFBbFuLfVslAwNe/Infrastructure-Projects----APP?node-id=2763-20457&m=dev
     */
    ```

    ### 3. 目录与文档保护
        a. **严禁删除/篡改**：
            - `docs/` 文件夹下的任何产品、规划、规范文档。
            - `prototype/` 文件夹下的原型资产。
        b. **项目初始化**：所有子任务相关的临时文件或新模块，必须初始化在当前工作区目录下，不得跨越项目边界。


    ## 二、 开发模式
    必须依据下表判定是否使用TDD开发模式：

    | 模块类型 | 判定特征 | 开发模式要求 | 示例 |
    | :--- | :--- | :--- | :--- |
    | **通用模块** | 可复用、无特定业务逻辑、多处引用 | ✅ **必须 TDD** (先写测试，再写实现) | `utils/`, `hooks/`, `components/ui/` |
    | **业务模块** | 特定页面、包含特定垂直业务逻辑 | ⏩ **直接实现** (跳过 TDD) | `pages/`, `features/profile/`, `services/` |

    > ⚠️ *无法确定类型时，默认按业务模块处理。*


    ## 三、 开发流程
    **重要约束**
        - 组件样式必须要参考对应的`prototype/`目录下原型文件。
        - **通用逻辑永远先写测试**: 通用逻辑不要跳过 TDD 流程

    步骤：
        1. 开始工作前，先检查是否有前置依赖需要等待
        2. 如果是TDD 开发流程：
                a. **必须遵循 TDD 开发流程**，先写测试，确保测试覆盖所有边界情况，再写实现。
                b. 运行测试确保通过.
        如果是非TDD开发流程:
                a. **跳过 TDD 流程**，直接实现功能.
                b. 关注功能完整性和用户体验.
        3. 任务完成后提交代码并合并代码到主分支
        4. 完成后报告：完成的功能、文件变更、测试覆盖情况（如适用）


    ## 四、 Subagent 交付与自检

    Subagent 完成任务向主 Agent 汇报时，**必须**附带以下合规清单：

        - [ ] **文件头注释**：是否已补齐文件说明？
        - [ ] **视觉锚点**：是否有应用[原型文件]？
        - [ ] **文档安全**：确认未误删 docs/ 和 prototype/ 文件夹，如果误删**必须恢复**。
```