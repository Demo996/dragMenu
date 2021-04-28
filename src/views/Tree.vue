<template>
  <div style="width: 100%; height: 100%">
    <div class="super-flow-demo1">
      <el-card
        style="float: left; width: 200px; height: 100%; margin-right: 10px"
      >
        <el-tag style="width: 100%; text-align: center" type="success"
          >数据转换</el-tag
        >
        <!-- <h5 style="color: blue"></h5> -->
        <el-menu
          default-active="2"
          class="el-menu-vertical-demo"
          @open="handleOpen"
          @close="handleClose"
        >
          <el-submenu
            v-for="(item, index) in parentList"
            :key="index"
            :index="item.id + ''"
          >
            <template slot="title">
              <i :class="arrow"></i>
              <span>{{ item.title }}</span>
            </template>
            <el-menu-item
              class="node-container"
              v-for="(citem, cindex) in item.nodeItemList"
              :key="cindex"
            >
              <i style="width: 20px; height: 20px" class="el-icon-document"></i>
              <span @mousedown="(evt) => nodeItemMouseDown(evt, citem.value)">{{
                citem.label
              }}</span>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-card>
      <el-card style="width: calc(100% - 220px); height: 100%; float: right">
        <div style="width: 100%; height: 100%" ref="flowContainer">
          <super-flow
            ref="superFlow"
            :graph-menu="graphMenu"
            :node-menu="nodeMenu"
            :link-menu="linkMenu"
            :link-base-style="linkBaseStyle"
            :link-style="linkStyle"
            :link-desc="linkDesc"
          >
            <template v-slot:node="{ meta }">
              <div
                @mouseup="nodeMouseUp"
                @click="nodeClick"
                class="flow-node ellipsis"
              >
                {{ meta.name }}
              </div>
            </template>
          </super-flow>
        </div>
      </el-card>
    </div>

    <el-dialog
      :title="drawerConf.title"
      :visible.sync="drawerConf.visible"
      :close-on-click-modal="false"
      width="500px"
    >
      <el-form
        @keyup.native.enter="settingSubmit"
        @submit.native.prevent
        v-show="drawerConf.type === drawerType.node"
        ref="nodeSetting"
        :model="nodeSetting"
      >
        <el-form-item label="节点名称" prop="name">
          <el-input
            v-model="nodeSetting.name"
            placeholder="请输入节点名称"
            maxlength="30"
          >
          </el-input>
        </el-form-item>
        <el-form-item label="节点描述" prop="desc">
          <el-input
            v-model="nodeSetting.desc"
            placeholder="请输入节点描述"
            maxlength="30"
          >
          </el-input>
        </el-form-item>
      </el-form>
      <el-form
        @keyup.native.enter="settingSubmit"
        @submit.native.prevent
        v-show="drawerConf.type === drawerType.link"
        ref="linkSetting"
        :model="linkSetting"
      >
        <el-form-item label="连线描述" prop="desc">
          <el-input v-model="linkSetting.desc" placeholder="请输入连线描述">
          </el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="drawerConf.cancel"> 取 消 </el-button>
        <el-button type="primary" @click="settingSubmit"> 确 定 </el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
