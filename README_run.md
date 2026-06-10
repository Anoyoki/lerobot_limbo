# LeRobot PI05 Training Run Guide

本文档记录如何在服务器上运行 `limboyo/lerobot_openarm` 数据集的 PI05 训练任务，并保证在 VS Code SSH 断开后训练仍然继续运行。

## 1. 进入项目目录并激活环境

```bash
cd /home/yanglinbo/lerobot_limbo
conda activate lerobot
```

## 2. 配置 WandB

如需将训练日志同步到 WandB，请先登录（在网页获取 API key）：

```bash
wandb login --relogin
```

检查 WandB 状态：

```bash
wandb status
```

如果 `api_key` 不是 `null`，说明配置成功。

## 3. 配置 Hugging Face Token

若下载模型时出现：

```
Warning: You are sending unauthenticated requests to the HF Hub.
```

请登录 Hugging Face：

```bash
hf auth login
hf auth whoami
```

## 4. 在后台运行训练（推荐使用 `nohup`）

推荐使用 `nohup`，这样即使 SSH 断开训练也会继续运行。示例命令：

```bash
cd /home/yanglinbo/lerobot_limbo
# 查看显卡使用情况
nvidia-smi
# 指定使用的显卡（可选）
export CUDA_VISIBLE_DEVICES=4

nohup lerobot-train \
    --dataset.repo_id=limboyo/lerobot_openarm \
    --dataset.root=/home/yanglinbo/lerobot_limbo/data/bi_openarm_test \
    --dataset.revision=v3.0 \
    --policy.type=pi05 \
    --output_dir=output_lerobot_train/shake/pi05_A \
    --job_name=shake_pi05_A \
    --policy.pretrained_path=lerobot/pi05_base \
    --policy.compile_model=true \
    --policy.gradient_checkpointing=true \
    --wandb.enable=true \
    --wandb.project=Lerobot_limboyo_Project \
    --policy.dtype=bfloat16 \
    --policy.freeze_vision_encoder=false \
    --policy.train_expert_only=false \
    --steps=50000 \
    --policy.push_to_hub=false \
    --policy.device=cuda \
    --batch_size=32 \
    --policy.max_action_dim=48 \
    --policy.max_state_dim=48 \
    > train_pi05_A.log 2>&1 &
```

说明：

- `nohup`：让程序在 SSH 断开后继续运行。
- `> train_pi05_A.log`：将标准输出写入日志文件。
- `2>&1`：将错误输出也写入同一个日志文件。
- `&`：放到后台运行。

本数据集为双臂 OpenArm 数据，`action` 和 `observation.state` 均为 48 维，因此必须设置：

```text
--policy.max_action_dim=48
--policy.max_state_dim=48
```

## 5. 查看与停止训练

查找运行中的训练进程 PID：

```bash
ps -ef | grep lerobot-train | grep -v grep
```

示例输出：

```
yanglinbo 1857339 ... lerobot-train ...
```

其中 `1857339` 即为 PID。正常停止进程：

```bash
kill 1857339
```

如果需要我也可以替你将该文件提交到 Git（`git add` / `commit` / `push`）。