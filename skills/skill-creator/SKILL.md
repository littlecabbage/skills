---
name: skill-creator
description: 创建有效技能的指南。当用户想要创建一个新技能（或更新现有技能）以通过专门知识、工作流程或工具集成来扩展Claude的能力时，应使用此技能。
license: 完整条款见LICENSE.txt
---

# 技能创建者

此技能提供创建有效技能的指导。

## 关于技能

技能是模块化、自包含的包，通过提供专门知识、工作流程和工具来扩展Claude的能力。可以将它们视为特定领域或任务的"入门指南"——它们将Claude从通用代理转变为配备了模型无法完全拥有的程序化知识的专门代理。

### 技能提供的功能

1. 专门工作流程 - 特定领域的多步骤程序
2. 工具集成 - 处理特定文件格式或API的指令
3. 领域专业知识 - 公司特定知识、模式、业务逻辑
4. 捆绑资源 - 用于复杂和重复任务的脚本、参考资料和资产

## 核心原则

### 简洁是关键

上下文窗口是公共资源。技能与其他Claude需要的一切共享上下文窗口：系统提示、对话历史、其他技能的元数据以及实际的用户请求。

**默认假设：Claude已经非常智能。** 只添加Claude还没有的上下文。对每条信息提出质疑："Claude真的需要这个解释吗？"以及"这个段落是否值得其令牌成本？"

优先选择简洁的示例而不是冗长的解释。

### 设置适当的自由度

将具体程度与任务的脆弱性和变异性相匹配：

**高自由度（基于文本的指令）**：当多个方法都有效、决策取决于上下文或启发式方法指导方法时使用。

**中等自由度（带参数的伪代码或脚本）**：当存在首选模式、一些变体是可以接受的或配置影响行为时使用。

**低自由度（特定脚本、少量参数）**：当操作脆弱且容易出错、一致性至关重要或必须遵循特定序列时使用。

可以将Claude想象成探索一条路径：有悬崖的狭窄桥梁需要特定的护栏（低自由度），而开阔的田野允许许多路线（高自由度）。

### 技能解剖

每个技能由必需的SKILL.md文件和可选的捆绑资源组成：

```
skill-name/
├── SKILL.md (必需)
│   ├── YAML前置数据元数据 (必需)
│   │   ├── name: (必需)
│   │   └── description: (必需)
│   └── Markdown指令 (必需)
└── 捆绑资源 (可选)
    ├── scripts/          - 可执行代码 (Python/Bash/等)
    ├── references/       - 文档，旨在根据需要加载到上下文中
    └── assets/           - 输出中使用的文件 (模板、图标、字体等)
```

#### SKILL.md (必需)

每个SKILL.md包含：

- **前置数据** (YAML)：包含`name`和`description`字段。这些是Claude读取以确定何时使用技能的唯一字段，因此非常重要的是要清楚全面地描述技能是什么以及何时应使用它。
- **主体** (Markdown)：使用技能的指令和指导。只有在技能触发后（如果有的话）才会加载。

#### 捆绑资源 (可选)

##### 脚本 (`scripts/`)

用于需要确定性可靠性或反复重写的任务的可执行代码 (Python/Bash/等)。

- **何时包含**：当相同的代码被反复重写或需要确定性可靠性时
- **示例**：`scripts/rotate_pdf.py` 用于PDF旋转任务
- **优势**：令牌高效、确定性，可能在不加载到上下文中就执行
- **注意**：Claude可能仍然需要读取脚本以进行修补或环境特定调整

##### 参考资料 (`references/`)

旨在根据需要加载到上下文中以告知Claude过程和思考的文档和参考资料。

- **何时包含**：用于Claude在工作时应参考的文档
- **示例**：`references/finance.md` 用于财务模式，`references/mnda.md` 用于公司NDA模板，`references/policies.md` 用于公司政策，`references/api_docs.md` 用于API规范
- **用例**：数据库模式、API文档、领域知识、公司政策、详细工作流程指南
- **优势**：保持SKILL.md简洁，只在Claude确定需要时加载
- **最佳实践**：如果文件很大（>10k字），在SKILL.md中包含grep搜索模式
- **避免重复**：信息应该位于SKILL.md或参考文件中，而不是两者兼有。除非真正核心于技能，否则优先使用参考文件存储详细信息——这保持SKILL.md简洁，同时使信息可发现而不会占用上下文窗口。只在SKILL.md中保留基本程序指令和工作流程指导；将详细参考资料、模式和示例移至参考文件。

##### 资产 (`assets/`)

不打算加载到上下文中，而是用于Claude产生的输出中的文件。

- **何时包含**：当技能需要将在最终输出中使用的文件时
- **示例**：`assets/logo.png` 用于品牌资产，`assets/slides.pptx` 用于PowerPoint模板，`assets/frontend-template/` 用于HTML/React样板，`assets/font.ttf` 用于排版
- **用例**：模板、图像、图标、样板代码、字体、被复制或修改的示例文档
- **优势**：将输出资源与文档分离，使Claude能够在不将它们加载到上下文中就使用文件

