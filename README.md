# 静电-拥挤粘度模型

本项目用于根据抗体序列、环境条件、离子种类、辅料参数以及可选结构信息，预测抗体在不同条件下的粘度或相关替代指标。

项目目前包含三套模型：

- `V1`：基于静电排斥与拥挤效应的半机理基线模型
- `V2`：基于序列/环境/结构/辅助指标特征的监督学习模型（`MLR` + `Random Forest`）
- `Regime`：按浓度范围分段的流变模型，预测 `η(c, T, pH, I, shear_rate)`

## 当前能力

### V1

`V1` 的四步流程为：

1. 根据 `VH/VL` 序列和环境 `pH` 计算净电荷
2. 根据离子屏蔽与 静电-拥挤 排斥层计算壳层厚度
3. 将几何体积分数膨胀为有效体积分数
4. 用宏观流变学方程映射为粘度

### V2

`V2` 在 `V1` 基础上加入：

- `Fv` 电荷特征
- 全长 `IgG` 电荷代理
- 电荷斑块特征
- 疏水/芳香聚集倾向特征
- 结构特征接口
- 离子种类与辅料参数
- `GDPa1` 中 `ACSINS` 作为辅助监督目标
- `MLR` 与 `Random Forest`

### Regime

`Regime` 模型面向 `η(c, T, pH, I, shear_rate)` 形式的流变预测：

1. `c < 30 mg/mL`：使用 `Einstein + Huggins` 稀溶液公式
2. `30 <= c < 100 mg/mL`：使用 `Ross-Minton + B2` 修正
3. `c >= 100 mg/mL`：使用高浓度 `cluster-entanglement` 修正
4. 温度项使用 `Andrade-Eyring`
5. 缓冲液/离子强度/pH 使用经验修正项
6. 剪切依赖使用 `Carreau-Yasuda`

该模型使用 `PDGF38` 的 `150 mg/mL` 粘度标签拟合序列/`SCM` 相关的残差项。由于 `PDGF38` 没有温度、pH、离子强度和剪切扫描，这些变量轴是机理先验外推，不是由 `PDGF38` 独立拟合得到。

## 新增功能

当前版本额外支持：

- 结构信息输入接口：`structure_file`、`structure_id`
- 无结构时的自动结构获取：
  - 优先使用用户给定结构
  - 若提供 `structure_id`，尝试从 AlphaFold DB 下载
  - 若无结构信息，可按序列调用 `ESMFold` 自动生成结构代理
- `GDPa1` 中 `AC-SINS` 值作为辅助目标，增强 `V2` 训练
- 指定序列在大范围环境变化下的趋势图预测
- 批量 CSV 预测接口
- 浓度分段流变模型训练与批量预测接口

## 目录结构

- `electrostatic_crowding_viscosity/`：核心代码
- `outputs/`：`V1` 默认结果
- `outputs_v2/`：`V2` 默认结果
- `outputs_regime/`：浓度分段流变模型默认结果
- `outputs_envtest/`：带辅料环境的 `V1` 示例结果
- `outputs_v2_envtest/`：带辅料环境的 `V2` 示例结果
- `trend_test/`：趋势图示例输出

## 依赖

依赖包如下：

- `numpy`
- `pandas`
- `openpyxl`
- `scikit-learn`
- `requests`
- `matplotlib`
- `biopython`

安装：

```powershell
pip install -r requirements.txt
```

## 最常用命令

### 1. 拟合 `V1`

```powershell
python -m electrostatic_crowding_viscosity.cli fit-pdgf38 --output-dir outputs
```

### 2. 运行 `V1` 监测

```powershell
python -m electrostatic_crowding_viscosity.cli monitor-gdpa1 --params outputs/reference_fit.json --output-dir outputs
```

### 3. 在指定离子/辅料环境下运行 `V1`

```powershell
python -m electrostatic_crowding_viscosity.cli monitor-gdpa1 --params outputs/reference_fit.json --output-dir outputs_envtest --default-ph 6.0 --default-salt 50 --default-concentration 150 --cation-species Na --anion-species Cl --excipient-name Arginine --excipient-concentration 50
```

### 4. 运行 `V2`

```powershell
python -m electrostatic_crowding_viscosity.cli run-v2 --output-dir outputs_v2
```

### 5. 在指定离子/辅料环境下运行 `V2`

```powershell
python -m electrostatic_crowding_viscosity.cli run-v2 --output-dir outputs_v2_envtest --cation-species Na --anion-species Cl --excipient-name Arginine --excipient-concentration 50
```

### 6. 批量预测

```powershell
python -m electrostatic_crowding_viscosity.cli predict-batch --input-file standard_input_template_full.csv --output-file predictions.csv --model-kind v2-both
```

### 7. 运行浓度分段流变模型

```powershell
python -m electrostatic_crowding_viscosity.cli run-regime --output-dir outputs_regime
```

### 8. 浓度分段流变模型批量预测

```powershell
python -m electrostatic_crowding_viscosity.cli predict-regime-batch --input-file standard_input_template_full.csv --output-file predictions_regime.csv --params outputs_regime/regime_parameters.json
```

### 9. 自动结构模式批量预测

```powershell
python -m electrostatic_crowding_viscosity.cli predict-batch --input-file standard_input_template_full.csv --output-file predictions_struct.csv --model-kind v2-mlr --structure-policy auto
```

### 10. 趋势图预测

```powershell
python -m electrostatic_crowding_viscosity.cli plot-trend --input-file standard_input_template_full.csv --output-dir trend_outputs --model-kind v2-both --vary pH --start 5.0 --end 8.0 --points 20
```

### 11. 双变量 heatmap 预测

