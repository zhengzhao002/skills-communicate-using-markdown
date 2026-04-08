graph TD
    Start([输入：密集的深度神经网络模型]) --> Stage1

    subgraph Stage1 [阶段一：非结构化剪枝 Unstructured Pruning]
        direction TB
        A1[计算层级自适应阈值] --> A2[学习感知比较]
        A2 -->|低于阈值且未在学习| A3[剔除权重置为零]
        A2 -->|高于阈值或仍在学习| A4[保留权重]
        A3 --> A5
        A4 --> A5
        A5[结合 ER/ERK 正则化惩罚进行 SGD 权重更新]
    end

    Stage1 --> Transition{达到过渡条件？<br>稀疏率达标或模型稳定}

    Transition -->|否 (继续非结构化剪枝)| Stage1
    Transition -->|是 (触发切换)| Stage2

    subgraph Stage2 [阶段二：动态分层 N:M 结构化剪枝 Structured Pruning]
        direction TB
        B1[将权重按固定大小 M 进行分组] --> B2[计算层级阈值 & 学习感知比较]
        B2 --> B3[局部试探：统计各组剩余的非零权重数]
        B3 --> B4[多数投票：确定该层统一的非零保留数 N]
        B4 --> B5[结构化强制截断：每组仅保留前 N 个最大权重]
        B5 --> B6[结合 N:M 结构化正则化惩罚进行 SGD 权重更新]
    end

    Stage2 --> CheckEnd{达到最终目标稀疏率？}

    CheckEnd -->|否 (递减 N, 提高稀疏度)| Stage2
    CheckEnd -->|是 (训练结束)| End([输出：硬件友好的 N:M 结构化稀疏模型])
