<template>
  <div class="goDemo">
    <el-row style="margin-top:10px;text-align:left;">
      <el-button size="mini" @click="initTreeModel" round>初始化树图</el-button>
      项目使用Vue + ElementUI + Webpack 集成了GoJs的流程图插件，基本实现了流程图的各种操作，方便大家使用
    </el-row>
    
    <el-row style="margin-top:10px;">
      <el-col :span="18">
        <div id="mygoChart" style="width:1000px; height:800px; background-color: #F5F5F5;"></div>
      </el-col>
      <el-col :span="6" style="text-align:left;">
          如下功能：<br/>
            1、树形拖拽<br/>
            2、树形放大缩小<br/>
            3、节点的增、删、改（节点鼠标右键）<br/>
            4、节点关系的拖拽连接（鼠标点击节点边缘，拖拽指向其他没有关联的节点，自动连接）<br/>
            5、图纸上初始化一颗新树<br/><br/>
          注： 打开浏览器F12 ，console打印节点操作的各种信息，方便大家使用
      </el-col>
    </el-row>
    <el-dialog title="树图命名" :visible.sync="dialogTreeNameVisible">
      <el-form>
        <el-form-item
          label="树图名称">
          <el-input type="text" v-model="tree_group_name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogTreeNameVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveTreeName">确 定</el-button>
      </div>
    </el-dialog>

    <el-dialog title="节点编辑" :visible.sync="dialogUpdateFormVisible">
      <el-form :model="node_form">
        <el-form-item
          v-for="(v,i) in lables_col"
          :key="i"
          :label="v.cn_name">
          <el-input :readonly="v.en_name=='text'?false:true" :type="v.en_name=='text'?'textarea':'text'" :rows="2" v-model="node_form[v.en_name]" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogUpdateFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveNodeInfo">确 定</el-button>
      </div>
    </el-dialog>
    
  </div>
</template>

<script>
// import { Inspector } from '@/api/DataInspector'

