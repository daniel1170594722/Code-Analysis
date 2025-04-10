# FastDDS 代码库走读工作流

## 工作流概述

本工作流描述了为 **FastDDS 代码库**进行系统化走读分析的AI驱动流程。工作流采用结构化方式，由AI引导完成代码库的初始化扫描、构架分析、核心模块解读和文档生成。

## 核心原则

与通用代码库走读工作流相同，FastDDS 代码库走读工作流也遵循以下核心原则：

1. **原文尊重原则**
    * **绝对禁止** 修改任何源代码。
    * 保持原有的代码结构和组织方式。
    * 在分析前必须完整阅读或充分理解代码上下文。
    * 严格遵循 FastDDS 项目的编码规范和风格（如果明确）。

2. **文档同步原则**
    * 分析文档力求准确反映分析时的代码状态。
    * 重要发现必须及时记录和更新到分析文档中。
    * 图表必须反映分析时的代码结构。
    * 确保文档的可追溯性和准确性。

3. **上下文管理原则**
    * 维护全局代码上下文。
    * 追踪代码依赖关系（特别是 C++ 的 include 关系和类继承关系）。
    * 记录分析历史状态（通过进度文件）。
    * 管理临时分析结果。
    * 保持上下文连贯性。

4. **连续执行原则**
    * 每次迭代完成后立即开始下一次迭代。
    * 不停下来询问用户，保持连续性直到所有迭代完成。
    * 只在出现错误或整个工作流程完成时通知用户。

## 执行流程

### 阶段1：代码库初始化与结构生成

1. **创建输出目录**:
    * AI 首先创建输出目录：`mkdir -p output`
2. **项目扫描与结构生成**:
    * AI 执行预定义的命令或脚本来扫描项目目录，生成包含文件列表的结构文件。考虑到 FastDDS 是一个 C++ 项目，常用的命令可能包括：
        * 生成包含所有源文件和头文件的列表：`Get-ChildItem -Path src,include -Directory -Recurse | Sort-Object FullName | Select-Object FullName | Out-File -FilePath output\project_directories.txt `
        * 使用 `tree` 命令生成更详细的目录结构（限制深度）：`tree /F /A > output/project_structure.txt `
        * 如果项目使用 CMake，可以考虑解析 `CMakeLists.txt` 文件来理解项目结构和组件划分（更高级，可能需要额外的脚本或逻辑）。
    * AI 确认结构文件已生成。
3. **分析环境准备**:
    * 在 `output` 目录建立临时分析数据存储。
    * 初始化文档生成目录（`output` 目录）。
    * 准备使用 Mermaid 代码块生成图表。
    * 配置日志记录系统（可选，用于记录 AI 操作）。

### 阶段 2：全面项目分析与架构理解

1. **读取结构**: AI 读取阶段 1 生成的结构文件（例如 `output/project_structure.txt` 或 `output/project_files.txt`）。
2. **识别关键文件**: AI 根据结构文件，结合通用启发式规则和 **FastDDS 特有的知识** 识别出需要进行深入分析的关键文件和目录：
   * **入口文件**: 通常在 `src/` 或顶层目录，可能包含 `main` 函数的 `.cpp` 文件（如果存在可执行程序示例）。
   * **核心模块目录**:
     * `dds/`: 包含 DDS API 的实现。
     * `rtps/`: 包含 RTPS (Real-time Publish-Subscribe) 协议的实现。
     * `transport/`: 包含底层传输协议的实现（如 UDP, TCP）。
     * `security/`: 包含安全相关的实现（如果启用）。
     * `topic_domain/`: 包含主题和域相关的实现。
     * `qos/`: 包含服务质量 (QoS) 策略的实现。
     * `serialization/`: 包含数据序列化和反序列化的实现.
     * `idl/`: 可能包含 IDL (Interface Definition Language) 相关的处理代码。
   * **基础类和接口文件**: 查找文件名或目录名包含 `base`, `interface`, `abstract` 关键字的文件，以及常见的 DDS 概念类，如 `DomainParticipant`, `Publisher`, `Subscriber`, `DataWriter`, `DataReader`, `Topic`, `QosPolicy`.
   * **配置文件**: FastDDS 通常通过 API 或 XML 配置文件进行配置，需要关注相关的类或文件。
   * **工具和辅助类**: 查找 `utils/`, `common/`, `helper/` 等目录下的文件。