#### What to Not Include in a Skill

技能应该只包含直接支持其功能的必要文件。不要创建无关的文档或辅助文件，包括：

- README.md
- INSTALLATION_GUIDE.md
- QUICK_REFERENCE.md
- CHANGELOG.md
- 等。

技能应该只包含AI代理完成手头工作所需的信息。它不应该包含关于创建过程的辅助上下文、设置和测试程序、面向用户的文档等。创建额外的文档文件只会增加杂乱和混淆。

### 渐进披露设计原则

技能使用三级加载系统来高效管理上下文：

1. **元数据（名称+描述）** - 始终在上下文中（~100字）
2. **SKILL.md主体** - 当技能触发时（<5k字）
3. **捆绑资源** - 根据Claude需要（无限，因为脚本可以在不读取到上下文窗口的情况下执行）

#### 渐进披露模式

保持SKILL.md主体简洁并低于500行以最小化上下文膨胀。当接近此限制时，将内容拆分到单独的文件中。当将内容拆分到其他文件中时，非常重要的是从SKILL.md引用它们并清楚描述何时读取它们，以确保技能的读者知道它们的存在以及何时使用它们。

**关键原则：** 当技能支持多个变体、框架或选项时，只在SKILL.md中保留核心工作流程和选择指导。将变体特定细节（模式、示例、配置）移至单独的参考文件中。

**模式1：带参考资料的高层指南**

```markdown
# PDF处理

## 快速开始

使用pdfplumber提取文本：
[代码示例]

## 高级功能

- **表单填写**：请参阅[FORMS.md](FORMS.md)以获取完整指南
- **API参考**：请参阅[REFERENCE.md](REFERENCE.md)以获取所有方法
- **示例**：请参阅[EXAMPLES.md](EXAMPLES.md)以获取常见模式
```

Claude仅在需要时加载FORMS.md、REFERENCE.md或EXAMPLES.md。

**模式2：特定领域组织**

对于具有多个领域的技能，按领域组织内容以避免加载无关上下文：

```
bigquery-skill/
├── SKILL.md (概述和导航)
└── reference/
    ├── finance.md (收入、计费指标)
    ├── sales.md (机会、管道)
    ├── product.md (API使用、功能)
    └── marketing.md (活动、归因)
```

当用户询问销售指标时，Claude只读取sales.md。

类似地，对于支持多个框架或变体的技能，按变体组织：

```
cloud-deploy/
├── SKILL.md (工作流程+提供商选择)
└── references/
    ├── aws.md (AWS部署模式)
    ├── gcp.md (GCP部署模式)
    └── azure.md (Azure部署模式)
```

当用户选择AWS时，Claude只读取aws.md。

**模式3：条件细节**

显示基本内容，链接到高级内容：

```markdown
# DOCX处理

## 创建文档

使用docx-js创建新文档。请参阅[DOCX-JS.md](DOCX-JS.md)。

## 编辑文档

对于简单编辑，直接修改XML。

**对于跟踪更改**：请参阅[REDLINING.md](REDLINING.md)
**对于OOXML细节**：请参阅[OOXML.md](OOXML.md)
```

Claude仅在用户需要这些功能时读取REDLINING.md或OOXML.md。

**重要指南：**

- **避免深度嵌套的参考资料** - 保持参考资料距离SKILL.md一级深度。所有参考文件都应该直接从SKILL.md链接。
- **构建较长的参考文件结构** - 对于超过100行的文件，在顶部包含目录，以便Claude在预览时看到完整范围。

## 技能创建过程

Skill creation involves these steps:

1. Understand the skill with concrete examples
2. Plan reusable skill contents (scripts, references, assets)
3. Initialize the skill (run init_skill.py)
4. Edit the skill (implement resources and write SKILL.md)
5. Package the skill (run package_skill.py)
6. Iterate based on real usage

Follow these steps in order, skipping only if there is a clear reason why they are not applicable.

### Step 1: Understanding the Skill with Concrete Examples

Skip this step only when the skill's usage patterns are already clearly understood. It remains valuable even when working with an existing skill.

To create an effective skill, clearly understand concrete examples of how the skill will be used. This understanding can come from either direct user examples or generated examples that are validated with user feedback.

For example, when building an image-editor skill, relevant questions include:

- "What functionality should the image-editor skill support? Editing, rotating, anything else?"
- "Can you give some examples of how this skill would be used?"
- "I can imagine users asking for things like 'Remove the red-eye from this image' or 'Rotate this image'. Are there other ways you imagine this skill being used?"
- "What would a user say that should trigger this skill?"

To avoid overwhelming users, avoid asking too many questions in a single message. Start with the most important questions and follow up as needed for better effectiveness.

Conclude this step when there is a clear sense of the functionality the skill should support.

### Step 2: Planning the Reusable Skill Contents

To turn concrete examples into an effective skill, analyze each example by:

1. Considering how to execute on the example from scratch
2. Identifying what scripts, references, and assets would be helpful when executing these workflows repeatedly

