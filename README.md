# rnaseq-plot-mcp

RNA-seq **下游出图 / 分析** MCP（旧称 rgraph）。把一套参数化 R 出图脚本
（ggplot2 / pheatmap / clusterProfiler / edgeR / limma / WGCNA …）
封装为 MCP 工具，由本机 **Rscript** 渲染 **png + pdf**。

吃用户自己的 count / FPKM 表，不捆绑任何物种基因组或表达矩阵。

## 为什么是「R 引擎 + Python MCP」

- **忠实**：不在 Python 里重画，直接驱动 R。
- **参数化**：无 `setwd()` 硬编码，输入/输出/阈值/配色走参数。
- **优雅降级**：找不到 Rscript → 返回可手动运行的 `.R`；缺包 → 返回包名与安装命令。
- **可复现**：`--no-init-file --no-site-file`，屏蔽用户 `.Rprofile` 横幅。

## R 引擎

优先级：环境变量 `RGRAPH_RSCRIPT` → PATH 上的 `Rscript` → `C:\Program Files\R\...` / 注册表。

```powershell
$env:RGRAPH_RSCRIPT = "C:\Program Files\R\R-4.5.1\bin\Rscript.exe"
uv run rgraph-cli
```

## 数据格式

| 文件 | 必需列 |
| --- | --- |
| `sample_group.csv` | `sample_name`, `group`/`group_name`, `TvsC`(treatment/control) |
| `gene_count.csv` | `gene_id`, `Length`, 各样本 count |
| `gene_fpkm.csv` / `gene_tpm.csv` | `gene_id`, 各样本列 |
| 差异结果 | `gene_id`, `log2FoldChange`, `pvalue`, `padj` |

差异基因判定默认用 **padj（FDR）**。

## 工具（34）

**核心**：`rgraph_env` `rgraph_normalize` `rgraph_correlation` `rgraph_pca` `rgraph_distribution` `rgraph_diff` `rgraph_volcano` `rgraph_heatmap` `rgraph_enrich` `rgraph_go_plot` `rgraph_kegg_plot` `rgraph_ppi` `rgraph_gsea`

**高级**：韦恩/象限/堆叠火山、WGCNA、网络图、ssGSEA、圈图、桑基、交互火山等。缺 `ComplexHeatmap` 时热图回退 pheatmap；缺 `DESeq2` 时可用 edgeR/limma。

WGCNA → 网络：`rgraph_wgcna` 导出 `Cytoscape_edges_*.txt` 后交给 `rgraph_network(mode="edge")`。

## 注册

```json
{
  "mcpServers": {
    "r-analysis": {
      "command": "uv",
      "args": ["run", "--directory", "/absolute/path/to/rnaseq-plot-mcp", "server.py"],
      "env": { "RGRAPH_RSCRIPT": "/path/to/Rscript" }
    }
  }
}
```

合成测试矩阵在 `tests/data/`（`g1`–`g10`，不含真实实验数据）。

MCP 工具名仍是 `rgraph_*`（如 `rgraph_volcano`），与本机 Cursor 配置兼容。

## 许可

MIT。
