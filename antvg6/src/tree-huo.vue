<template>
  <div class="container">
    <div :id="container" ref="tree" class="tree"></div>
  </div>
</template>

<script>
// import ResizeObserver from 'resize-observer-polyfill';
// import { debounce } from 'lodash';
import elementResizeDetectorMaker from "element-resize-detector";
import G6 from "@antv/g6";
import insertCss from "insert-css";
// import loadingSvg from "@/assets/images/health/faultTreeIcon.png";
import datas from "./datas";

import { isObject } from "@antv/util";
const colorMap = {
  A: "#72CC4A",
  B: "#1A91FF",
  C: "#FFAA15",
};
export default {
  name: "tree",
  props: {
    container: {
      type: String,
      default: "tree",
    },
    treeData: {
      type: Object,
      default: datas.alarmTreeDatas,
    },
  },
  data() {
    return {
      width: 0,
      height: 0,
      tree: "",
    };
  },
  mounted() {
    this.$nextTick(() => {
      if (Object.keys(this.treeData).length > 0) {
        this.showChart();
        this.$forceUpdate();
      }
    });
  },
  watch: {
    treeData: {
      handler(val, old) {
        if (val !== old && Object.keys(val).length > 0) {
          this.refresh();
        }
      },
      deep: true,
    },
  },
  methods: {
    refresh() {
      if (this.tree) {
        this.tree.destroy();
      }
      this.showChart();
      this.$forceUpdate();
    },
    showChart() {
      const that = this;
      //提示框样式
      insertCss(`
            .g6-tooltip {
                position: relative;
                border: 1px solid rgba(26,90,170,1);
                color:#fff;
                font-size: 14px;
                background-color: rgba(2,25,61,0.95);
                padding: 10px 20px 10px 20px;
                box-shadow: rgba(26,90,170,0.3) 0px 0px 10px 3px;
                border-radius: 10px;

            }
            .g6-tooltip .card{
                display: flex;
                flex-direction: column;
                width:500px
            }
            .g6-tooltip .card>div{
                display: flex;
                padding: 4px 0;
            }
            .g6-tooltip .card-item>div:first-child{
                width:80px;
            }
            .g6-tooltip .card-item>div:last-child{
                flex:1;
                white-space:normal;
                word-wrap:break-word;
            }
            `);
      //超长文本处理
      const fittingString = (str, maxWidth, fontSize) => {
        const ellipsis = "...";
        const ellipsisLength = G6.Util.getTextSize(ellipsis, fontSize)[0];
        let currentWidth = 0;
        let res = str;
        const pattern = new RegExp("[\u4E00-\u9FA5]+"); // distinguish the Chinese charactors and letters
        str.split("").forEach((letter, i) => {
          if (currentWidth > maxWidth - ellipsisLength) return;
          if (pattern.test(letter)) {
            // Chinese charactors
            currentWidth += fontSize;
          } else {
            // get the width of single letter according to the fontSize
            currentWidth += G6.Util.getLetterWidth(letter, fontSize);
          }
          if (currentWidth > maxWidth - ellipsisLength) {
            res = `${str.substr(0, i)}${ellipsis}`;
          }
        });
        return res;
      };

      //父节点
      G6.registerNode("root", {
        draw: (cfg, group) => {
          const size = [132, 90];
          const hasChildren = cfg.children && cfg.children.length !== 0;
          const keyShape = group.addShape("rect", {
            attrs: {
              width: size[0],
              height: size[1],
              x: -size[0] / 2,
              y: -size[1] / 2,
              // fill: 'rgb(1, 137, 147)',
              radius: 5,
            },
            draggable: true,
            name: "root-keyshape",
          });
          group.addShape("text", {
            attrs: {
              text: fittingString(cfg.nodeName, 150, 14),
              fill: "rgba(255, 255, 255, 1)",
              fontSize: 14,
              y: -size[1] / 2 + 26 + 50,
              textAlign: "center",
            },
            draggable: true,
            name: "sublabel-shape",
          });
          group.addShape("image", {
            attrs: {
              width: size[0],
              height: size[1],
              x: -size[0] / 2,
              y: -size[1] / 2 - 44,
              //   img: loadingSvg, // 本地图片 URL
            },
            name: "node-background-image",
          });
          return keyShape;
        },
      });

      //level1 node
      G6.registerNode(
        "sub",
        {
          draw: (cfg, group) => {
            const size = [200, 30];
            const keyShape = group.addShape("rect", {
              attrs: {
                width: size[0],
                height: size[1],
                x: -size[0] / 2,
                y: -size[1] / 2,
                fill: "l(0) 1:rgba(126, 72, 255, 0.05) 0:rgba(72, 170, 255, 0.5)",
                stroke: "#48AAFF", // 边框颜色
                lineWidth: 1, // 边框宽度
                radius: 5,
              },
              draggable: true,
              name: "sub-keyShape",
            });
            group.addShape("rect", {
              attrs: {
                x: -size[0] / 2 + 5,
                y: -size[1] / 2 + 3,
                width: 38,
                height: 24,
                radius: 8,
                fill: cfg.increase,
                stroke: cfg.borderColor, // 边框颜色
                lineWidth: 1, // 边框宽度
              },
              draggable: true,
              name: "ratio-box",
            });
            group.addShape("text", {
              attrs: {
                text: `${cfg.faultReasonPr}%`,
                fill: "rgba(255, 255, 255, 1)",
                fontSize: 12,
                x: -size[0] / 2 + 25,
                y: 11 - size[1] / 2,
                textAlign: "center",
                textBaseline: "top",
              },
              draggable: true,
              name: "ratio-shape",
            });
            group.addShape("text", {
              attrs: {
                // text: `${cfg.label}`,
                text: fittingString(cfg.nodeName, 120, 12),
                fill: "rgba(255, 255, 255, 1)",
                fontSize: 12,
                x: -size[0] / 2 + 65,
                y: 7,
              },
              draggable: true,
              name: "label-shape",
            });
            // // 是否有子节点
            // if(cfg.children && cfg.children.length > 0) {
            //     group.addShape('marker', {
            //         attrs: {
            //             x:size[0] /2 -5,
            //             y: 0,
            //             r: 7,
            //             symbol: cfg.collapsed ? G6.Marker.expand : G6.Marker.collapse,
            //             stroke: '#fff',
            //             fill: 'rgba(52,64,83,0.8)',
            //             lineWidth: 1,
            //             cursor: 'pointer', // 鼠标变手
            //         },
            //         name: 'collapse-icon',
            //     });
            //     if(cfg.depth === 0) {
            //         group.addShape('marker', {
            //             attrs: {
            //                 x: 0,
            //                 y: 20,
            //                 r: 7,
            //                 symbol: cfg.collapsed ? G6.Marker.expand : G6.Marker.collapse,
            //                 stroke: '#fff',
            //                 fill: 'rgba(52,64,83,0.8)',
            //                 lineWidth: 1,
            //                 cursor: 'pointer', // 鼠标变手
            //             },
            //             name: 'collapse-icon',
            //         });
            //     }
            // }
            return keyShape;
          },
          setState(name, value, item) {
            if (name === "collapsed") {
              const marker = item
                .get("group")
                .findAll((ele) => ele.get("name") === "collapse-icon");
              marker[0] &&
                marker[0].attr(
                  "symbol",
                  value ? G6.Marker.collapse : G6.Marker.expand
                );
              // 如果是根节点需要处理两个marker
              if (item._cfg.model.depth === 0) {
                marker[1] &&
                  marker[1].attr(
                    "symbol",
                    value ? G6.Marker.collapse : G6.Marker.expand
                  );
              }
            } else if (name === "hover") {
              const label = item
                .get("group")
                .findAll((ele) => ele.get("name") === "sub-keyShape");
              label[0].attr(
                "stroke",
                value ? "rgba(255,255,255,0.4)" : "rgba(255, 255, 255, 0)"
              );
              label[0].attr(
                "fill",
                value ? "rgba(255, 255, 255, 0.3)" : "rgba(255, 255, 255, 0.2)"
              );
              label[0].attr("shadowBlur", value ? 10 : 0);
              label[0].attr(
                "shadowColor",
                value ? "rgba(26,90,170,0.8)" : "rgba(255, 255, 255, 0)"
              );
            }
          },
          update: undefined,
        },
        "rect"
      );
      // other node
      G6.registerNode(
        "leaf",
        {
          draw: (cfg, group) => {
            // const width = G6.Util.getTextSize(cfg.nodeName, 12)[0] +100;
            const size = [200, 28];
            const keyShape = group.addShape("rect", {
              attrs: {
                width: size[0],
                height: size[1],
                x: -size[0] / 2,
                y: -size[1] / 2,
                // fill: 'rgba(255, 255, 255, 0.2)',
                fill: "rgba(126, 72, 255, 0)",
                stroke: "#48AAFF", // 边框颜色
                lineWidth: 1, // 边框宽度
                radius: 5,
              },
              draggable: true,
              name: "leaf-keyShape",
            });
            group.addShape("rect", {
              attrs: {
                x: -size[0] / 2 + 5,
                y: -size[1] / 2 + 3,
                width: 38,
                height: 22,
                radius: 8,
                fill: cfg.increase,
                stroke: cfg.borderColor, // 边框颜色
                lineWidth: 1, // 边框宽度
              },
              draggable: true,
            });
            group.addShape("text", {
              attrs: {
                text: `${cfg.faultReasonPr}%`,
                fill: "rgba(255, 255, 255, 1)",
                fontSize: 12,
                x: -size[0] / 2 + 25,
                y: 10 - size[1] / 2,
                textAlign: "center",
                textBaseline: "top",
              },
              draggable: true,
              name: "ratio-shape",
            });
            group.addShape("text", {
              attrs: {
                // text: `${cfg.label}`,
                text: fittingString(cfg.nodeName, size[0] - 80, 12),
                fill: "rgba(255, 255, 255, 1)",
                fontSize: 12,
                x: -size[0] / 2 + 65,
                y: 7,
              },
              draggable: true,
            });
            // 是否有子节点
            if (cfg.children && cfg.children.length > 0) {
              group.addShape("marker", {
                attrs: {
                  x: size[0] / 2 - 5,
                  y: 0,
                  r: 6,
                  symbol: cfg.collapsed ? G6.Marker.expand : G6.Marker.collapse,
                  stroke: "#fff",
                  fill: "rgba(52,64,83,0.8)",
                  lineWidth: 1,
                  cursor: "pointer", // 鼠标变手
                },
                name: "collapse-icon",
              });
              if (cfg.depth === 0) {
                group.addShape("marker", {
                  attrs: {
                    x: 0,
                    y: 20,
                    r: 6,
                    symbol: cfg.collapsed
                      ? G6.Marker.expand
                      : G6.Marker.collapse,
                    stroke: "#fff",
                    fill: "rgba(52,64,83,0.8)",
                    lineWidth: 1,
                    cursor: "pointer", // 鼠标变手
                  },
                  name: "collapse-icon",
                });
              }
            }
            return keyShape;
          },
          setState(name, value, item) {
            if (name === "collapsed") {
              const marker = item
                .get("group")
                .findAll((ele) => ele.get("name") === "collapse-icon");
              marker[0] &&
                marker[0].attr(
                  "symbol",
                  value ? G6.Marker.collapse : G6.Marker.expand
                );
              // 如果是根节点需要处理两个marker
              if (item._cfg.model.depth === 0) {
                marker[1] &&
                  marker[1].attr(
                    "symbol",
                    value ? G6.Marker.collapse : G6.Marker.expand
                  );
              }
            } else if (name === "hover") {
              const label = item
                .get("group")
                .findAll((ele) => ele.get("name") === "leaf-keyShape");
              label[0].attr(
                "stroke",
                value ? "rgba(255,255,255,0.4)" : "rgba(255, 255, 255, 0)"
              );
              label[0].attr(
                "fill",
                value ? "rgba(255, 255, 255, 0.3)" : "rgba(255, 255, 255, 0.2)"
              );
              label[0].attr("shadowBlur", value ? 10 : 0);
              label[0].attr(
                "shadowColor",
                value ? "rgba(26,90,170,0.8)" : "rgba(255, 255, 255, 0)"
              );
            }
          },
        },
        "single-node"
      );

      G6.registerEdge("mid-point-edge", {
        // draw(cfg, group) {
        //     const startPoint = cfg.startPoint;
        //     const endPoint = cfg.endPoint;
        //     const shape = group.addShape('path', {
        //         attrs: {
        //             stroke: '#999',
        //             path: [
        //                 ['M', startPoint.x, startPoint.y],
        //                 ['L', endPoint.x / 2 + (1 / 2) * startPoint.x, startPoint.y], // 三分之一处
        //                 ['L', endPoint.x / 2 + (1 / 2) * startPoint.x, endPoint.y], // 三分之二处
        //                 ['L', endPoint.x, endPoint.y],
        //             ],
        //         },
        //     // 在 G6 3.3 及之后的版本中，必须指定 name，可以是任意字符串，但需要在同一个自定义元素类型中保持唯一性
        //     name: 'path-shape',
        //     });
        //     return shape;
        // },
        draw(cfg, group) {
          const startPoint = cfg.startPoint;
          const endPoint = cfg.endPoint;

          // 计算中间点，用于贝塞尔曲线的控制点
          const midPointX = (startPoint.x + endPoint.x) / 2;
          const midPointY = (startPoint.y + endPoint.y) / 2;

          // 使用贝塞尔曲线绘制平滑路径
          const shape = group.addShape("path", {
            attrs: {
              stroke: "#999",
              path: [
                ["M", startPoint.x, startPoint.y],
                ["Q", midPointX, midPointY, endPoint.x, endPoint.y], // 使用二次贝塞尔曲线
              ],
            },
            name: "path-shape",
          });

          return shape;
        },
        afterDraw(cfg, group) {
          const shape = group.get("children")[0];
          const midPoint = shape.getPoint(0.5); // 获取路径中间点
          const width = midPoint.x;
          const height = midPoint.y;

          var result = cfg.target.match(/-/g);
          var size = (result || []).length;
          var len = cfg.sourceNode._cfg.children.length;

          // if (size == 1 && len > 1) {
          //   group.addShape('rect', {
          //     attrs: {
          //       x: width + 10,
          //       y: height - 10,
          //       width: 20,
          //       height: 20,
          //       stroke: 'rgba(255, 255, 255, 0.2)',
          //       fill: 'rgba(52, 65, 89, 1)',
          //       radius: [12, 1, 1, 12]
          //     },
          //   });
          // } else if (cfg.source.includes("-") && size > 1 && len > 1) {
          //   group.addShape('rect', {
          //     attrs: {
          //       x: width + 10,
          //       y: height - 10,
          //       width: 20,
          //       height: 20,
          //       stroke: 'rgba(255, 255, 255, 0.2)',
          //       fill: 'rgba(52, 65, 89, 1)',
          //       radius: [12, 1, 1, 12]
          //     },
          //   });
          // }
        },
        // afterDraw(cfg, group) {
        //     const shape = group.get('children')[0];
        //     const midPoint = shape.getPoint(0.1);
        //     const width = cfg.startPoint.x
        //     const height = cfg.startPoint.y
        //     var result = cfg.target.match(/-/g)
        //     var size = (result || []).length
        //     var len = cfg.sourceNode._cfg.children.length
        //     if(size==1 && len >1){
        //         group.addShape('rect', {
        //             attrs: {
        //                 x: width+10,
        //                 y: height-10,
        //                 width: 20,
        //                 height: 20,
        //                 stroke: 'rgba(255, 255, 255, 0.2)',
        //                 fill: 'rgba(52, 65, 89, 1)',
        //                 radius: [12,1,1,12]
        //             },
        //         });

        //     }else if(cfg.source.includes("-") && size>1 && len >1){
        //           group.addShape('rect', {
        //             attrs: {
        //                 x: width+10,
        //                 y:height-10,

        //                 width: 20,
        //                 height: 20,
        //                 stroke: 'rgba(255, 255, 255, 0.2)',
        //                 fill: 'rgba(52, 65, 89, 1)',
        //                 radius: [12,1,1,12]
        //             },
        //         });
        //     }
        // }
      });

      G6.registerEdge("fund-polyline", {
        itemType: "edge",
        draw: function draw(cfg, group) {
          const startPoint = cfg.startPoint;
          const endPoint = cfg.endPoint;

          const Ydiff = endPoint.y - startPoint.y;

          const slope = Ydiff !== 0 ? Math.min(500 / Math.abs(Ydiff), 20) : 0;

          const cpOffset = slope > 15 ? 0 : 16;
          const offset = Ydiff < 0 ? cpOffset : -cpOffset;

          const line1EndPoint = {
            x: startPoint.x + slope,
            y: endPoint.y + offset,
          };
          const line2StartPoint = {
            x: line1EndPoint.x + cpOffset,
            y: endPoint.y,
          };

          // 控制点坐标
          const controlPoint = {
            x:
              ((line1EndPoint.x - startPoint.x) * (endPoint.y - startPoint.y)) /
                (line1EndPoint.y - startPoint.y) +
              startPoint.x,
            y: endPoint.y,
          };

          let path = [
            ["M", startPoint.x, startPoint.y],
            ["L", line1EndPoint.x, line1EndPoint.y],
            [
              "Q",
              controlPoint.x,
              controlPoint.y,
              line2StartPoint.x,
              line2StartPoint.y,
            ],
            ["L", endPoint.x, endPoint.y],
          ];

          if (Math.abs(Ydiff) <= 5) {
            path = [
              ["M", startPoint.x, startPoint.y],
              ["L", endPoint.x, endPoint.y],
            ];
          }

          const endArrow =
            cfg?.style && cfg.style.endArrow ? cfg.style.endArrow : false;
          if (isObject(endArrow)) endArrow.fill = stroke;
          const line = group.addShape("path", {
            attrs: {
              path,
              stroke: colorMap[cfg.data && cfg.data.type],
              lineWidth: 1.2,
              endArrow,
            },
            // must be assigned in G6 3.3 and later versions. it can be any string you want, but should be unique in a custom item type
            name: "path-shape",
          });

          const labelLeftOffset = 0;
          const labelTopOffset = 8;
          // amount
          const amount = group.addShape("text", {
            attrs: {
              text: cfg.data && cfg.data.amount,
              x: line2StartPoint.x + labelLeftOffset,
              y: endPoint.y - labelTopOffset - 2,
              fontSize: 14,
              textAlign: "left",
              textBaseline: "middle",
              fill: "#000000D9",
            },
            // must be assigned in G6 3.3 and later versions. it can be any string you want, but should be unique in a custom item type
            name: "text-shape-amount",
          });
          // type
          group.addShape("text", {
            attrs: {
              text: cfg.data && cfg.data.type,
              x: line2StartPoint.x + labelLeftOffset,
              y: endPoint.y - labelTopOffset - amount.getBBox().height - 2,
              fontSize: 10,
              textAlign: "left",
              textBaseline: "middle",
              fill: "#000000D9",
            },
            // must be assigned in G6 3.3 and later versions. it can be any string you want, but should be unique in a custom item type
            name: "text-shape-type",
          });
          // date
          group.addShape("text", {
            attrs: {
              text: cfg.data && cfg.data.date,
              x: line2StartPoint.x + labelLeftOffset,
              y: endPoint.y + labelTopOffset + 4,
              fontSize: 12,
              fontWeight: 300,
              textAlign: "left",
              textBaseline: "middle",
              fill: "#000000D9",
            },
            // must be assigned in G6 3.3 and later versions. it can be any string you want, but should be unique in a custom item type
            name: "text-shape-date",
          });
          return line;
        },
      });

      G6.registerEdge("smooth-edge", {
        draw(cfg, group) {
          const startPoint = cfg.startPoint;
          const endPoint = cfg.endPoint;

          // 计算两个拐弯点
          const controlPoint1 = {
            x: startPoint.x,
            y: (startPoint.y + endPoint.y) / 2,
          };
          const controlPoint2 = {
            x: endPoint.x,
            y: (startPoint.y + endPoint.y) / 2,
          };

          const path = [
            ["M", startPoint.x, startPoint.y],
            [
              "C",
              controlPoint1.x,
              controlPoint1.y,
              controlPoint2.x,
              controlPoint2.y,
              endPoint.x,
              endPoint.y,
            ],
          ];

          const shape = group.addShape("path", {
            attrs: {
              stroke: "#999",
              path: path,
            },
            name: "path-shape",
          });

          return shape;
        },
      });
      //初始化
      const container = document.getElementById(this.container);
      const width = container.scrollWidth;
      const height = container.scrollHeight || 500;
      this.tree = new G6.TreeGraph({
        container: this.container,
        width,
        height,
        fitView: true,
        fitViewPadding: [10, 10, 10, 10],
        // plugins: [tooltip],

        layout: {
          type: "compactBox", //dendrogram、CompactBox、mindmap
          direction: "TB",
          // direction: 'LR',
          linkDistance: 80,
          getHeight: () => {
            return 100;
          },
          getWidth: (node) => {
            //  return node.level === 0 ?50:150;
            return node.level === 0 ? 50 : 50;
          },
          getVGap: () => {
            return 50;
          },
          getHGap: () => {
            return 100;
          },
          // getHeight: function getHeight() {
          //   return 16;
          // },
          // getWidth: function getWidth() {
          //   return 16;
          // },
          // getVGap: function getVGap() {
          //   return 80;
          // },
          // getHGap: function getHGap() {
          //   return 20;
          // }
        },
        defaultNode: {
          type: "tree-node",
          anchorPoints: [
            [0, 0.5],
            [1, 0.5],
          ],
          style: {
            cursor: "pointer",
          },
        },
        defaultEdge: {
          type: "mid-point-edge", // round-poly、polyline、cubic-horizontal
          // type: 'fund-polyline',// round-poly、polyline、cubic-horizontal
          type: "cubic-horizontal", // round-poly、polyline、cubic-horizontal
          // type: 'smooth-edge',// round-poly、polyline、cubic-horizontal
          style: {
            radius: 8,
            lineWidth: 1,
            stroke: "rgba(72, 170, 255, 0.50)", // 边描边颜色
            // stroke: 'rgba(72, 170, 255, 0.50)', // 边描边颜色
          },
        },
        nodeStateStyles: {
          hover: {
            fill: "rgba(255,255,255,0.3)",
            // shadowBlur: 3,
            // shadowColor: '#ddd',
          },
        },
        minZoom: 0.2,
        maxZoom: 1.8,
        modes: {
          // default: [
          //     'drag-canvas',
          //     'zoom-canvas',
          //     'dice-mindmap',
          //     {
          //         type: 'tooltip',
          //         formatText(model) {
          //         return  `<div class="card">
          //                 <div class="card-item"><div> 故障名称:</div><div>${model.nodeName}</div></div>
          //                 <div class="card-item"><div> 节点ID:  </div><div>${model.nodeId}</div></div>
          //                 <div class="card-item"><div> 节点类型：</div><div>${model.nodeType == "1"? "根节点":"子节点"}</div></div>
          //                 <div class="card-item"><div> 节点描述：</div><div>${model.nodeDesc}</div></div>
          //                 <div class="card-item"><div> 原因概率：</div><div>${model.faultReasonPr===null?'-':model.faultReasonPr +'%'}</div></div>
          //                 <div class="card-item"><div> 判断方法：</div><div>${model.judgeMethods}</div></div>
          //                 <div class="card-item"><div> 处理方法：</div><div>${model.processingMethods}</div></div>
          //             </div>`;
          //         },
          //         offset: 20,
          //     },
          // ],
        },
      });

      this.tree.on("node:click", (e) => {
        if (e.target.get("name") == "collapse-icon") {
          e.item.getModel().collapsed = !e.item.getModel().collapsed;
          this.tree.setItemState(
            e.item,
            "collapsed",
            !e.item.getModel().collapsed
          );
          this.tree.layout();
        }
      });

      // this.tree.on('node:mouseenter', e => {
      //     this.tree.setItemState(e.item, 'hover', true);
      // })
      // this.tree.on('node:mouseleave', e => {
      //     this.tree.setItemState(e.item, 'hover', false);
      // });

      this.tree.data(this.dataTransform(this.treeData));
      this.tree.render();

      let erd = elementResizeDetectorMaker();
      erd.listenTo(document.getElementById(that.container), (element) => {
        const width = element.clientWidth;
        const height = element.clientHeight;
        that.tree.changeSize(width, height);
        // this.tree.fitCenter();
      });
    },
    dataTransform(data) {
      console.log(data);
      const changeData = (d, level = 0, color) => {
        const data = {
          ...d,
        };
        switch (level) {
          case 0:
            data.type = "root";
            break;
          case 1:
            data.type = "sub";
            break;
          default:
            data.type = "leaf";
            break;
        }
        data.hover = false;

        if (color) {
          data.color = color;
        }
        if (d.children) {
          data.children = d.children.map((child) =>
            changeData(child, level + 1, data.color)
          );
        }
        return data;
      };
      return changeData(data);
    },
  },
};
</script>
<style scoped>
.container {
  width: 100%;
  height: 100%;
}
.tree {
  width: 100%;
  height: 100%;
  margin: 0 auto;
  position: relative;
}
.g6-tooltip {
  border: 1px solid #05062e;
  border-radius: 4px;
  font-size: 12px;
  color: #545454;
  background-color: rgba(2, 25, 61, 1);
  padding: 10px 8px;
  box-shadow: rgb(174, 174, 174) 0px 0px 10px;
}
.g6-component-tooltip {
  color: #fff;
  background-color: rgba(2, 25, 61, 1);
  padding: 0px 10px 24px 10px;
  box-shadow: rgb(7, 25, 109) 0px 0px 10px;
}
</style>
