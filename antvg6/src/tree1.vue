<template>
  <div id="container1">sssss</div>
</template>

<script setup lang="ts">
import { Rect as GRect, Text as GText, Image as GImage } from "@antv/g";
import {
  Badge,
  CommonEvent,
  ExtensionCategory,
  Graph,
  GraphEvent,
  iconfont,
  Label,
  Rect,
  register,
  treeToGraphData,
  Line,
  BaseEdge,
} from "@antv/g6";

import { datas } from "./tree1-datas";

const style = document.createElement("style");
style.innerHTML = `@import url('${iconfont.css}');`;
document.head.appendChild(style);

const COLORS = {
  B: "#1783FF",
  R: "#F46649",
  Y: "#DB9D0D",
  G: "#60C42D",
  DI: "#A7A7A7",
};
const GREY_COLOR = "#CED4D9";

class TreeNode extends Rect {
  get data() {
    return this.context.model.getNodeLikeDatum(this.id);
  }

  get childrenData() {
    return this.context.model.getChildrenData(this.id);
  }

  getLabelStyle(attributes) {
    const [width, height] = this.getSize(attributes);
    return {
      x: -width / 2 + 8,
      y: -height / 2 + 16,
      text: this.data.name,
      fontSize: 12,
      opacity: 0.85,
      fill: "#000",
      cursor: "pointer",
    };
  }

  // getPriceStyle(attributes) {
  //   const [width, height] = this.getSize(attributes);
  //   return {
  //     x: -width / 2 + 8,
  //     y: height / 2 - 8,
  //     text: this.data.label,
  //     fontSize: 16,
  //     fill: "#000",
  //     opacity: 0.85,
  //   };
  // }

  // drawPriceShape(attributes, container) {
  //   const priceStyle = this.getPriceStyle(attributes);
  //   this.upsert("price", GText, priceStyle, container);
  // }
  getLeaderStyle(attributes) {
    const [width, height] = this.getSize(attributes);
    return {
      x: -width / 2 + 8,
      y: height / 2 - 8,
      text: `负责人：${this.data.leader}`,
      fontSize: 12,
      fill: "#000",
      opacity: 0.85,
    };
  }

  drawLeaderShape(attributes, container) {
    const priceStyle = this.getLeaderStyle(attributes);
    this.upsert("price", GText, priceStyle, container);
  }

  getImageStyle(attributes) {
    const [width, height] = this.getSize(attributes);
    return {
      x: -20,
      y: -20,
      width: 40,
      height: 40,
      src: "https://gw.alipayobjects.com/mdn/rms_6ae20b/afts/img/A*N4ZMS7gHsUIAAAAAAAAAAABkARQnAQ",
      // style: { size: 20 },
    };
  }

  drawImageShape(attributes, container) {
    const imageStyle = this.getImageStyle(attributes);
    this.upsert("image", GImage, imageStyle, container);
  }

  // getCurrencyStyle(attributes) {
  //   const [, height] = this.getSize(attributes);
  //   return {
  //     x: this.shapeMap["price"].getLocalBounds().max[0] + 4,
  //     y: height / 2 - 8,
  //     text: this.data.currency,
  //     fontSize: 12,
  //     fill: "#000",
  //     opacity: 0.75,
  //   };
  // }

  // drawCurrencyShape(attributes, container) {
  //   const currencyStyle = this.getCurrencyStyle(attributes);
  //   this.upsert("currency", GText, currencyStyle, container);
  // }

  // getPercentStyle(attributes) {
  //   const [width, height] = this.getSize(attributes);
  //   return {
  //     x: width / 2 - 4,
  //     y: height / 2 - 8,
  //     text: `${((Number(this.data.variableValue) || 0) * 100).toFixed(2)}%`,
  //     fontSize: 12,
  //     textAlign: "right",
  //     fill: COLORS[this.data.status],
  //   };
  // }

  // drawPercentShape(attributes, container) {
  //   const percentStyle = this.getPercentStyle(attributes);
  //   this.upsert("percent", GText, percentStyle, container);
  // }

  getTriangleStyle(attributes) {
    // const percentMinX = this.shapeMap["percent"].getLocalBounds().min[0];
    // console.log(percentMinX);
    const percentMinX = 55.287109375;
    const [, height] = this.getSize(attributes);
    return {
      fill: COLORS[this.data.status],
      x: this.data.variableUp ? percentMinX - 18 : percentMinX,
      y: height / 2 - 16,
      fontFamily: "iconfont",
      fontSize: 16,
      text: "\ue62d",
      transform: this.data.variableUp ? [] : [["rotate", 180]],
    };
  }

