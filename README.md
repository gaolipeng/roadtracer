RoadTracer Code
===============
RoadTracer不仅是一个软件包，它是对现代城市规划和地理信息系统的一次革新。该工具集成了先进的算法和深度学习模型，使得无须人工干预，即可将卫星图像转换为精准的道路图谱。结合其姊妹项目gomapinfer作为依赖，RoadTracer为您打开了通往未来城市数据处理的新大门。
RoadTracer 提供了一个从航拍图像中提取道路网络的数据集。 它由大量高分辨率卫星图像和地面实况道路网络图组成，覆盖了六个国家的四十个城市的城市核心。 对于每个城市，数据集覆盖了市中心周围约24 平方公里的区域。 卫星图像来自谷歌，分辨率为60 厘米/像素，道路网络来自OSM。

This is the code for ["RoadTracer: Automatic Extraction of Road Networks from Aerial Images"](https://roadmaps.csail.mit.edu/roadtracer/).

There are several components, and each folder has a README with more usage details:

* dataset: code for dataset preparation
* roadtracer: RoadTracer
* roadcnn: our segmentation approach (baseline)
* deeproadmapper: DeepRoadMapper (baseline)

You will need gomapinfer (https://github.com/mitroadmaps/gomapinfer/) as a dependency.

The training/inference code is built on top of TensorFlow.

Usage
-----

First, follow instructions in dataset/ to download the dataset.

Then, follow instructions in the other folders to train a model and run inference.

Junction Metric
---------------

The junction metric matches junctions (any vertex with three or more incident edges) between a ground truth road network graph and an inferred one.

	go run junction_metric.go /data/graphs/chicago.graph chicago.out.graph chicago

Visualization
-------------

`viz.go` will generate an SVG from a road network graph. It will refer to the `/data/testsat/` images; to view the SVG, those images will need to be in the same folder as the generated SVG.

	go run viz.go chicago /data/graphs/chicago.graph
	go run viz.go chicago chicago.out.graph

Applying RoadTracer on a new region
-----------------------------------

You need to make a few modifications to run the code on a region outside of the 40-city RoadTracer dataset.

First, download the imagery. Update `dataset/lib/regions.go` and put a latitude/longitude bounding box around your region in the regionMap. You can comment out the existing regions. Then, follow instructions in dataset/ for running `1_sat.go`.

Then, update `roadtracer/tileloader.py` and set `TRAINING_REGIONS` and `REGIONS` to just a list with your region label from the regionMap. Also update `REGION`, `TILE_START`, and `TILE_END` in `infer.py` (e.g. if your imagery tiles were saved as `xyz_-1_-1_sat.png` through `xyz_1_1_sat.png`, set `TILE_START = -1, -1` and `TILE_END = 2, 2`.

Finally, manually specify a starting location for RoadTracer in infer.py. Set `USE_TL_LOCATIONS = False`, `MANUAL_RELATIVE = 0, 0`, and set the manual points to the pixel positions of two points on the road network in `xyz_0_0_sat.png`. These two points should be close to each other (around SEGMENT_LENGTH apart) and best to be in the middle of a road.

Now you should be able to get a road network graph by running infer.py.

To convert this to latitude/longitude, you can use `dataset/convertarg.go`:

	go run convertarg.go YOUR_REGION_LABEL frompix out.graph out.lonlat.graph



### 贡献指南（Contribution Guide）
#### 1. 外部开发者如何贡献
- **Fork 仓库**：在 GitHub/GitLab 上 Fork 本项目到你自己的账号下。 
- **创建分支**：从 `main`（或 `dev`）分支拉出新分支，分支命名建议如下：  
  - 新功能：`feature/xxx`  
  - Bug 修复：`fix/xxx`  
  - 文档：`docs/xxx`  
- **本地开发**：  
  - 安装依赖：`pip install -r requirements.txt`  
  - 进行功能开发 / Bug 修复，并确保本地测试通过：`python manage.py test`（如有）
- **提交代码**：按下文“提交规范”编写 commit 信息并推送到你自己的远程仓库。 
- **发起 Pull Request**：在主页仓库中从你的分支发起 PR，按照“PR 描述格式”填写说明。
- **Code Review & 修改**：根据维护者的 Review 意见进行修改，直至 PR 被合并。

---

#### 2. 代码规范（Coding Style）
- **Python 版本与风格**
  - 使用 Python 3.8+。
  - 遵循 **PEP8** 代码风格（推荐使用 `flake8` 或 `ruff` 等工具进行检查）。  
- **Django 相关约定**
  - `views.py` 中避免写过重业务逻辑，复杂逻辑封装到 `services` / `utils` / `tools` 中。  
  - `models.py` 中字段命名清晰、含义明确，不使用难以理解的缩写。  
  - `serializers.py` 中保持字段与模型一致性，避免过多业务判断。  
- **命名与结构**
  - 变量 / 函数：使用小写加下划线，如 `get_user_token`。  
  - 类名：使用大驼峰，如 `UserProfileView`。  
  - 常量：全部大写加下划线，如 `DEFAULT_TIMEOUT`.  
- **注释与文档**
  - 关键逻辑处添加中文或英文注释，避免只靠命名猜测含义。  
  - 公共函数、类建议添加 docstring，说明用途与参数。  
- **测试**
  - 新功能和重要 Bug 修复，尽量补充或更新对应的单元测试。  
  - 确保 `python manage.py test`（或项目内的测试命令）能通过。 

---

#### 3. Pull Request 描述格式规范
建议 PR 的标题与描述遵循统一格式，便于 Review 和后期追踪。

- **PR 标题格式**

  推荐结构：`[类型] 简要说明`，例如：  
  - `[feat] 新增用户语音上传接口`  
  - `[fix] 修复社区列表接口分页错误`  
  - `[docs] 完善 README 贡献说明`  

- **PR 描述模板**

 建议在 PR 描述中包含以下内容（可直接复制作为模板）：  
   ```markdown
  ## 变更内容
  - [x] 简要说明本次 PR 做了哪些修改或新增了哪些功能
  - [ ] 如有接口变更，请写明请求方式、URL、参数和返回示例

  ## 相关 Issue
  - 关联 Issue：#123 （如果有）

  ## 测试情况
  - [ ] 已在本地运行 `python manage.py test`
  - [ ] 已在本地手动验证关键接口：
    - 接口 A：请求参数 / 返回结果
    - 接口 B：请求参数 / 返回结果

  ## 其他说明
  - 如有兼容性变更、迁移脚本或部署注意事项，请在此说明。
  ```

---

#### 4. Issue 提交规范
为便于定位问题和跟踪需求，提交 Issue 时建议遵循以下规则：
- **Issue 类型标签（Labels）**
  - `bug`：明确的错误或异常。  
  - `feature`：新功能或功能增强。  
  - `docs`：文档相关问题或改进建议。  
  - `question`：使用上的疑问或讨论。  
- **Issue 标题**
  - 简洁且能体现问题核心，例如：  
    - `bug: 用户登录接口返回 500 错误`  
    - `feature: 希望支持音频转文本功能`  

- **Issue 内容模板**
  ```markdown
  ## 类型
  - [ ] Bug
  - [ ] 新功能 / 改进建议
  - [ ] 文档 / 其他

  ## 问题或需求描述
  请用一两句话清晰描述你遇到的问题或希望实现的功能。

  ## 复现步骤（针对 Bug）
  1. 调用接口：`POST /api/v1/xxx`
  2. 请求参数：
     json
     {
       "example": "value"
     }
     
  3. 实际返回结果：
     json
     {
       "code": 500,
       "msg": "server error"
     }
     
  4. 预期结果：应该返回 200，并包含 xxx 字段。

  ## 运行环境
  - OS：Windows / macOS / Linux（及版本）
  - Python 版本：3.x
  - 数据库：SQLite / MySQL / PostgreSQL（及版本）
  - 其他相关信息：如是否使用 Docker 等

  ## 补充说明
  如有截图、日志（请注意脱敏）、错误堆栈，可在此补充。
   ```

### 下一步工作计划
to-do list is ready.