Example: When building a `pdf-editor` skill to handle queries like "Help me rotate this PDF," the analysis shows:

1. Rotating a PDF requires re-writing the same code each time
2. A `scripts/rotate_pdf.py` script would be helpful to store in the skill

Example: When designing a `frontend-webapp-builder` skill for queries like "Build me a todo app" or "Build me a dashboard to track my steps," the analysis shows:

1. Writing a frontend webapp requires the same boilerplate HTML/React each time
2. An `assets/hello-world/` template containing the boilerplate HTML/React project files would be helpful to store in the skill

Example: When building a `big-query` skill to handle queries like "How many users have logged in today?" the analysis shows:

1. Querying BigQuery requires re-discovering the table schemas and relationships each time
2. A `references/schema.md` file documenting the table schemas would be helpful to store in the skill

To establish the skill's contents, analyze each concrete example to create a list of the reusable resources to include: scripts, references, and assets.

### Step 3: Initializing the Skill

At this point, it is time to actually create the skill.

Skip this step only if the skill being developed already exists, and iteration or packaging is needed. In this case, continue to the next step.

When creating a new skill from scratch, always run the `init_skill.py` script. The script conveniently generates a new template skill directory that automatically includes everything a skill requires, making the skill creation process much more efficient and reliable.

Usage:

```bash
scripts/init_skill.py <skill-name> --path <output-directory>
```

The script:

- Creates the skill directory at the specified path
- Generates a SKILL.md template with proper frontmatter and TODO placeholders
- Creates example resource directories: `scripts/`, `references/`, and `assets/`
- Adds example files in each directory that can be customized or deleted

After initialization, customize or remove the generated SKILL.md and example files as needed.

### Step 4: Edit the Skill

When editing the (newly-generated or existing) skill, remember that the skill is being created for another instance of Claude to use. Include information that would be beneficial and non-obvious to Claude. Consider what procedural knowledge, domain-specific details, or reusable assets would help another Claude instance execute these tasks more effectively.

#### Learn Proven Design Patterns

Consult these helpful guides based on your skill's needs:

- **Multi-step processes**: See references/workflows.md for sequential workflows and conditional logic
- **Specific output formats or quality standards**: See references/output-patterns.md for template and example patterns

These files contain established best practices for effective skill design.

#### Start with Reusable Skill Contents

To begin implementation, start with the reusable resources identified above: `scripts/`, `references/`, and `assets/` files. Note that this step may require user input. For example, when implementing a `brand-guidelines` skill, the user may need to provide brand assets or templates to store in `assets/`, or documentation to store in `references/`.

Added scripts must be tested by actually running them to ensure there are no bugs and that the output matches what is expected. If there are many similar scripts, only a representative sample needs to be tested to ensure confidence that they all work while balancing time to completion.

Any example files and directories not needed for the skill should be deleted. The initialization script creates example files in `scripts/`, `references/`, and `assets/` to demonstrate structure, but most skills won't need all of them.

#### Update SKILL.md

**Writing Guidelines:** Always use imperative/infinitive form.

##### Frontmatter

Write the YAML frontmatter with `name` and `description`:

- `name`: The skill name
- `description`: This is the primary triggering mechanism for your skill, and helps Claude understand when to use the skill.
  - Include both what the Skill does and specific triggers/contexts for when to use it.
  - Include all "when to use" information here - Not in the body. The body is only loaded after triggering, so "When to Use This Skill" sections in the body are not helpful to Claude.
  - Example description for a `docx` skill: "Comprehensive document creation, editing, and analysis with support for tracked changes, comments, formatting preservation, and text extraction. Use when Claude needs to work with professional documents (.docx files) for: (1) Creating new documents, (2) Modifying or editing content, (3) Working with tracked changes, (4) Adding comments, or any other document tasks"

Do not include any other fields in YAML frontmatter.

##### Body

Write instructions for using the skill and its bundled resources.

### Step 5: Packaging a Skill

Once development of the skill is complete, it must be packaged into a distributable .skill file that gets shared with the user. The packaging process automatically validates the skill first to ensure it meets all requirements:

```bash
scripts/package_skill.py <path/to/skill-folder>
```

Optional output directory specification:

```bash
scripts/package_skill.py <path/to/skill-folder> ./dist
```

The packaging script will:

1. **Validate** the skill automatically, checking:

   - YAML frontmatter format and required fields
   - Skill naming conventions and directory structure
   - Description completeness and quality
   - File organization and resource references

2. **Package** the skill if validation passes, creating a .skill file named after the skill (e.g., `my-skill.skill`) that includes all files and maintains the proper directory structure for distribution. The .skill file is a zip file with a .skill extension.

If validation fails, the script will report the errors and exit without creating a package. Fix any validation errors and run the packaging command again.

### Step 6: Iterate

After testing the skill, users may request improvements. Often this happens right after using the skill, with fresh context of how the skill performed.

**Iteration workflow:**

1. Use the skill on real tasks
2. Notice struggles or inefficiencies
3. Identify how SKILL.md or bundled resources should be updated
4. Implement changes and test again