const drawerType = {
  node: 0,
  link: 1,
};
export default {
  data() {
    return {
      arrow: "el-icon-caret-right",
      drawerType,
      drawerConf: {
        title: "",
        visible: false,
        type: null,
        info: null,
        open: (type, info) => {
          const conf = this.drawerConf;
          conf.visible = true;
          conf.type = type;
          conf.info = info;
          if (conf.type === drawerType.node) {
            conf.title = "节点";
            if (this.$refs.nodeSetting) this.$refs.nodeSetting.resetFields();
            this.$set(this.nodeSetting, "name", info.meta.name);
            this.$set(this.nodeSetting, "desc", info.meta.desc);
          } else {
            conf.title = "连线";
            if (this.$refs.linkSetting) this.$refs.linkSetting.resetFields();
            this.$set(
              this.linkSetting,
              "desc",
              info.meta ? info.meta.desc : ""
            );
          }
        },
        cancel: () => {
          this.drawerConf.visible = false;
          if (this.drawerConf.type === drawerType.node) {
            this.$refs.nodeSetting.clearValidate();
          } else {
            this.$refs.linkSetting.clearValidate();
          }
        },
      },
      linkSetting: {
        desc: "",
      },
      nodeSetting: {
        name: "",
        desc: "",
      },

      dragConf: {
        isDown: false,
        isMove: false,
        offsetTop: 0,
        offsetLeft: 0,
        clientX: 0,
        clientY: 0,
        ele: null,
        info: null,
      },
      parentList: [
        {
          title: "输入",
          id: 1,
          nodeItemList: [
            {
              label: "表输入",
              value: {
                width: 120,
                height: 40,
                meta: {
                  label: "表输入",
                  name: "表输入",
                },
              },
            },
            {
              label: "增量输入",
              value: {
                width: 120,
                height: 40,
                meta: {
                  label: "增量输入",
                  name: "增量输入",
                },
              },
            },
            {
              label: "Excel输入",
              value: {
                width: 120,
                height: 40,
                meta: {
                  label: "Excel输入",
                  name: "Excel输入",
                },
              },
            },
            {
              label: "文本文件输入",
              value: {
                width: 120,
                height: 40,
                meta: {
                  label: "文本文件输入",
                  name: "文本文件输入",
                },
              },
            },
          ],
        },
        {
          title: "转换",
          id: 2,
          nodeItemList: [
            {
              label: "表输入",
              value: {
                width: 120,
                height: 40,
                meta: {
                  label: "表输入",
                  name: "表输入",
                },
              },
            },
            {
              label: "增量输入",
              value: {
                width: 120,
                height: 40,
                meta: {
                  label: "增量输入",
                  name: "增量输入",
                },
              },
            },
            {
              label: "Excel输入",
              value: {
                width: 120,
                height: 40,
                meta: {
                  label: "Excel输入",
                  name: "Excel输入",
                },
              },
            },
            {
              label: "文本文件输入",
              value: {
                width: 120,
                height: 40,
                meta: {
                  label: "文本文件输入",
                  name: "文本文件输入",
                },
              },
            },
          ],
        },
      ],
      graphMenu: [
        [
          {
            // 选项 label
            label: "节点1",
            // 选项是否禁用
            disable(graph) {
              return !!graph.nodeList.find((node) => node.meta.label === "1");
            },
            // 选项选中后回调函数
            selected(graph, coordinate) {
              graph.addNode({
                width: 120,
                height: 40,
                coordinate,
                meta: {
                  label: "1",
                  name: "1",
                },
              });
            },
          },
          {
            label: "节点2",
            selected(graph, coordinate) {
              graph.addNode({
                width: 120,
                height: 40,
                coordinate,
                meta: {
                  label: "2",
                  name: "2",
                },
              });
            },
          },
          {
            label: "节点3",
            selected(graph, coordinate) {
              graph.addNode({
                width: 120,
                height: 40,
                coordinate,
                meta: {
                  label: "3",
                  name: "3",
                },
              });
            },
          },
        ],
        [
          {
            label: "节点4",
            selected(graph, coordinate) {
              graph.addNode({
                width: 120,
                height: 40,
                coordinate,
                meta: {
                  label: "4",
                  name: "4",
                },
              });
            },
          },
        ],
        [
          {
            label: "全选",
            selected: (graph) => {
              graph.selectAll();
            },
          },
        ],
      ],
      nodeMenu: [
        [
          {
            label: "删除",
            selected: (node) => {
              node.remove();
            },
          },
          {
            label: "编辑",
            selected: (node) => {
              this.drawerConf.open(drawerType.node, node);
            },
          },
        ],
      ],
      linkMenu: [
        [
          {
            label: "删除",
            selected: (link) => {
              link.remove();
            },
          },
          {
            label: "编辑",
            selected: (link) => {
              this.drawerConf.open(drawerType.link, link);
            },
          },
        ],
      ],

      linkBaseStyle: {
        color: "#666666", // line 颜色
        hover: "#FF0000", // line hover 的颜色
        textColor: "#666666", // line 描述文字颜色
        textHover: "#FF0000", // line 描述文字 hover 颜色
        font: "14px Arial", // line 描述文字 字体设置 参考 canvas font
        dotted: false, // 是否是虚线
        lineDash: [4, 4], // 虚线时生效
        background: "rgba(255,255,255,0.6)", // 描述文字背景色
      },
    };
  },
  mounted() {
    document.addEventListener("mousemove", this.docMousemove);
    document.addEventListener("mouseup", this.docMouseup);
    this.$once("hook:beforeDestroy", () => {
      document.removeEventListener("mousemove", this.docMousemove);
      document.removeEventListener("mouseup", this.docMouseup);
    });
  },
  methods: {
    handleOpen() {
      this.arrow = "el-icon-caret-bottom";
    },
    handleClose() {
      this.arrow = "el-icon-caret-right";
    },
    flowNodeClick(meta) {
      console.log(this.$refs.superFlow.graph);
    },
    linkStyle(link) {
      return {
        // hover: '#FF00FF'
      };
    },
    linkDesc(link) {
      return link.meta ? link.meta.desc : "";
    },
    settingSubmit() {
      const conf = this.drawerConf;
      if (this.drawerConf.type === drawerType.node) {
        if (!conf.info.meta) conf.info.meta = {};
        Object.keys(this.nodeSetting).forEach((key) => {
          this.$set(conf.info.meta, key, this.nodeSetting[key]);
        });
        this.$refs.nodeSetting.resetFields();
      } else {
        if (!conf.info.meta) conf.info.meta = {};
        Object.keys(this.linkSetting).forEach((key) => {
          this.$set(conf.info.meta, key, this.linkSetting[key]);
        });
        this.$refs.linkSetting.resetFields();
      }
      conf.visible = false;
    },

    nodeMouseUp(evt) {
      evt.preventDefault();
    },

    nodeClick() {
      console.log(arguments);
    },

    docMousemove({ clientX, clientY }) {
      const conf = this.dragConf;

      if (conf.isMove) {
        conf.ele.style.top = clientY - conf.offsetTop + "px";
        conf.ele.style.left = clientX - conf.offsetLeft + "px";
      } else if (conf.isDown) {
        // 鼠标移动量大于 5 时 移动状态生效
        conf.isMove =
          Math.abs(clientX - conf.clientX) > 5 ||
          Math.abs(clientY - conf.clientY) > 5;
      }
    },

    docMouseup({ clientX, clientY }) {
      const conf = this.dragConf;
      conf.isDown = false;

      if (conf.isMove) {
        const {
          top,
          right,
          bottom,
          left,
        } = this.$refs.flowContainer.getBoundingClientRect();

        // 判断鼠标是否进入 flow container
        if (
          clientX > left &&
          clientX < right &&
          clientY > top &&
          clientY < bottom
        ) {
          // 获取拖动元素左上角相对 super flow 区域原点坐标
          const coordinate = this.$refs.superFlow.getMouseCoordinate(
            clientX - conf.offsetLeft,
            clientY - conf.offsetTop
          );

          // 添加节点
          this.$refs.superFlow.addNode({
            coordinate,
            ...conf.info,
          });
        }

        conf.isMove = false;
      }

      if (conf.ele) {
        conf.ele.remove();
        conf.ele = null;
      }
    },

    nodeItemMouseDown(evt, info) {
      const { clientX, clientY, currentTarget } = evt;

      const { top, left } = evt.currentTarget.getBoundingClientRect();

      const conf = this.dragConf;
      const ele = currentTarget.cloneNode(true);

      Object.assign(this.dragConf, {
        offsetLeft: clientX - left,
        offsetTop: clientY - top,
        clientX: clientX,
        clientY: clientY,
        info,
        ele,
        isDown: true,
      });

      ele.style.position = "fixed";
      ele.style.margin = "0";
      ele.style.top = clientY - conf.offsetTop + "px";
      ele.style.left = clientX - conf.offsetLeft + "px";

      this.$el.appendChild(this.dragConf.ele);
    },
  },
};
</script>
<style>
html,
body {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
}
.el-card__body {
  height: 100% !important;
}
.el-icon-arrow-down:before {
  display: none !important;
}
.el-menu {
  border: none !important;
}
.el-submenu__title {
  padding-left: 0px !important;
}
.el-menu-item {
  height: 30px !important;
  line-height: 30px !important;
}
.text {
  font-size: 14px;
}

