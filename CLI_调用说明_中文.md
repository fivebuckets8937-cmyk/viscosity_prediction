# CLI 调用说明（中文）

本文档说明项目中所有命令行入口的用途、调用方式和参数含义。

统一调用格式：

```powershell
python -m electrostatic_crowding_viscosity.cli <子命令> [参数]
```

主入口文件：

- [cli.py](/c:/Users/yiyang.su/project/Vicosity_prediction/electrostatic_crowding_viscosity_model/electrostatic_crowding_viscosity/cli.py)

---

## 1. `fit-pdgf38`

用途：

- 在 `PDGF38` 数据集上拟合 `V1` 静电-拥挤半机理模型

示例：

```powershell
python -m electrostatic_crowding_viscosity.cli fit-pdgf38
```

参数：

| 参数 | 默认值 | 含义 |
| --- | --- | --- |
| `--dataset-root` | `../Dataset` | 数据目录 |
| `--output-dir` | `outputs` | 输出目录 |
| `--reference-ph` | `7.4` | 参考 `pH` |
| `--reference-concentration` | `150.0` | 参考蛋白浓度，单位 `mg/mL` |
| `--seed` | `13` | 随机种子 |
| `--broad-trials` | `30000` | 全局搜索次数 |
| `--local-trials` | `10000` | 局部搜索次数 |

输出：

- `reference_fit.json`
- `pdgf38_reference_predictions.csv`

---

## 2. `monitor-gdpa1`

用途：

- 使用 `V1` 参数在 `GDPa1` 上进行默认条件预测
- 输出环境扫描结果和监测报告

示例：

```powershell
python -m electrostatic_crowding_viscosity.cli monitor-gdpa1 --params outputs/reference_fit.json
```

参数：

| 参数 | 默认值 | 含义 |
| --- | --- | --- |
| `--dataset-root` | `../Dataset` | 数据目录 |
| `--params` | `outputs/reference_fit.json` | `V1` 参数文件 |
| `--output-dir` | `outputs` | 输出目录 |
| `--default-ph` | `6.0` | 默认监测 `pH` |
| `--default-salt` | `50.0` | 默认盐浓度，单位 `mM` |
| `--default-concentration` | `150.0` | 默认蛋白浓度，单位 `mg/mL` |
| `--cation-species` | `Na` | 阳离子种类 |
| `--anion-species` | `Cl` | 阴离子种类 |
| `--excipient-name` | `None` | 辅料名称 |
| `--excipient-concentration` | `0.0` | 辅料浓度，单位 `mM` |
| `--excipient-net-charge-override` | `None` | 手工覆盖辅料净电荷 |
| `--excipient-hbond-donor-override` | `None` | 手工覆盖辅料供氢键数 |
| `--excipient-hbond-acceptor-override` | `None` | 手工覆盖辅料受氢键数 |
| `--excipient-logp-override` | `None` | 手工覆盖辅料 `logP` |
| `--excipient-aromatic-ring-override` | `None` | 手工覆盖辅料芳香环数 |
| `--excipient-is-amino-acid-override` | `None` | 手工覆盖是否按氨基酸处理 |

输出：

- `gdpa1_default_predictions.csv`
- `gdpa1_environment_sweep.csv`
- `gdpa1_environment_summary.csv`
- `gdpa1_monitor_correlations.csv`
- `gdpa1_monitor_report.md`

---

## 3. `run-all`

用途：

- 依次执行：
  1. `fit-pdgf38`
  2. `monitor-gdpa1`

示例：

```powershell
python -m electrostatic_crowding_viscosity.cli run-all
```

参数：

兼容 `fit-pdgf38` 和 `monitor-gdpa1` 的常用参数。

---

## 4. `run-v2`

用途：

- 构建 `V2` 特征矩阵
- 在 `PDGF38` 上执行 `MLR` / `Random Forest` 训练与交叉验证
- 使用 `GDPa1` 的 `ACSINS` 作为辅助监督来源
- 对 `GDPa1` 输出 `V2` 推理结果

示例：

```powershell
python -m electrostatic_crowding_viscosity.cli run-v2 --output-dir outputs_v2
```

参数：

