# media_dir 使用说明

## 概述

现在支持在 `dataset_info.json` 中为每个数据集单独配置 `media_dir`，这样可以为不同的数据集指定不同的图片/视频/音频路径。

## 功能说明

### 优先级

1. **数据集级别的 `media_dir`**：在 `dataset_info.json` 中为特定数据集配置的 `media_dir`
2. **全局 `media_dir`**：训练参数中指定的全局 `media_dir`
3. **默认值**：如果都未指定，则使用 `dataset_dir`

### 图片路径构建逻辑

实际图片路径 = `media_dir` + `image_name`

例如：
- `media_dir`: `/mnt/jfs5/llamafactory_train_data_style_processed/caption-des_sg`
- `image_name`: `images/000593887-272271-0.png`
- 实际路径: `/mnt/jfs5/llamafactory_train_data_style_processed/caption-des_sg/images/000593887-272271-0.png`

## 配置示例

### dataset_info.json 配置

```json
{
  "caption-des_sg": {
    "file_name": "caption-des_sg-del2k-rand.jsonl",
    "formatting": "sharegpt",
    "columns": {
      "messages": "conversations",
      "images": "image_name"
    },
    "tags": {
      "role_tag": "from",
      "content_tag": "value",
      "user_tag": "human",
      "assistant_tag": "gpt"
    },
    "media_dir": "/mnt/jfs5/llamafactory_train_data_style_processed/caption-des_sg",
    "dataset_type": "caption数据"
  }
}
```

### JSONL 数据格式

```jsonl
{"conversations": [{"from": "human", "value": "<image>Could you please transcribe the image into a descriptive paragraph?"}, {"from": "gpt", "value": "The image features a tote bag..."}], "images": ["/mnt/jfs5/ui-understand-Original/caption/img/010674135.jpg"], "image_name": ["images/010674135-842636-0.png"]}
```

**注意**：
- `images` 字段可能包含原始图片路径（用于其他目的）
- `image_name` 字段（通过 `columns.images` 映射）包含相对于 `media_dir` 的路径

## 多数据集配置示例

当训练使用多个数据集时，可以为每个数据集配置不同的 `media_dir`：

```json
{
  "dataset1": {
    "file_name": "dataset1.jsonl",
    "formatting": "sharegpt",
    "columns": {
      "messages": "conversations",
      "images": "images"
    },
    "media_dir": "/path/to/dataset1/images"
  },
  "dataset2": {
    "file_name": "dataset2.jsonl",
    "formatting": "sharegpt",
    "columns": {
      "messages": "conversations",
      "images": "images"
    },
    "media_dir": "/path/to/dataset2/images"
  },
  "dataset3": {
    "file_name": "dataset3.jsonl",
    "formatting": "sharegpt",
    "columns": {
      "messages": "conversations",
      "images": "images"
    }
    // 未配置 media_dir，将使用全局 media_dir
  }
}
```

## 训练参数

在训练脚本或配置文件中，可以设置全局 `media_dir`（作为后备选项）：

```yaml
# examples/train_lora/qwen2vl_lora_sft.yaml
dataset: caption-des_sg
dataset_dir: data
media_dir: /default/path/to/media  # 可选：全局默认 media_dir
```

或在命令行中：

```bash
llamafactory-cli train \
  --model_name_or_path Qwen/Qwen2-VL-7B-Instruct \
  --dataset caption-des_sg \
  --dataset_dir data \
  --media_dir /default/path/to/media \
  ...
```

## 日志增强

在数据预处理阶段，现在会显示更详细的日志信息：

```
================================================================================
数据集加载统计信息:
================================================================================
数据集 'caption-des_sg': 1,000 个样本
数据集 'another_dataset': 500 个样本
--------------------------------------------------------------------------------
合并前总样本数: 1,500 个样本
应用 max_samples=1000 限制后: 1,000 个样本
================================================================================
合并后最终样本数: 1,000 个样本
================================================================================
```

## 注意事项

1. `media_dir` 必须是绝对路径或相对于工作目录的路径
2. `image_name` 字段中的路径应该是相对于 `media_dir` 的相对路径
3. 如果图片文件不存在，系统会输出调试日志并尝试使用原始路径
4. 支持图片、视频和音频文件

## 更新历史

- 2025-10-15: 添加数据集级别的 `media_dir` 支持
- 2025-10-15: 增强数据加载日志显示

