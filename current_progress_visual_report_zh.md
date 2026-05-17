# SCF-FMF 模式合成器当前进度可视化报告

**日期：** 2026-05-17  
**当前状态：** scalar-FD 设计与容差通过；Lumerical MODE FDE vector import 通过；high-count Lumerical vector transfer 已恢复 MMI 增益并通过 Gate F。  
**核心结论：** 之前 16 个 SCF modes 下 Lumerical vector transfer 不出现明显 MMI 增益，主要是 SCF 模式截断造成的假阴性；提升到 `SCF=64, FMF=12` 后，最佳点回到 `L≈1.6 mm`，与 scalar-FD refined candidate 定量接近。

---

## 1. 一页摘要

| 项目 | 当前结果 |
|---|---:|
| 主候选 | `N=8 four-corner` |
| 输入 offset | `14 um` |
| SMF waist / proxy radius | `5.5 um` waist / `4.1 um` radius |
| SCF square core | `37 um` |
| FMF radius | `12 um` |
| 目标模式 | FMF `LP11` group |
| Lumerical 标准数据集 | `final64`: SCF 64 modes, FMF 12 modes |
| 最佳 Lumerical 长度 | `1.600 mm` |
| Lumerical eta | `0.855701` |
| Lumerical delta_eta_vs_L0 | `0.731276` |
| Lumerical rho_LP11_group | `1.000` |
| Gate F basis/import | `True` |
| Gate F MMI/design | `True` |

![项目阶段路线图](../figures/report_project_progress_pipeline.png)

---

## 2. 物理模型和输入布局

系统仍按线性 transfer matrix 建模：

```text
c = T(L) a = O D(L) B a
```

其中 `B` 是 SMF 输入到 SCF 模式，`D(L)` 是 SCF 多模传播，`O` 是 SCF 输出到 FMF 模式投影。当前成功验证的是 high-count Lumerical FDE 模式导入后的 `T=ODB` 趋势；还不是 EME taper/splice 或实验验证。

![N=8 four-corner SMF 输入布局](../figures/report_smf_four_corner_code_coordinates.png)

注：这里的 `four-corner` 使用代码中的 `layout_positions(8, "four-corner", 14e-6)` 定义。8 个输入中心不是每个角两个紧邻通道，而是 4 个角点加 4 个边中点，坐标为 `(+/-14,+/-14) um`、`(+/-14,0) um`、`(0,+/-14) um`。

---

## 3. Scalar-FD 与 Lumerical high-count 对比

| 指标 | Scalar-FD refined tolerance | Lumerical final64 vector transfer |
|---|---:|---:|
| eta / eta_p05 | `0.8835` | `0.855701` |
| delta_eta | `0.7799` | `0.731276` |
| rho | `0.9865 p05` | `1.000` |
| ER | `18.64 dB p05` | `299.32 dB retained-basis` |
| 最佳长度 | `1.6 mm` | `1.600 mm` |

![Scalar-FD 与 Lumerical final64 关键指标对比](../figures/report_gate_f_metric_comparison.png)

![Lumerical final64 长度扫描](../figures/lumerical_vector_length_scan.png)

---

## 4. Lumerical imported modes 与 LP11 标签

Lumerical MODE FDE 已导出 SMF/SCF/FMF 的 `Ex,Ey,Ez,Hx,Hy,Hz`，内部使用 full-vector reciprocal power overlap，并用 Gram-aware dual-basis 处理非严格正交 retained basis。FMF LP11 简并模式不强行固定为单一 `LP11x/LP11y`，而是作为 `LP11_a..LP11_d` group 处理。

![Lumerical FDE 模式场预览](../figures/report_lumerical_mode_fields.png)

| FMF mode | 原始标签 | 分配标签 | group | projection | orientation ambiguous |
|---:|---|---|---|---:|---|
| 1 | mode1 | LP01_a | LP01 | 0.988 | False |
| 2 | mode2 | LP01_b | LP01 | 0.988 | False |
| 3 | mode3 | LP11_a | LP11 | 0.926 | True |
| 4 | mode4 | LP11_b | LP11 | 0.926 | True |
| 5 | mode5 | LP11_c | LP11 | 0.928 | True |
| 6 | mode6 | LP11_d | LP11 | 0.928 | True |
| 7 | mode7 | LP21_a | LP21 | 0.802 | False |
| 8 | mode8 | LP21_b | LP21 | 0.806 | False |