3. **系统性分析**: AI 依次读取识别出的关键文件（可能需要分多次读取大文件并结合上下文），进行深入分析，理解：
   * **项目的主要目标和功能**: 实现 DDS 和 RTPS 标准。
   * **核心组件/模块/类的职责和关键接口**: 例如，`DomainParticipantFactory` 的作用，`Publisher::create_datawriter` 的参数和返回值等。
   * **主要的执行流程和数据流**: 例如，发布者如何将数据发送给订阅者，发现机制是如何工作的。
   * **关键的数据结构或模型定义**: 例如，RTPS 协议消息的结构，DDS 数据样本的表示。
   * **配置管理方式**: 如何通过 API 或配置文件设置 QoS 等参数。
   * **主要的外部依赖及其用途**: 例如，可能依赖于操作系统 API、网络库等。
   * **组件间的依赖关系**: 例如，`dds/` 模块如何使用 `rtps/` 模块的功能。
4. **生成架构分析**: AI 将分析结果整理成一份结构化的 Markdown 文档（例如 `output/architecture_analysis.md`），包含系统概述、组件详解、流程说明等章节。该文件保存在工作区的 `output` 目录中。特别需要关注 FastDDS 的核心概念，如 DDS 规范、RTPS 协议、传输层选择等。
5. **制定走读计划**: 基于生成的架构分析，AI 制定一份适应性的代码走读计划。计划应明确每个迭代周期要分析的目标文件或模块，例如：
   * 迭代 1: 分析 `dds/domain/DomainParticipantImpl.hpp` 和 `.cpp`，理解域参与者的创建和管理。
   * 迭代 2: 分析 `rtps/participant/RTPSParticipantImpl.hpp` 和 `.cpp`，理解 RTPS 参与者的实现。
   * 迭代 3: 分析 `dds/pub/PublisherImpl.hpp` 和 `.cpp`，理解发布者的创建和管理。
   * 迭代 4: 分析 `rtps/writer/RTPSWriter.hpp` 和 `.cpp`，理解 RTPS 写入器的实现。
   * ...以此类推，覆盖核心 DDS 和 RTPS 概念。
6. **上下文准备**: AI 将架构分析的关键信息和走读计划载入内存（或准备随时读取），作为后续走读工作的上下文参考。AI **直接进入下一阶段**，无需用户审核分析指南。

### 阶段 3：迭代式代码走读