  drawTriangleShape(attributes, container) {
    const triangleStyle = this.getTriangleStyle(attributes);
    this.upsert("triangle", Label, triangleStyle, container);
  }

  // getVariableStyle(attributes) {
  //   const [, height] = this.getSize(attributes);
  //   return {
  //     fill: "#000",
  //     fontSize: 12,
  //     opacity: 0.45,
  //     text: this.data.variableName,
  //     textAlign: "right",
  //     x: this.shapeMap["triangle"].getLocalBounds().min[0] - 4,
  //     y: height / 2 - 8,
  //   };
  // }

  // drawVariableShape(attributes, container) {
  //   const variableStyle = this.getVariableStyle(attributes);
  //   this.upsert("variable", GText, variableStyle, container);
  // }

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
        // console.log(graph);
        // console.log(collapsed);
        // console.log(this.id);
        if (collapsed) graph.expandElement(this.id);
        else graph.collapseElement(this.id);
        graph.fitView();
      });
    }
  }

  getProcessBarStyle(attributes) {
    const { rate, status } = this.data;
    const { radius } = attributes;
    const color = COLORS[status];
    const percent = `${Number(rate) * 100}%`;
    const [width, height] = this.getSize(attributes);
    return {
      x: -width / 2,
      y: height / 2 - 4,
      width: width,
      height: 4,
      radius: [0, 0, radius, radius],
      fill: `linear-gradient(to right, ${color} ${percent}, ${GREY_COLOR} ${percent})`,
    };
  }

  drawProcessBarShape(attributes, container) {
    const processBarStyle = this.getProcessBarStyle(attributes);
    this.upsert("process-bar", GRect, processBarStyle, container);
  }

  getKeyStyle(attributes) {
    const keyStyle = super.getKeyStyle(attributes);
    return {
      ...keyStyle,
      fill: "#fff",
      lineWidth: 1,
      stroke: GREY_COLOR,
    };
  }

  render(attributes = this.parsedAttributes, container) {
    // console.log(this.parsedAttributes);
    if (container.id == "g1") {
      console.log(attributes);
      console.log(container);
    }
    super.render(attributes, container);

    // this.drawPriceShape(attributes, container);
    // this.drawCurrencyShape(attributes, container);
    this.drawLeaderShape(attributes, container);
    this.drawImageShape(attributes, container);

    // this.drawPercentShape(attributes, container);
    this.drawTriangleShape(attributes, container);
    // this.drawVariableShape(attributes, container);
    this.drawProcessBarShape(attributes, container);
    this.drawCollapseShape(attributes, container);
  }
}
class PolylineEdge extends BaseEdge {
  getKeyPath(attributes) {
    const [sourcePoint, targetPoint] = this.getEndpoints(attributes);

    return [
      ["M", sourcePoint[0], sourcePoint[1]],
      ["L", targetPoint[0] / 2 + (1 / 2) * sourcePoint[0], sourcePoint[1]],
      ["L", targetPoint[0] / 2 + (1 / 2) * sourcePoint[0], targetPoint[1]],
      ["L", targetPoint[0], targetPoint[1]],
    ];
  }
}
console.log(ExtensionCategory);
register(ExtensionCategory.NODE, "tree-node", TreeNode);
register(ExtensionCategory.EDGE, "custom-polyline", PolylineEdge);