| 参数 | 默认值 | 含义 |
| --- | --- | --- |
| `--dataset-root` | `../Dataset` | 数据目录 |
| `--output-dir` | `outputs_v2` | 输出目录 |
| `--reference-ph` | `7.4` | 参考 `pH` |
| `--auxiliary-ph` | `6.0` | 辅助 `pH` |
| `--reference-salt` | `50.0` | 参考盐浓度 |
| `--reference-concentration` | `150.0` | 参考蛋白浓度 |
| `--max-features` | `3` | `MLR` 最大特征数 |
| `--cation-species` | `Na` | 阳离子种类 |
| `--anion-species` | `Cl` | 阴离子种类 |
| `--excipient-name` | `None` | 辅料名称 |
| `--excipient-concentration` | `0.0` | 辅料浓度 |
| `--excipient-net-charge-override` | `None` | 辅料净电荷覆盖 |
| `--excipient-hbond-donor-override` | `None` | 辅料供氢键覆盖 |
| `--excipient-hbond-acceptor-override` | `None` | 辅料受氢键覆盖 |
| `--excipient-logp-override` | `None` | 辅料 `logP` 覆盖 |
| `--excipient-aromatic-ring-override` | `None` | 辅料芳香环数覆盖 |
| `--excipient-is-amino-acid-override` | `None` | 是否按氨基酸处理 |
| `--structure-policy` | `off` | 结构策略：`off` / `auto` / `full` |
| `--structure-cache-dir` | `structure_cache` | 自动获取结构时的缓存目录 |

输出：

- `v2_pdgf38_feature_matrix.csv`
- `v2_loocv_predictions.csv`
- `v2_full_model_coefficients.csv`
- `v2_rf_feature_importances.csv`
- `v2_gdpa1_reference_predictions.csv`
- `v2_summary.json`
- `v2_report.md`

---

## 5. `run-regime`

用途：

- 基于浓度范围分段的流变模型训练一个新版本
- 低浓度使用 `Einstein + Huggins`
- 中等浓度使用 `Ross-Minton + B2` 修正
- 高浓度使用 `cluster-entanglement` 修正
- 温度使用 `Andrade-Eyring`
- 剪切依赖使用 `Carreau-Yasuda`
- 使用 `PDGF38` 的 `150 mg/mL` 粘度标签拟合序列/SCM 相关的残差项

示例：

```powershell
python -m electrostatic_crowding_viscosity.cli run-regime --output-dir outputs_regime
```

参数：

| 参数 | 默认值 | 含义 |
| --- | --- | --- |
| `--dataset-root` | `../Dataset` | 数据目录 |
| `--output-dir` | `outputs_regime` | 输出目录 |
| `--reference-concentration` | `150.0` | PDGF38 参考浓度，单位 `mg/mL` |
| `--reference-temperature-c` | `25.0` | 参考温度，单位 Celsius |
| `--reference-ph` | `7.4` | 参考 `pH` |
| `--reference-ionic-strength` | `50.0` | 参考离子强度，单位 `mM` |
| `--reference-shear-rate` | `0.0` | 参考剪切速率，单位 `1/s` |
| `--ridge-alpha` | `0.6` | 残差回归正则强度 |

输出：

- `regime_parameters.json`
- `regime_pdgf38_feature_matrix.csv`
- `regime_loocv_predictions.csv`
- `regime_fitted_predictions.csv`
- `regime_full_model_coefficients.csv`
- `regime_concentration_grid.csv`
- `regime_summary.json`
- `regime_report.md`

注意：

- `PDGF38` 只有 `150 mg/mL` 单点粘度标签，没有温度、pH、盐浓度和剪切扫描；这些轴在该模型中是机理先验和趋势外推，不是由 PDGF38 独立拟合得到。

---

## 6. `predict-regime-batch`

用途：

- 使用 `run-regime` 训练出的 `regime_parameters.json` 对输入 CSV 批量预测
- 支持输入 `concentration_mg_ml`、`temperature_C`/`temperature_K`、`pH`、`ionic_strength_mM`/`salt_mM`、`shear_rate_s`
- 若输入没有 `scm` 或 `scm_score`，模型会回退到训练集 SCM 均值

示例：

```powershell
python -m electrostatic_crowding_viscosity.cli predict-regime-batch --input-file standard_input_template_full.csv --output-file predictions_regime.csv --params outputs_regime/regime_parameters.json
```