1. **初始化进度**: AI 创建或更新进度追踪文件 `output/reading_progress.json`，记录总计划和当前状态。
2. **连续迭代执行**: 根据阶段 2 制定的走读计划，AI 连续执行所有迭代周期，每个周期执行以下子步骤：
   * **a. 读取目标文件**: 按照计划读取当前迭代周期需要分析的文件或模块相关文件。
   * **b. 代码分析**: 结合架构上下文，深入分析代码实现细节：
     * 类和函数的职责（特别是 DDS API 和 RTPS 协议相关的接口）
     * 变量和参数的用途（例如 QoS 策略的成员变量）
     * 算法和实现逻辑（例如数据匹配算法、传输机制）
     * 错误处理机制（例如异常处理、状态码）
     * 性能考量点（例如内存管理、并发控制）
   * **c. 文档生成**: 为当前迭代周期创建独立的 Markdown 文档（例如 `output/iteration_1_analysis.md`），或在统一的走读文档中添加新章节，包含：
     * 概述和分析范围（例如，本次迭代分析了 `DomainParticipantImpl` 类）
     * 关键组件解析（例如，`DomainParticipantImpl` 的成员变量 `rtps_participant_` 的作用）
     * 实现特点分析（例如，`DomainParticipantImpl` 如何管理内部的 RTPS 参与者）
     * 代码示例（带行号和文件路径）
       ```cpp
       // src/dds/domain/DomainParticipantImpl.cpp
       DDS::DomainParticipantImpl::DomainParticipantImpl(
           DDS::DomainParticipantFactoryImpl* factory,
           const DDS::DomainParticipantQos& qos,
           DDS::DomainParticipantListener* a_listener,
           const DDS::StatusMask& mask)
           : ...
       {
           ...
       }
       ```
     * **Mermaid 图表代码块**（用于类图、序列图、流程图等）
       * 优先使用多个小图表而非一个复杂大图，每个小图聚焦于特定功能区域或组件关系
       * 小图应该有明确的标题和简短说明
       * 尽量控制每个图的节点数量在10个以内
       * **类图示例 (Mermaid):**
         ```mermaid
         classDiagram
           class DomainParticipantImpl {
             +rtps_participant_: RTPSParticipant*
             +qos_: DomainParticipantQos
             +listener_: DomainParticipantListener*
             +create_publisher(...)
             +create_subscriber(...)
             +create_topic(...)
             ...
           }
           class RTPSParticipant {
             <<interface>>
           }
           DomainParticipantImpl --|> RTPSParticipant : uses
         ```
       * **序列图示例 (Mermaid):**
         ```mermaid
         sequenceDiagram
           participant User
           participant DomainParticipantFactory
           participant DomainParticipantImpl
           User->>DomainParticipantFactory: create_participant(domain_id, qos, listener, mask)
           DomainParticipantFactory->>DomainParticipantImpl: new DomainParticipantImpl(factory, qos, listener, mask)
           DomainParticipantImpl-->>DomainParticipantFactory: returns DomainParticipantImpl
           DomainParticipantFactory-->>User: returns DomainParticipant
         ```
     * 关键发现和待讨论点
   * **d. 状态更新**: 在 `output/reading_progress.json` 中更新当前迭代周期的完成状态和任何备注。
   * **e. 立即开始下一个迭代**: 当前迭代完成后立即开始下一个迭代，无需暂停或等待用户指令，直到所有迭代完成。
3. **持续分析验证**: 在整个走读过程中，AI 持续检查：
   * 分析是否覆盖了计划中的关键文件/模块
   * 文档与代码的一致性
   * Mermaid 图表代码是否正确表达结构或流程
   * 分析深度是否足够（是否理解了核心逻辑和设计意图）

### 阶段 4：总结与最终文档生成

1. **整体分析汇总**: 当所有计划的迭代周期完成后，AI 生成一份综合分析报告 `output/final_report.md`，包含：
   * **导航**: 链接到各个迭代周期的分析文档或章节。
   * **系统架构总结**: 基于 `architecture_analysis.md` 和迭代分析的最终理解，重点描述 FastDDS 的整体架构，包括 DDS API 层、RTPS 层、传输层等。
   * **关键组件及其职责总结**: 详细描述 `DomainParticipant`, `Publisher`, `Subscriber`, `Topic`, `DataWriter`, `DataReader` 等核心 DDS 概念在 FastDDS 中的实现和职责。
   * **主要流程和数据流总结**: 描述发布/订阅流程、发现机制、QoS 处理等关键流程。
   * **设计模式和实现特点**: 分析 FastDDS 中使用的设计模式（例如工厂模式、策略模式）以及其实现上的特点（例如线程模型、内存管理）。
   * **潜在问题和改进建议**: 基于代码走读的理解，提出可能存在的性能瓶颈、代码可读性问题或潜在的改进方向。
   * **代码质量初步评估**: 对代码的组织结构、注释、错误处理等方面进行初步评估。
   * **项目结构图**: 使用 **Mermaid 或 Graphviz 代码块** 展示整体结构或核心依赖关系。例如，可以展示模块之间的依赖关系或者关键类的继承关系。
     * **模块依赖图 (Mermaid):**
       ```mermaid
       graph LR
         DDS --o RTPS
         RTPS --o Transport
         Security --o RTPS
         TopicDomain --o DDS
         QoS --o DDS
         Serialization --o RTPS
       ```
     * **核心类继承图 (Mermaid):**
       ```mermaid
       classDiagram
         DomainParticipantImpl --|> DomainParticipant
         PublisherImpl --|> Publisher
         SubscriberImpl --|> Subscriber
         DataWriterImpl --|> DataWriter
         DataReaderImpl --|> DataReader
       ```
   * **工作流执行摘要**:
     * 扫描的项目路径
     * 生成的结构文件名和架构分析文件名
     * 处理的文件/模块总数
     * 生成的文档和图表清单
     * 工作流执行的总时长（估算）
     * 未完全覆盖或理解的区域（如有）

