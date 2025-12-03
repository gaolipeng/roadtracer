RoadTracer Code
===============

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