参数：

| 参数 | 默认值 | 含义 |
| --- | --- | --- |
| `--input-file` | 无 | 输入 CSV |
| `--output-file` | 无 | 输出 CSV |
| `--params` | `outputs_regime/regime_parameters.json` | `run-regime` 输出的参数文件 |

---

## 7. `predict-batch`

用途：

- 从输入 CSV 批量预测结果
- 支持 `V1`、`V2-MLR`、`V2-RF`、`V2-both`

示例：

```powershell
python -m electrostatic_crowding_viscosity.cli predict-batch --input-file standard_input_template_full.csv --output-file predictions.csv --model-kind v2-both
```

参数：

| 参数 | 默认值 | 含义 |
| --- | --- | --- |
| `--input-file` | 无 | 输入 CSV |
| `--output-file` | 无 | 输出 CSV |
| `--dataset-root` | `../Dataset` | 数据目录 |
| `--model-kind` | `v2-both` | `v1` / `v2-mlr` / `v2-rf` / `v2-both` |
| `--params` | `outputs/reference_fit.json` | `V1` 参数文件 |
| `--reference-ph` | `7.4` | 默认 `pH` |
| `--auxiliary-ph` | `6.0` | `V2` 辅助 `pH` |
| `--reference-salt` | `50.0` | 默认盐浓度 |
| `--reference-concentration` | `150.0` | 默认蛋白浓度 |
| `--max-features` | `3` | `V2-MLR` 最大特征数 |
| `--cation-species` | `Na` | 默认阳离子 |
| `--anion-species` | `Cl` | 默认阴离子 |
| `--excipient-name` | `None` | 默认辅料 |
| `--excipient-concentration` | `0.0` | 默认辅料浓度 |
| 各类 `override` 参数 | `None` | 默认辅料属性覆盖 |
| `--structure-policy` | `auto` | 结构策略 |
| `--structure-cache-dir` | `structure_cache` | 结构缓存目录 |

说明：

- 如果输入 CSV 某一行已经包含环境列，则优先使用该行的值
- 缺失环境值时，回退到命令行默认参数

### 批量输入 CSV 推荐列

必需列：

- `vh_sequence`
- `vl_sequence`

推荐列：

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

模板文件：

- [standard_input_template_minimal.csv](/c:/Users/yiyang.su/project/Vicosity_prediction/electrostatic_crowding_viscosity_model/standard_input_template_minimal.csv)
- [standard_input_template_full.csv](/c:/Users/yiyang.su/project/Vicosity_prediction/electrostatic_crowding_viscosity_model/standard_input_template_full.csv)

---

## 8. `plot-trend`

用途：

- 针对某一个输入样本，在大范围环境变化下生成趋势预测曲线
- 同时输出趋势明细 CSV 和趋势图 PNG

示例：

```powershell
python -m electrostatic_crowding_viscosity.cli plot-trend --input-file standard_input_template_full.csv --output-dir trend_outputs --model-kind v2-both --vary pH --start 5.0 --end 8.0 --points 20
```

参数：

| 参数 | 默认值 | 含义 |
| --- | --- | --- |
| `--input-file` | 无 | 输入 CSV |
| `--output-dir` | 无 | 输出目录 |
| `--dataset-root` | `../Dataset` | 数据目录 |
| `--model-kind` | `v2-both` | 趋势图使用哪种模型 |
| `--params` | `outputs/reference_fit.json` | `V1` 趋势图用参数文件 |
| `--sample-id` | `None` | 指定样本 ID |
| `--row-index` | `0` | 若不写 `sample_id`，按行号选样本 |
| `--vary` | `pH` | 变化变量：`pH` / `salt_mM` / `concentration_mg_ml` |
| `--start` | `4.5` | 横轴起点 |
| `--end` | `8.0` | 横轴终点 |
| `--points` | `20` | 采样点数 |
| 其余环境参数 | 同 `predict-batch` | 用于其余固定环境条件 |
| `--structure-policy` | `auto` | 结构策略 |
| `--structure-cache-dir` | `structure_cache` | 结构缓存目录 |

输出：

- `<sample_id>_<变量>_trend.csv`
- `<sample_id>_<变量>_trend.png`
- `<sample_id>_<变量>_trend_summary.md`

