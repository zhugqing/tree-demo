<script setup lang="ts">
// import Tree from "./tree-huo.vue";
import {
  Badge,
  CommonEvent,
  BaseBehavior,
  ExtensionCategory,
  Graph,
  GraphEvent,
  Label,
  Rect,
  register,
} from "@antv/g6";

// import datas from "./datas";

const statusColors = {
  online: "#17BEBB",
  busy: "#E36397",
  offline: "#B7AD99",
};

const DEFAULT_LEVEL = "detailed";
const GREY_COLOR = "#CED4D9";

/**
 * Draw a chart node with different ui based on the zoom level.
 */
class ChartNode extends Rect {
  get data() {
    return this.context.model.getElementDataById(this.id).data;
  }

  get childrenData() {
    return this.context.model.getChildrenData(this.id);
  }

  get level() {
    return this.data.level || DEFAULT_LEVEL;
  }

  getLabelStyle() {
    const text = this.data.name;
    const labelStyle =
      this.level === "overview"
        ? {
            fill: "#fff",
            fontSize: 20,
            fontWeight: 600,
            textAlign: "center",
            transform: [["translate", 0, 0]],
          }
        : {
            fill: "#2078B4",
            fontSize: 14,
            fontWeight: 400,
            textAlign: "left",
            transform: [["translate", -65, -15]],
          };
    return { text, ...labelStyle };
  }

  getKeyStyle(attributes) {
    return {
      ...super.getKeyStyle(attributes),
      fill: this.level === "overview" ? statusColors[this.data.status] : "#fff",
    };
  }

  getPositionStyle(attributes) {
    if (this.level === "overview") return false;
    return {
      text: this.data.position,
      fontSize: 8,
      fontWeight: 400,
      textTransform: "uppercase",
      fill: "#343f4a",
      textAlign: "left",
      transform: [["translate", -65, 0]],
    };
  }

  drawPositionShape(attributes, container) {
    const positionStyle = this.getPositionStyle(attributes);
    this.upsert("position", Label, positionStyle, container);
  }

  getStatusStyle(attributes) {
    if (this.level === "overview") return false;
    return {
      text: this.data.status,
      fontSize: 8,
      textAlign: "left",
      transform: [["translate", 40, -16]],
      padding: [0, 4],
      fill: "#fff",
      backgroundFill: statusColors[this.data.status],
    };
  }

  drawStatusShape(attributes, container) {
    const statusStyle = this.getStatusStyle(attributes);
    this.upsert("status", Badge, statusStyle, container);
  }

  getPhoneStyle(attributes) {
    if (this.level === "overview") return false;
    return {
      text: this.data.phone,
      fontSize: 8,
      fontWeight: 300,
      textAlign: "left",
      transform: [["translate", -65, 20]],
    };
  }

  drawPhoneShape(attributes, container) {
    const style = this.getPhoneStyle(attributes);
    this.upsert("phone", Label, style, container);
  }

  getCollapseStyle(attributes) {
    if (this.childrenData.length === 0) return false;
    const { collapsed } = attributes;
    const [width, height] = this.getSize(attributes);
    return {
      backgroundFill: "#fff",
      backgroundHeight: 16,
      backgroundLineWidth: 1,
      backgroundRadius: 0,
      backgroundStroke: GREY_COLOR,
      backgroundWidth: 16,
      cursor: "pointer",
      fill: GREY_COLOR,
      fontSize: 16,
      text: collapsed ? "+" : "-",
      textAlign: "center",
      textBaseline: "middle",
      // x: width / 2,
      // y: 0,
      x: 0,
      y: height / 2,
    };
  }

  drawCollapseShape(attributes, container) {
    const collapseStyle = this.getCollapseStyle(attributes);
    const btn = this.upsert("collapse", Badge, collapseStyle, container);
    // console.log(collapseStyle);
    // console.log(btn);

    if (btn && !Reflect.has(btn, "__bind__")) {
      Reflect.set(btn, "__bind__", true);
      btn.addEventListener(CommonEvent.CLICK, () => {
        console.log("点击");
        const { collapsed } = this.attributes;
        const graph = this.context.graph;
        console.log(graph);
        console.log(collapsed);
        console.log(this.id);
        if (collapsed) graph.expandElement(this.id);
        else graph.collapseElement(this.id);
      });
    }
  }

  render(attributes = this.parsedAttributes, container = this) {
    super.render(attributes, container);

    this.drawPositionShape(attributes, container);

    this.drawStatusShape(attributes, container);

    this.drawPhoneShape(attributes, container);
    this.drawCollapseShape(attributes, container);
  }
}

/**
 * Implement a level of detail rendering, which will show different details based on the zoom level.
 */
class LevelOfDetail extends BaseBehavior {
  prevLevel = DEFAULT_LEVEL;
  levels = {
    ["overview"]: [0, 0.6],
    ["detailed"]: [0.6, Infinity],
  };

  constructor(context, options) {
    super(context, options);
    this.bindEvents();
  }

  update(options) {
    this.unbindEvents();
    super.update(options);
    this.bindEvents();
  }

  updateZoomLevel = async (e) => {
    if ("scale" in e.data) {
      const scale = e.data.scale;
      const level = Object.entries(this.levels).find(
        ([key, [min, max]]) => scale > min && scale <= max
      )?.[0];
      if (level && this.prevLevel !== level) {
        const { graph } = this.context;
        graph.updateNodeData((prev) =>
          prev.map((node) => ({ ...node, data: { ...node.data, level } }))
        );
        await graph.draw();
        this.prevLevel = level;
      }
    }
  };

  bindEvents() {
    const { graph } = this.context;
    graph.on(GraphEvent.AFTER_TRANSFORM, this.updateZoomLevel);
  }

  unbindEvents() {
    const { graph } = this.context;
    graph.off(GraphEvent.AFTER_TRANSFORM, this.updateZoomLevel);
  }

  destroy() {
    this.unbindEvents();
    super.destroy();
  }
}
console.log(ExtensionCategory.NODE);
console.log(ExtensionCategory.BEHAVIOR);
register(ExtensionCategory.NODE, "chart-node", ChartNode);
register(ExtensionCategory.BEHAVIOR, "level-of-detail", LevelOfDetail);

fetch("https://assets.antv.antgroup.com/g6/organization-chart.json")
  .then((res) => res.json())
  .then((data) => {
    console.log(data);
    const graph = new Graph({
      container: "container",
      width: 1500, // 图的宽度
      height: 1000, // 图的高度
      data,
      node: {
        type: "chart-node",
        style: {
          labelPlacement: "center",
          lineWidth: 1,
          ports: [{ placement: "top" }, { placement: "bottom" }],
          radius: 2,
          shadowBlur: 10,
          shadowColor: "#e0e0e0",
          shadowOffsetX: 3,
          size: [150, 60],
          stroke: "#C0C0C0",
        },
      },
      edge: {
        type: "polyline",
        style: {
          router: {
            type: "orth",
          },
          stroke: "#C0C0C0",
        },
      },
      layout: {
        type: "dagre",
      },
      autoFit: "view",
      behaviors: ["level-of-detail", "zoom-canvas", "drag-canvas"],
    });

    graph.render();
  });
</script>

<template>
  <!-- <Tree /> -->
  <div id="container">sssss</div>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