fetch("https://assets.antv.antgroup.com/g6/decision-tree.json")
  .then((res) => res.json())
  .then((data) => {
    console.log(data);
    const graphData = treeToGraphData(datas.trre1, {
      getNodeData: (datum, depth) => {
        if (!datum.style) datum.style = {};
        // datum.style.collapsed = depth >= 2;
        if (!datum.children) return datum;
        const { children, ...restDatum } = datum;
        // console.log({
        //   ...restDatum,
        //   children: children.map((child) => child.id),
        // });
        return { ...restDatum, children: children.map((child) => child.id) };
      },
    });
    console.log(graphData);
    graphData.edges?.push({
      source: "a1",
      target: "a2",
      // type: "line",
      type: "custom-polyline",
      style: {
        startArrow: true,
        endArrow: true,
        stroke: "#1783FF",
      },
      id: "edge1",
    });
    graphData.nodes?.push({
      id: "a1",
      type: "react",
      name: "a1111111111",
      leader: "xxxxxx",
      count: 123456,
      label: "Circle",
      rate: 0.186,
      status: "G",
      currency: "Yuan",
      variableName: "V2",
      variableValue: 0.531,
      variableUp: true,
      style: {
        opacity: 0,
        fill: "transparent",
        stroke: "transparent",
      },
      children: [],
      x: 100,
      y: 0,
    });
    graphData.nodes?.push({
      id: "a2",
      name: "a2222222222",
      leader: "xxxxxx",
      count: 123456,
      label: "Circle",
      rate: 0.186,
      status: "G",
      currency: "Yuan",
      variableName: "V2",
      variableValue: 0.531,
      variableUp: true,
      style: {},
      children: [],
      x: 250,
      y: 350,
    });
    console.log(graphData);
    const graph = new Graph({
      container: "container1",
      width: 1892, // 图的宽度
      height: 1000, // 图的高度
      data: datas.aimTreeData,
      // data: treeToGraphData(datas.trre1, {
      //   getNodeData: (datum, depth) => {
      //     if (!datum.style) datum.style = {};
      //     // datum.style.collapsed = depth >= 2;
      //     if (!datum.children) return datum;
      //     const { children, ...restDatum } = datum;
      //     // console.log({
      //     //   ...restDatum,
      //     //   children: children.map((child) => child.id),
      //     // });
      //     return { ...restDatum, children: children.map((child) => child.id) };
      //   },
      // }),
      node: {
        // type: "tree-node",
        // type: "tree-node",
        style: {
          size: [202, 100],
          // ports: [{ placement: "left" }, { placement: "right" }],
          // ports: [{ placement: "top" }, { placement: "bottom" }],
          ports: [
            { placement: "top" },
            { placement: "bottom" },
            // { placement: "left" },
            // { placement: "right" },
          ],
          radius: 4,
        },
      },
      edge: {
        // type: "cubic-horizontal",
        // style: {
        //   stroke: GREY_COLOR,
        // },
        // type: "quadratic",
        // type: "polyline",
        // style: {
        //   router: {
        //     type: "orth",
        //   },
        //   // stroke: "#C0C0C0",
        //   stroke: GREY_COLOR,
        // },
      },
      // edge: [
      //   {
      //     // type: "cubic-horizontal",
      //     // style: {
      //     //   stroke: GREY_COLOR,
      //     // },

      //     type: "quadratic",
      //     // type: "polyline",
      //     style: {
      //       router: {
      //         type: "orth",
      //       },
      //       // stroke: "#C0C0C0",
      //       stroke: GREY_COLOR,
      //     },
      //   },
      //   // {
      //   //   type: "extra-label-edge",
      //   //   style: {
      //   //     startArrow: true,
      //   //     endArrow: true,
      //   //     stroke: "#F6BD16",
      //   //     startLabelText: "start",
      //   //     endLabelText: "end",
      //   //   },
      //   // },
      // ],
      layout: {
        type: "dagre",
        // direction: "TB", // 从上到下布局
        nodeSep: 200, // 节点之间的垂直间距
        rankSep: 100,

        // type: "indented",
        // direction: "TB",
        // // direction: "LR",
        // dropCap: false,
        // indent: 300,
        // getHeight: () => 160,
      },
      autoFit: "view",
      // modes: {
      //   default: [{
      //     type: 'collapse-expand',
      //     onChange: function onChange(item, collapsed) {
      //       const data = item.get('model');
      //       const icon = item.get('group').findByClassName('collapse-icon');
      //       if (collapsed) {
      //         icon.attr('symbol', EXPAND_ICON);
      //       } else {
      //         icon.attr('symbol', COLLAPSE_ICON);
      //       }
      //       data.collapsed = collapsed;
      //       return true;
      //     }
      //   }, 'drag-canvas', 'zoom-canvas' ]
      // },
      behaviors: ["level-of-detail", "zoom-canvas", "drag-canvas"],
    });

    graph.once(GraphEvent.AFTER_RENDER, () => {
      console.log(GraphEvent);
      // 适配容器大小
      graph.fitView();
    });
    // // 手动设置根节点的位置
    // const rootNode = graph.findNodeById('g1');
    // if (rootNode) {
    //   rootNode.setModel({
    //     x: fixedX,
    //     y: fixedY,
    //   });
    // }
    // console.log(graph);

    graph.render();

    graph.updateNodeData([{ id: "a1", style: { x: 2000, y: 1000 } }]);
  });
</script>

<style scoped></style>