---

## 9. `plot-heatmap`

用途：

- 针对一个样本，同时扫描两个环境变量
- 输出二维热图、明细矩阵和文字摘要

示例：

```powershell
python -m electrostatic_crowding_viscosity.cli plot-heatmap --input-file standard_input_template_full.csv --output-dir heatmap_outputs --model-kind v2-both --vary-x pH --start-x 5.0 --end-x 8.0 --points-x 25 --vary-y salt_mM --start-y 20 --end-y 150 --points-y 20
```

参数：

| 参数 | 默认值 | 含义 |
| --- | --- | --- |
| `--input-file` | 无 | 输入 CSV |
| `--output-dir` | 无 | 输出目录 |
| `--dataset-root` | `../Dataset` | 数据目录 |
| `--model-kind` | `v2-both` | `v1` / `v2-mlr` / `v2-rf` / `v2-both` |
| `--params` | `outputs/reference_fit.json` | `V1` heatmap 用参数文件 |
| `--sample-id` | `None` | 指定样本 ID |
| `--row-index` | `0` | 若不写 `sample_id`，按行号选样本 |
| `--vary-x` | `pH` | 横轴变量 |
| `--start-x` | `5.0` | 横轴起点 |
| `--end-x` | `8.0` | 横轴终点 |
| `--points-x` | `25` | 横轴采样点数 |
| `--vary-y` | `salt_mM` | 纵轴变量 |
| `--start-y` | `20.0` | 纵轴起点 |
| `--end-y` | `150.0` | 纵轴终点 |
| `--points-y` | `20` | 纵轴采样点数 |
| 其余环境参数 | 同 `predict-batch` | 其余固定环境条件 |
| `--structure-policy` | `auto` | 结构策略 |
| `--structure-cache-dir` | `structure_cache` | 结构缓存目录 |

输出：

- `<sample_id>_<变量X>_vs_<变量Y>_heatmap.csv`
- `<sample_id>_<变量X>_vs_<变量Y>_<预测列>_heatmap.png`
- `<sample_id>_<变量X>_vs_<变量Y>_heatmap_summary.md`

---

## 共享环境参数说明

| 参数 | 含义 |
| --- | --- |
| `cation_species` | 阳离子种类，如 `Na`、`K`、`Mg`、`Ca` |
| `anion_species` | 阴离子种类，如 `Cl`、`SO4`、`SCN`、`Acetate`、`Phosphate` |
| `excipient_name` | 辅料名称，如 `None`、`Arginine`、`Histidine`、`Lysine`、`Sucrose` 等 |
| `excipient_concentration` | 辅料浓度，单位 `mM` |
| `excipient_net_charge_override` | 手工指定辅料净电荷 |
| `excipient_hbond_donor_override` | 手工指定供氢键数 |
| `excipient_hbond_acceptor_override` | 手工指定受氢键数 |
| `excipient_logp_override` | 手工指定疏水性 `logP` |
| `excipient_aromatic_ring_override` | 手工指定芳香环个数 |
| `excipient_is_amino_acid_override` | 手工指定是否按氨基酸处理，常用 `0/1` |

## 结构策略说明

| 参数值 | 含义 |
| --- | --- |
| `off` | 不使用结构特征 |
| `auto` | 如果没有结构文件，则基于序列自动获取结构 |
| `full` | 优先按全长链构造结构代理 |

自动结构获取顺序：

1. 优先使用 `structure_file`
2. 若提供 `structure_id`，尝试从 AlphaFold DB 下载
3. 若仍无结构，则按序列调用 `ESMFold` 获取结构代理

结构缓存与重试机制：

- 本地缓存目录默认：`structure_cache/`
- 缓存索引文件：`structure_cache/structure_cache_index.json`
- 对同一 accession 或同一序列哈希，优先复用已有缓存
- 下载或折叠失败时，会自动重试并将状态写入缓存索引

## 推荐查看

- 项目概览：[README.md](/c:/Users/yiyang.su/project/Vicosity_prediction/electrostatic_crowding_viscosity_model/README.md)
- 精简速查表：[CLI_速查表_中文.md](/c:/Users/yiyang.su/project/Vicosity_prediction/electrostatic_crowding_viscosity_model/CLI_速查表_中文.md)