```powershell
python -m electrostatic_crowding_viscosity.cli plot-heatmap --input-file standard_input_template_full.csv --output-dir heatmap_outputs --model-kind v2-both --vary-x pH --start-x 5.0 --end-x 8.0 --points-x 25 --vary-y salt_mM --start-y 20 --end-y 150 --points-y 20
```

## 环境参数

多个命令都支持以下环境输入：

- `cation_species`
- `anion_species`
- `excipient_name`
- `excipient_concentration`
- `excipient_net_charge_override`
- `excipient_hbond_donor_override`
- `excipient_hbond_acceptor_override`
- `excipient_logp_override`
- `excipient_aromatic_ring_override`
- `excipient_is_amino_acid_override`

`Regime` 模型额外支持：

- `temperature_C`
- `temperature_K`
- `ionic_strength_mM`
- `shear_rate_s`
- `scm`
- `scm_score`

其中 `predict-regime-batch` 会优先读取 `ionic_strength_mM`，若没有则使用 `salt_mM` 作为近似；若输入没有 `scm` 或 `scm_score`，会回退到训练集 `SCM` 均值。

## 结构输入

批量输入和趋势预测支持以下结构列：

- `structure_file`
- `structure_id`

其中：

- `structure_file`：本地已有结构文件路径
- `structure_id`：例如可用于 AlphaFold DB 下载的 accession

结构策略通过命令行参数 `--structure-policy` 指定：

- `off`：不使用结构特征
- `auto`：优先结构文件，否则自动按序列获取结构
- `full`：优先按全长链生成结构代理

自动结构获取带有本地缓存索引与失败重试机制：

- 缓存目录默认是 `structure_cache/`
- 缓存索引文件为 `structure_cache/structure_cache_index.json`
- 对同一 accession 或同一序列生成的结构会优先复用本地缓存
- 首次自动获取失败时会进行重试，并把结果写入缓存索引

## 输入模板

已提供三份模板：

- [standard_input_template_minimal.csv](/c:/Users/yiyang.su/project/Vicosity_prediction/electrostatic_crowding_viscosity_model/standard_input_template_minimal.csv)
- [standard_input_template_full.csv](/c:/Users/yiyang.su/project/Vicosity_prediction/electrostatic_crowding_viscosity_model/standard_input_template_full.csv)
- [standard_input_template.csv](/c:/Users/yiyang.su/project/Vicosity_prediction/electrostatic_crowding_viscosity_model/standard_input_template.csv)

其中：

- `minimal`：适合只提供序列和基本环境参数
- `full`：适合提供全量环境、结构和辅料覆盖参数
- `standard`：兼容旧版模板

## 批量预测输入 CSV 最少需要的列

必需：

- `vh_sequence`
- `vl_sequence`

推荐：

- `sample_id`
- `hc_subtype`
- `lc_subtype`
- `full_heavy_sequence_batch1`
- `full_light_sequence_batch1`
- `structure_file`
- `structure_id`
- `pH`
- `salt_mM`
- `concentration_mg_ml`
- `cation_species`
- `anion_species`
- `excipient_name`
- `excipient_concentration_mM`
- `temperature_C` 或 `temperature_K`
- `ionic_strength_mM`
- `shear_rate_s`
- `scm` 或 `scm_score`

## 输出文件

### V1 输出

- `reference_fit.json`
- `pdgf38_reference_predictions.csv`
- `gdpa1_default_predictions.csv`
- `gdpa1_environment_sweep.csv`
- `gdpa1_environment_summary.csv`
- `gdpa1_monitor_correlations.csv`
- `gdpa1_monitor_report.md`

### V2 输出

- `v2_pdgf38_feature_matrix.csv`
- `v2_loocv_predictions.csv`
- `v2_full_model_coefficients.csv`
- `v2_rf_feature_importances.csv`
- `v2_gdpa1_reference_predictions.csv`
- `v2_summary.json`
- `v2_report.md`

### Regime 输出

- `regime_parameters.json`
- `regime_pdgf38_feature_matrix.csv`
- `regime_loocv_predictions.csv`
- `regime_fitted_predictions.csv`
- `regime_full_model_coefficients.csv`
- `regime_concentration_grid.csv`
- `regime_summary.json`
- `regime_report.md`

默认训练结果位于：

- [outputs_regime](/c:/Users/yiyang.su/project/Vicosity_prediction/electrostatic_crowding_viscosity_model/outputs_regime)

### 趋势图输出

- `<sample_id>_<变量>_trend.csv`
- `<sample_id>_<变量>_trend.png`
- `<sample_id>_<变量>_trend_summary.md`

### Heatmap 输出

- `<sample_id>_<变量X>_vs_<变量Y>_heatmap.csv`
- `<sample_id>_<变量X>_vs_<变量Y>_<预测列>_heatmap.png`
- `<sample_id>_<变量X>_vs_<变量Y>_heatmap_summary.md`

## 说明文档

- 详细 CLI 参数说明：
  [CLI_调用说明_中文.md](/c:/Users/yiyang.su/project/Vicosity_prediction/electrostatic_crowding_viscosity_model/CLI_调用说明_中文.md)
- 精简速查表：
  [CLI_速查表_中文.md](/c:/Users/yiyang.su/project/Vicosity_prediction/electrostatic_crowding_viscosity_model/CLI_速查表_中文.md)

## 当前边界

- `PDGF38` 仍是当前粘度标签的主训练集
- `GDPa1` 主要提供 `ACSINS` 辅助监督和大规模监测
- 结构特征已接入，但全量训练默认不自动抓取结构，避免大规模网络请求导致耗时过长
- `Regime` 模型的 `c/T/pH/I/shear_rate` 轴目前主要是机理先验外推；`PDGF38` 只校准 `150 mg/mL` 参考条件下的粘度残差
- 若要进一步提升离子/辅料条件下的精度，仍建议补充真实多环境粘度数据