export default {
  data() {
    return {
      tree_group_name: "tree_model",
      dialogTreeNameVisible: false,
      div_show: false,
      nodeIdCounter: 50,
      dialogUpdateFormVisible: false,
      sel_nodeInfo: null,
      lables_col: [
        { en_name: "key", cn_name: "编码" },
        { en_name: "text", cn_name: "内容" },
        { en_name: "group", cn_name: "树标" }
      ],
      node_form: { key: "", text: "", group: "" },
      modelData: {
        nodeDataArray: [
          // 节点信息数组
          {
            key: 8,
            text: "根节点",
            group: "tree_1204"
          },
          {
            key: 1,
            text: "节点一",
            group: "tree_1204"
          },
          {
            key: 2,
            text: "节点二",
            group: "tree_1204"
          },
          {
            key: 3,
            text: "节点三",
            group: "tree_1204"
          },
          {
            key: 4,
            text: "节点四",
            group: "tree_1204"
          },
          {
            key: 5,
            text: "节点五",
            group: "tree_1204"
          }
        ],
        linkDataArray: [
          // 节点关系数组
          { id: 1001, from: 8, to: 1, group: "tree_1204" },
          { id: 1002, from: 8, to: 2, group: "tree_1204" },
          { id: 1003, from: 1, to: 3, group: "tree_1204" },
          { id: 1004, from: 1, to: 4, group: "tree_1204" },
          { id: 1005, from: 4, to: 5, group: "tree_1204" }
        ]
      },
      rm_list_node: [],
      rm_list_link: []
    };
  },
  mounted() {
    var mySelf = this;
    const MAKE = go.GraphObject.make;

    // 图表初始化配置
    mySelf.myDiagram = MAKE(go.Diagram, "mygoChart", {
      "grid.visible": true, //显示网格
      grid: MAKE(
        // 网格样式
        go.Panel,
        "Grid",
        MAKE(go.Shape, "LineH", {
          stroke: "lightgray",
          strokeWidth: 0.5,
          interval: 5
        }),
        MAKE(go.Shape, "LineV", {
          stroke: "lightgray",
          strokeWidth: 0.5,
          interval: 5
        })
      ),
      padding: 20,
      initialDocumentSpot: go.Spot.TopCenter, // 树图显示位置
      initialViewportSpot: go.Spot.TopCenter,
      maxSelectionCount: 1,
      allowDrop: true,
      allowDelete: false,
      allowCopy: false,
      allowClipboard: false,
      // LinkDrawn: mySelf.showLinkLabel, // 节点连线
      LinkRelinked: mySelf.showLinkLabel,
      scrollsPageOnFocus: false,
      "undoManager.isEnabled": true,
      "toolManager.mouseWheelBehavior": go.ToolManager.WheelZoom, //有鼠标滚轮事件放大和缩小
      layout: MAKE(go.TreeLayout, {
        angle: 90,
        layerSpacing: 30
      }),
      ModelChanged: function(e) {
        // console.log(e)
        // if (e.isTransactionFinished)
        //     console.log(e)
      },
      LinkReshaped: function(e) {
        console.log(JSON.parse(mySelf.myDiagram.model.toJson()).linkDataArray);
        console.log(e.subject.data);
        mySelf.myDiagram.model.removeLinkData(e.subject.data);
        // mySelf.myDiagram.model.addLinkData({from: e.subject.data.from, to: e.subject.data.to, group: 'tree_001'})
        return;
      }
    });

    // 树的modify监听事件
    mySelf.myDiagram.addDiagramListener("Modified", function(e) {
      var button = document.getElementById("SaveButton");
      if (button) button.disabled = !mySelf.myDiagram.isModified;
      var idx = document.title.indexOf("*");
      if (mySelf.myDiagram.isModified) {
        if (idx < 0) document.title += "*";
      } else {
        if (idx >= 0) document.title = document.title.substr(0, idx);
      }
    });

    mySelf.myDiagram.addDiagramListener("LinkDrawn", function(e) {
      // console.log(e.parameter)
    });

    // 节点布局及数据绑定
    mySelf.myDiagram.nodeTemplate = MAKE(
      go.Node,
      "Auto",
      MAKE(go.Shape, "RoundedRectangle", {
        width: 120,
        height: 80,
        strokeWidth: 0,
        fill: "#CD5B45",
        portId: "",
        fromLinkable: true,
        toLinkable: true
      }),
      MAKE(
        go.TextBlock,
        { margin: 15, stroke: "#ffffff" },
        new go.Binding("text", "text", function(c) {
          if (c.length > 6) {
            return c.substring(0, 6) + "...";
          } else {
            return c;
          }
        })
      )
    );


    

    mySelf.myDiagram.nodeTemplate.selectionAdornmentTemplate = MAKE(
      go.Adornment,
      "Spot",
      MAKE(
        go.Panel,
        "Auto",
        MAKE(go.Shape, "RoundedRectangle", {
          fill: null,
          stroke: "#BC8F8F",
          strokeWidth: 2
        }),
        MAKE(go.Placeholder)
      ),
      MAKE(
        "Button",
        {
          alignment: go.Spot.BottomCenter,
          click: mySelf.addNewNodeClick // this function is defined below
        },
        MAKE(go.Shape, "PlusLine", { width: 12, height: 12 })
      )
    );

    //节点右键菜单配置
    mySelf.myDiagram.nodeTemplate.contextMenu = MAKE(
      go.Adornment,
      "Vertical",
      MAKE(
        "ContextMenuButton",
        MAKE(go.Shape, {
          fill: "#ffffff",
          height: 30,
          width: 55,
          strokeWidth: 0,
          margin: 0,
          cursor: "pointer"
        }),
        MAKE(
          go.TextBlock,
          "下级",
          { cursor: "pointer" },
          {
            margin: 8,
            font: "13px sans-serif",
            stroke: "gray"
          }
        ),
        {
          click: mySelf.addNewNodeClick
        }
      ),
      MAKE(
        "ContextMenuButton",
        MAKE(go.Shape, {
          fill: "#ffffff",
          height: 30,
          width: 55,
          strokeWidth: 0,
          margin: 0,
          cursor: "pointer"
        }),
        MAKE(
          go.TextBlock,
          "编辑",
          { cursor: "pointer" },
          {
            margin: 8,
            font: "13px sans-serif",
            stroke: "gray"
          }
        ),
        {
          click: mySelf.updateNodeInfo
        }
      ),
      MAKE(
        "ContextMenuButton",
        MAKE(go.Shape, {
          fill: "#ffffff",
          height: 30,
          width: 55,
          strokeWidth: 0,
          margin: 0,
          cursor: "pointer"
        }),
        MAKE(
          go.TextBlock,
          "移除",
          { cursor: "pointer" },
          {
            margin: 8,
            font: "13px sans-serif",
            stroke: "gray"
          }
        ),
        {
          click: mySelf.removeNodes
        }
      )
    );

    // 节点连接配置
    mySelf.myDiagram.linkTemplate = MAKE(
      go.Link,
      {
        relinkableFrom: true,
        relinkableTo: true,
        routing: go.Link.AvoidsNodes,
        curve: go.Link.JumpOver,
        corner: 5,
        toShortLength: 4
      },
      MAKE(go.Shape, { stroke: "gray" }),
      MAKE(go.Shape, { toArrow: "Standard", stroke: "gray", fill: "gray" })
    );

    // 控制节点关联线的走向（用于自定义link）
    mySelf.myDiagram.toolManager.linkingTool.temporaryLink.routing =
      go.Link.Orthogonal;
    mySelf.myDiagram.toolManager.relinkingTool.temporaryLink.routing =
      go.Link.Orthogonal;

    // 加载节点及关系数据
    // this.loadTreeData();
    var nodedata = [
      { key: 1, text: "根节点", group: "tree_001" },
      { key: 2, text: "节点一", group: "tree_001" },
      { key: 3, text: "节点二", group: "tree_001" },
      { key: 4, text: "节点三", group: "tree_001" },
      { key: 5, text: "节点四", group: "tree_001" },
      { key: 6, text: "节点五", group: "tree_001" },
      { key: 7, text: "节点六", group: "tree_001" },
      { key: 8, text: "节点七", group: "tree_001" },
      { key: 9, text: "节点八", group: "tree_001" },
      { key: 10, text: "节点九", group: "tree_001" },
      { key: 11, text: "节点十", group: "tree_001" },
      { key: 12, text: "节点十一", group: "tree_001" }
    ];
    var linkdata = [
      { from: 1, to: 2, group: "tree_001" },
      { from: 1, to: 3, group: "tree_001" },
      { from: 2, to: 4, group: "tree_001" },
      { from: 2, to: 9, group: "tree_001" },
      { from: 3, to: 7, group: "tree_001" },
      { from: 3, to: 8, group: "tree_001" },
      { from: 4, to: 5, group: "tree_001" },
      { from: 4, to: 6, group: "tree_001" },
      { from: 7, to: 10, group: "tree_001" },
      { from: 8, to: 11, group: "tree_001" },
      { from: 9, to: 12, group: "tree_001" }
    ];

    var model = new go.GraphLinksModel(nodedata, linkdata);
    mySelf.myDiagram.model = model;
   
  },
  methods: {
    // 加载一颗树
    loadTreeData() {
      this.myDiagram.model = go.Model.fromJson(this.modelData);
    },

    //初始化新树
    initTreeModel() {
      // 初始化一颗新树
      this.dialogTreeNameVisible = true;
    },

    // 新增节点关系
    addLink(link) {
      // console.log(link)
      this.myDiagram.model.addLinkData(link);
      // 后台添加新增的link数据
    },
    // 存储树的组别名称
    saveTreeName() {
      this.myDiagram.model.nodeDataArray = [];
      this.myDiagram.model.linkDataArray = [];
      console.log({ key: 1, text: "NewNode", group: this.tree_group_name });
      var newemp = { key: 1, text: "NewNode", group: this.tree_group_name };
      this.myDiagram.model.addNodeData(newemp);
      this.dialogTreeNameVisible = false;
    },
    // 获取新增节点的key（或id）
    getNextKey() {
      // 新增节点的key，可以从数据库中获取表中最大的node的ID
      var key = this.nodeIdCounter;
      while (this.myDiagram.model.findNodeDataForKey(key) !== null) {
        key = this.nodeIdCounter++;
      }
      return key;
    },
    // 获取整棵树的数据
    getTreeData() {
      let varset = this.myDiagram.model.toJson();
      console.log("这是获取编辑后的树的总数据：");
      console.log(varset);
    },

    // 默认添加子节点
    addNewNodeClick(e, obj) {
      const new_key = this.getNextKey();
      var clicked = obj.part; //获取点击节点的数据
      if (clicked !== null) {
        var thisemp = clicked.data;

        console.log({ text: "NewNode", group: thisemp.group });
        console.log({ from: thisemp.key, to: "new_key", group: thisemp.group });

        var newemp = {
          key: new_key,
          text: "NewNode",
          group: thisemp.group
        };
        var newlink = { from: thisemp.key, to: new_key, group: thisemp.group };

        this.myDiagram.model.addNodeData(newemp);
        this.myDiagram.model.addLinkData(newlink);
        this.myDiagram.requestUpdate()
      }
    },

    // 节点编辑
    updateNodeInfo(e, obj) {
      this.dialogUpdateFormVisible = true;
      this.sel_nodeInfo = obj.part.adornedPart.data;
      const nodedata = obj.part.adornedPart.data;
      this.node_form = {
        key: nodedata.key,
        text: nodedata.text,
        group: nodedata.group
      };
    },

    // 编辑节点关系提交
    saveNodeInfo() {
      console.log(this.node_form.key); // 修改的节点的ID
      console.log(this.node_form); //修改后节点的内容

      const txt_new = this.node_form.text;
      this.myDiagram.model.setDataProperty(
        this.myDiagram.selection.first().data,
        "text",
        txt_new
      );
      this.dialogUpdateFormVisible = false;
    },

    // 移除节点及其对应子节点
    removeNodes(e, obj) {
      var node = obj.part.adornedPart;
      this.rm_list_node = []; // 需要删除节点的ID （key）
      this.rm_list_link = []; // 需要删除的关系的ID
      const that = this;
      node.findTreeParts().iterator.each(e => {
        if (e.data.hasOwnProperty("key")) {
          that.rm_list_node.push(e.data.key);
        } else {
          that.rm_list_link.push(e.data.id);
        }
      });

      if (node !== null) {
        this.$confirm(
          "此操作将移除当前节点及与该节点相关各子节点, 是否继续?",
          "提示",
          {
            confirmButtonText: "确定",
            cancelButtonText: "取消",
            type: "warning"
          }
        )
          .then(() => {
            this.myDiagram.startTransaction("remove dept");
            this.myDiagram.removeParts(node.findTreeParts());
            this.myDiagram.commitTransaction("remove dept");

            console.log(this.rm_list_node);
            console.log(this.rm_list_link);

            this.$message({
              type: "success",
              message: "相关节点移除成功!"
            });
          })
          .catch(() => {
            this.$message({
              type: "info",
              message: "已取消移除"
            });
          });
      }
    },

    // 节点样式
    nodeStyle() {
      return [
        new go.Binding("location", "loc", go.Point.parse).makeTwoWay(
          go.Point.stringify
        ),
        {
          locationSpot: go.Spot.Center
        }
      ];
    },
    // 手动增加节点关系方法
    makePort(name, align, spot, output, input) {
      const MAKE = go.GraphObject.make;
      var horizontal =
        align.equals(go.Spot.Top) || align.equals(go.Spot.Bottom);
      return MAKE(go.Shape, {
        fill: "transparent",
        strokeWidth: 0,
        width: horizontal ? NaN : 8,
        height: !horizontal ? NaN : 8,
        alignment: align,
        stretch: horizontal
          ? go.GraphObject.Horizontal
          : go.GraphObject.Vertical,
        portId: name,
        fromSpot: spot,
        fromLinkable: output,
        toSpot: spot,
        toLinkable: input,
        cursor: "pointer",
        mouseEnter: function(e, port) {
          if (!e.diagram.isReadOnly) port.fill = "#00bfff";
        },
        mouseLeave: function(e, port) {
          port.fill = "transparent";
        }
      });
    },

    // text样式
    textStyle() {
      return {
        font: "11pt Helvetica, Arial, sans-serif",
        stroke: "whitesmoke"
      };
    },

    showLinkLabel(e) {
      console.log(e.subject.fromNode.data);
      var label = e.subject.findObject("LABEL");
      console.log(label);
      if (label !== null)
        label.visible = e.subject.fromNode.data.category === "Conditional";
    }
  }
};
</script>

<style scoped>
.editor_table {
  background-color: burlywood !important;
  /* width: 400px;
  position: absolute;
  top: 50px; */
}
</style>