![SCF vector Gram](../figures/lumerical_scf_vector_gram.png)

![FMF vector Gram](../figures/lumerical_fmf_vector_gram.png)

---

## 5. G1 mismatch resolution：为什么 16 个 SCF modes 不够

G1 诊断显示，16-SCF-mode baseline 下 SMF-FDE 输入和受限控制的 `delta_eta_vs_L0` 最大只有约 `0.1403`，低于 MMI gate 的 `0.2`。提升 SCF retained modes 后，`SCF=24/32/48/64` 与 `FMF=8/12` 均恢复 `gate_f_mmi_pass=True`。因此 dominant mismatch source 是 SCF 模式截断，而不是原 scalar-FD 设计失效。

![Mode-count Gate F 汇总](../figures/report_mode_count_gate_summary.png)

| SCF modes | FMF modes | eta | delta_eta | basis pass | MMI pass |
|---:|---:|---:|---:|---|---|
| 24 | 8 | 0.7226 | 0.5532 | True | True |
| 24 | 12 | 0.7226 | 0.5532 | True | True |
| 24 | 16 | 0.7317 | 0.3791 | False | False |
| 32 | 8 | 0.8169 | 0.7059 | True | True |
| 32 | 12 | 0.8169 | 0.7059 | True | True |
| 32 | 16 | 0.8183 | 0.6005 | False | False |
| 48 | 8 | 0.8492 | 0.7221 | True | True |
| 48 | 12 | 0.8492 | 0.7221 | True | True |
| 48 | 16 | 0.8715 | 0.5844 | False | False |
| 64 | 8 | 0.8557 | 0.7313 | True | True |
| 64 | 12 | 0.8557 | 0.7313 | True | True |
| 64 | 16 | 0.9143 | 0.6743 | False | False |

![G1 MMI resolution summary](../figures/lumerical_vector_mmi_resolution_summary.png)

![B/O singular spectrum](../figures/lumerical_vector_mmi_resolution_bo_spectrum.png)

---

## 6. 当前完成项、未完成项与下一步

| 阶段 | 状态 | 证据/备注 |
|---|---|---|
| Scalar MVP | 已完成 | `mvp_report.md`，基本 `T=ODB` pipeline 完成 |
| Scalar-FD design optimization | 已完成 | `fd_design_optimization_report.md`，发现 robust MMI candidate |
| Top-candidate tolerance | 已完成 | `SCF=37 um, L=1.6 mm`, `eta_p05≈0.8835` |
| Lumerical FDE vector import | 已完成 | SMF/SCF/FMF E/H raw NPZ 与 vector dataset validation |
| Lumerical high-count transfer | 已完成 | `final64` 报告中 `gate_f_basis_pass=True`, `gate_f_mmi_pass=True` |
| Lumerical EME interface/taper/splice | 未完成 | 只存在 gated scaffold；还没有真实 S-matrix/O_interface |
| COMSOL cross-check | 未完成 | 建议在 EME 初步通过后做独立 FEM 验证 |
| 实验设计 | 未完成 | 需要 modal metrics，不应只看近场 camera 图样 |

### 下一步建议

1. 进入 **Lumerical EME Interface Validation**，但只做真实 EME，不再用 synthetic rows 当物理证据。
2. 首先验证 ideal butt coupling，然后做 `dx/dy offset`、短 taper、square-to-circular transition。
3. 输出必须仍回到 modal metrics：`LP11 group coupling`、`LP01 leakage`、reflection、radiation/loss、S-matrix 或 `O_interface`。
4. FMF=16 暂不作为标准配置，先单独诊断 basis gate failure。
5. EME 初步通过后，再用 COMSOL Wave Optics 做 FEM cross-check。

---

## 7. 主要文件索引

| 类型 | 路径 |
|---|---|
| 当前中文可视化报告 | `results/reports/current_progress_visual_report_zh.md` |
| Lumerical final64 transfer report | `results/reports/lumerical_vector_transfer_report.md` |
| G1 mismatch resolution report | `results/reports/lumerical_vector_mmi_resolution_report.md` |
| top candidate tolerance report | `results/reports/top_candidate_tolerance_report.md` |
| final64 transfer CSV | `results/sweeps/lumerical_vector_transfer.csv` |
| mode-count sweep CSV | `results/sweeps/lumerical_vector_mmi_resolution_mode_count.csv` |
| Lumerical final64 raw modes | `results/lumerical/mode_count_sweeps/scf64_fmf12/` |
