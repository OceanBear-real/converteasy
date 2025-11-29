# ConvertEasy Backend - Postman 测试说明

## 问题说明

Postman 集合中的测试失败是因为 `formdata` 中的 `file` 字段 `src` 为空字符串 `""`，导致没有实际上传文件。

## 如何修复

### 方法 1: 在 Postman UI 中手动选择文件

1. 打开 Postman
2. 导入 `postman_collection.json`
3. 对于每个上传请求（如 "Upload & Convert - PDF to DOCX"）：
   - 点击 **Body** 标签
   - 找到 `file` 字段
   - 点击 **Select Files** 按钮
   - 选择对应格式的测试文件（如 PDF、DOCX、TXT 等）
4. 运行测试

### 方法 2: 使用测试样例文件

我们已经在 `backend/tests/samples/` 中生成了测试样例：

```
backend/tests/samples/
├── sample.txt
├── sample.html
├── sample.docx  (需要运行 gen_samples.py)
├── sample.xlsx  (需要运行 gen_samples.py)
└── sample.wav   (需要运行 gen_samples.py)
```

生成更多样例：

```bash
cd backend
python tests/gen_samples.py
```

然后在 Postman 中选择这些文件进行测试。

### 方法 3: 使用 Newman (命令行运行)

Postman 集合需要实际文件路径，但 Newman 不支持在 JSON 中直接指定文件路径。需要使用环境变量或数据文件。

### 方法 4: 运行 Python 集成测试 (推荐)

我们已经创建了完整的集成测试套件，自动使用样例文件测试所有转换流程：

```bash
cd backend
pytest tests/test_integration.py -v
```

这些测试包括：

- ✅ TXT → PDF 转换
- ✅ HTML → PDF 转换
- ✅ 检测支持的目标格式
- ✅ 不支持转换的错误处理
- ✅ 格式不匹配错误检测
- ✅ 缺少文件错误检测
- ✅ 任务不存在错误检测

## 单元测试 vs 集成测试

### 单元测试 (`test_*.py`)

- 使用真实样例文件（`samples/sample.txt` 等）
- 测试 API 端点的基本功能
- 转换逻辑被 mock/stub，快速运行
- 运行：`pytest -q`

### 集成测试 (`test_integration.py`)

- 使用真实样例文件
- 测试完整转换流程（上传 → 处理 → 查询状态）
- 包含实际转换等待和轮询
- 测试错误场景（格式不匹配、缺少文件等）
- 运行：`pytest tests/test_integration.py -v`

## 建议

1. **开发测试**: 运行快速单元测试 `pytest -q`
2. **完整验证**: 运行集成测试 `pytest tests/test_integration.py -v`
3. **手动测试**: 使用 Postman UI 手动选择文件并测试
4. **CI/CD**: 集成测试可用于持续集成流水线

## Postman 集合的局限性

- 无法在 JSON 中嵌入二进制文件数据
- `src: ""` 需要在 Postman UI 中手动选择文件
- Newman (CLI) 需要额外配置才能上传文件
- 建议使用 Python 集成测试进行自动化验证