2. **清理临时文件** (可选): AI 可以提议删除或归档在 `output` 目录中生成的中间文件（如迭代分析文档，如果它们已被汇总）。

## 工作流使用说明

1. **启动工作流**:
   * 在编辑环境中输入指令："开始 FastDDS 代码库走读工作流" (或类似指令)。
   * AI 将首先询问或确认用于生成项目结构文件的命令/脚本（可以根据 FastDDS 的实际情况进行选择）。
   * AI 开始执行阶段 1 的初始化。

2. **监控过程**:
   * 观察 AI 的操作和工具输出，特别是阶段 2 生成的架构分析和阶段 3 的迭代分析文档/章节。
   * 关注 `output/reading_progress.json` 文件以了解进度。
   * 用户可以在任何时候通过消息中断流程或提供指导，但AI不会在迭代之间主动停下来询问。

3. **获取结果**:
   * 工作流完成后，AI 会提示最终报告 `output/final_report.md` 已生成。
   * 用户可以在 `output` 目录找到架构分析、最终报告（包含导航和执行摘要）以及可能的迭代分析文档。

## 注意事项

1. **文件大小限制**
   * AI 工具可能对单次读取的文件大小有限制。对于非常大的文件，AI 应尝试分段读取和处理，并确保在分析前整合了足够的上下文。
   * 生成结构文件时建议限制扫描深度（如 `tree -L 4`），避免生成过大的结构文件。

2. **代码走读深度与计划**
   * 走读计划由 AI 在阶段 2 根据架构分析动态生成，不再是固定的天数。计划会更侧重于 FastDDS 的核心模块和概念。
   * 用户可以在计划生成后提出调整建议，但AI会自动按计划连续执行所有迭代。

3. **状态维护**
   * AI 通过 `output/reading_progress.json` 文件维护走读进度。
   * 支持中断后基于进度文件尝试恢复处理。

4. **图表设计建议**
   * 复杂系统应拆分为多个聚焦的小图，而非一个大而全的图表。
   * 每个小图应有清晰的目标，专注于一个特定方面（如类关系、模块依赖、消息流程）。
   * 图表颜色和样式保持一致，便于理解。
   * 考虑到 C++ 的特性，类图和继承关系图在 FastDDS 的分析中可能尤为重要。

## 错误处理

1. **分析错误**
   * 如果 AI 在分析过程中遇到复杂或难以理解的代码（例如模板元编程、复杂的继承结构），应在分析文档中记录，并在进度文件中标记。
   * AI会继续执行后续迭代，除非错误严重影响整体分析。
   * 用户可以在发现问题时通过消息中断流程并提供指导。

2. **工具调用错误** (如文件读取、结构生成失败)
   * 提供错误详情。
   * 建议可能的解决方案（如检查命令、文件权限）。
   * 在这种情况下AI会暂停执行，等待用户确认或提供修正指令后继续。
