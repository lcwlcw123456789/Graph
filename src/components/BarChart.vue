<template>
  <div @click="toggleChart" class="chart-container">
    <div v-if="isCollapsed" class="collapsed-icon">
      📊 <!-- 收起时显示的小图标 -->
    </div>
    <svg v-else ref="chart" :width="width" :height="height"></svg>
  </div>
</template>

<script>
import * as d3 from "d3";
import * as XLSX from "xlsx"; // 导入 xlsx 库

export default {
  name: "BarChart",
  props: {
    width: {
      type: Number,
      default: 600,
    },
    height: {
      type: Number,
      default: 400,
    },
  },
  data() {
    return {
      isCollapsed: false, // 图表状态：展开或收起
      data: [], // 第二行数据
      labels: [], // 第一行标签
    };
  },
  mounted() {
    this.loadDataFromExcel(); // 从 Excel 文件加载数据
    this.createTooltip();
  },
  methods: {
    createTooltip() {
      // 动态创建 tooltip 元素
      this.tooltip = d3
        .select("body")
        .append("div")
        .attr("class", "tooltip")
        .style("position", "absolute")
        .style("background-color", "black")
        .style("border", "1px solid #ccc")
        .style("border-radius", "4px")
        .style("padding", "5px 10px")
        .style("font-size", "12px")
        .style("pointer-events", "none")
        .style("opacity", 0); // 默认隐藏
    },
    toggleChart() {
      this.isCollapsed = !this.isCollapsed;

      // 如果展开，重新绘制图表
      if (!this.isCollapsed) {
        this.$nextTick(() => this.drawChart());
      }
    },
    loadDataFromExcel() {
      const fileName = "./data.xlsx"; // Excel 文件路径
      fetch(fileName)
        .then((response) => response.arrayBuffer())
        .then((buffer) => {
          const workbook = XLSX.read(buffer, { type: "array" });
          const sheetName = workbook.SheetNames[0];
          const sheet = workbook.Sheets[sheetName];
          const jsonData = XLSX.utils.sheet_to_json(sheet, { header: 1 }); // 转为二维数组

          // 设置 x轴标签（第一行）和 数据（第二行）
          this.labels = jsonData[0];
          this.data = jsonData[1];

          // 确保数据格式一致
          if (this.labels.length !== this.data.length) {
            console.error("Error: Labels and data length do not match.");
            return;
          }

          // 绘制图表
          this.drawChart();
        })
        .catch((error) => {
          console.error("Error loading Excel file:", error);
        });
    },
    drawChart() {
      const svg = d3.select(this.$refs.chart);
      svg.selectAll("*").remove(); // 清除之前的内容

      const margin = { top: 20, right: 30, bottom: 40, left: 40 };
      const chartWidth = this.width - margin.left - margin.right;
      const chartHeight = this.height - margin.top - margin.bottom;

      const x = d3
        .scaleBand()
        .domain(this.labels)
        .range([0, chartWidth])
        .padding(0.1);

      const y = d3
        .scaleLinear()
        .domain([0, d3.max(this.data)])
        .nice()
        .range([chartHeight, 0]);

      const g = svg
        .append("g")
        .attr("transform", `translate(${margin.left},${margin.top})`);

      // X Axis
      g.append("g")
        .attr("transform", `translate(0,${chartHeight})`)
        .call(d3.axisBottom(x));

      // Y Axis
      g.append("g").call(d3.axisLeft(y));

      // Bars
      g.selectAll(".bar")
        .data(this.data)
        .join("rect")
        .attr("class", "bar")
        .attr("x", (_, i) => x(this.labels[i]))
        .attr("y", (d) => y(d))
        .attr("width", x.bandwidth())
        .attr("height", (d) => chartHeight - y(d))
        .attr("fill", "steelblue")
        .on("mouseover", (event, d) => {
          this.tooltip
            .style("opacity", 1)
            .html(`Value: ${d}`)
            .style("left", `${event.pageX + 10}px`)
            .style("top", `${event.pageY - 20}px`);
        })
        .on("mousemove", (event) => {
          this.tooltip
            .style("left", `${event.pageX + 10}px`)
            .style("top", `${event.pageY - 20}px`);
        })
        .on("mouseout", () => {
          this.tooltip.style("opacity", 0);
        });
    },
  },
};
</script>

<style>
.chart-container {
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
}

.collapsed-icon {
  font-size: 64px; /* 图标大小 */
}

.bar:hover {
  fill: orange;
}

.tooltip {
  transition: opacity 0.2s;
}
</style>
