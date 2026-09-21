# CLI 速查表（中文）

## 最常用命令

### 1. 拟合 `V1`

```powershell
python -m electrostatic_crowding_viscosity.cli fit-pdgf38 --output-dir outputs
```

### 2. 用 `V1` 监测 `GDPa1`

```powershell
python -m electrostatic_crowding_viscosity.cli monitor-gdpa1 --params outputs/reference_fit.json --output-dir outputs
```

### 3. 用 `V1` 在指定离子/辅料环境下监测

```powershell
python -m electrostatic_crowding_viscosity.cli monitor-gdpa1 --params outputs/reference_fit.json --output-dir outputs_envtest --default-ph 6.0 --default-salt 50 --default-concentration 150 --cation-species Na --anion-species Cl --excipient-name Arginine --excipient-concentration 50
```

### 4. 运行 `V2` 训练与交叉验证

```powershell
python -m electrostatic_crowding_viscosity.cli run-v2 --output-dir outputs_v2
```

### 5. 运行 `V2` 并指定环境

```powershell
python -m electrostatic_crowding_viscosity.cli run-v2 --output-dir outputs_v2_arg50 --cation-species Na --anion-species Cl --excipient-name Arginine --excipient-concentration 50
```

### 6. 批量预测

```powershell
python -m electrostatic_crowding_viscosity.cli predict-batch --input-file standard_input_template_full.csv --output-file predictions.csv --model-kind v2-both
```

### 7. 运行浓度分段流变模型训练

```powershell
python -m electrostatic_crowding_viscosity.cli run-regime --output-dir outputs_regime
```

### 8. 自动结构模式批量预测

```powershell
python -m electrostatic_crowding_viscosity.cli predict-batch --input-file standard_input_template_full.csv --output-file predictions_struct.csv --model-kind v2-mlr --structure-policy auto
```

### 9. 浓度分段流变模型批量预测

```powershell
python -m electrostatic_crowding_viscosity.cli predict-regime-batch --input-file standard_input_template_full.csv --output-file predictions_regime.csv --params outputs_regime/regime_parameters.json
```

### 10. 生成趋势图

```powershell
python -m electrostatic_crowding_viscosity.cli plot-trend --input-file standard_input_template_full.csv --output-dir trend_outputs --model-kind v2-both --vary pH --start 5.0 --end 8.0 --points 20
```

### 11. 生成双变量 heatmap

```powershell
python -m electrostatic_crowding_viscosity.cli plot-heatmap --input-file standard_input_template_full.csv --output-dir heatmap_outputs --model-kind v2-both --vary-x pH --start-x 5.0 --end-x 8.0 --points-x 25 --vary-y salt_mM --start-y 20 --end-y 150 --points-y 20
```

## 模型类型

- `v1`：静电-拥挤半机理模型
- `v2-mlr`：V2 多元线性回归
- `v2-rf`：V2 随机森林
- `v2-both`：同时输出 `MLR` 和 `RF`
- `run-regime`：浓度分段流变模型，预测 `η(c, T, pH, I, shear_rate)`

## 最常用参数

| 参数 | 含义 |
| --- | --- |
| `--dataset-root` | 数据目录 |
| `--output-dir` | 输出目录 |
| `--params` | `V1` 参数文件 |
| `--reference-ph` | 默认/参考 `pH` |
| `--reference-salt` | 默认/参考盐浓度 |
| `--reference-concentration` | 默认/参考蛋白浓度 |
| `--reference-temperature-c` | `run-regime` 参考温度，单位 Celsius |
| `--reference-ionic-strength` | `run-regime` 参考离子强度，单位 mM |
| `--reference-shear-rate` | `run-regime` 参考剪切速率，单位 1/s |
| `--ridge-alpha` | `run-regime` 残差回归正则强度 |
| `--cation-species` | 阳离子种类 |
| `--anion-species` | 阴离子种类 |
| `--excipient-name` | 辅料名称 |
| `--excipient-concentration` | 辅料浓度 |
| `--structure-policy` | 结构策略：`off` / `auto` / `full` |
| `--max-features` | `V2-MLR` 最大特征数 |
| `--model-kind` | 选择输出哪种模型 |

## 结构策略说明

- `off`：不使用结构特征
- `auto`：若无结构文件，自动基于序列获取结构
- `full`：优先按全长链构建结构代理

自动结构获取会使用：

- 本地缓存目录：`structure_cache`
- 缓存索引文件：`structure_cache_index.json`
- 自动失败重试

## 输入模板文件

- [standard_input_template_minimal.csv](/c:/Users/yiyang.su/project/Vicosity_prediction/electrostatic_crowding_viscosity_model/standard_input_template_minimal.csv)
- [standard_input_template_full.csv](/c:/Users/yiyang.su/project/Vicosity_prediction/electrostatic_crowding_viscosity_model/standard_input_template_full.csv)
- [standard_input_template.csv](/c:/Users/yiyang.su/project/Vicosity_prediction/electrostatic_crowding_viscosity_model/standard_input_template.csv)

## 想看完整参数说明

查看：

- [CLI_调用说明_中文.md](/c:/Users/yiyang.su/project/Vicosity_prediction/electrostatic_crowding_viscosity_model/CLI_调用说明_中文.md)