.item {
  padding: 18px 0;
}
</style>
<style lang="less" scoped>
.ellipsis {
  white-space: nowrap;
  text-overflow: ellipsis;
  overflow: hidden;
  word-wrap: break-word;
}

.link-base-style-form {
  .el-form-item {
    margin-bottom: 12px;
  }

  padding-bottom: 20px;
  border-bottom: 1px solid #dcdcdc;
}

.super-flow-demo1 {
  width: 100%;
  height: 100%;
  background-color: #f5f5f5;
  @list-width: 200px;

  > .node-container {
    width: @list-width;
    float: left;
    height: 100%;
    text-align: center;
    background-color: #ffffff;
  }

  > .flow-container {
    height: 100%;
    overflow: hidden;
  }

  .super-flow__node {
    .flow-node {
      box-sizing: border-box;
      width: 100%;
      height: 100%;
      line-height: 40px;
      padding: 0 6px;
      font-size: 12px;
    }
  }
}

.node-item {
  @node-item-height: 30px;

  font-size: 14px;
  display: inline-block;
  height: @node-item-height;
  width: 120px;
  margin-top: 20px;
  background-color: #ffffff;
  line-height: @node-item-height;
  box-shadow: 1px 1px 4px rgba(0, 0, 0, 0.3);
  cursor: pointer;
  user-select: none;
  text-align: center;
  z-index: 6;

  &:hover {
    box-shadow: 1px 1px 8px rgba(0, 0, 0, 0.4);
  }
}
</style>