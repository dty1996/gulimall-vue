<template>
  <div>
    <el-tree :data="menus" :props="defaultProps" @node-click="handleNodeClick" :expand-on-click-node="false"
      show-checkbox node-key="catId" :default-expanded-keys="expandedKey"   draggable :allow-drop="allowDrop"  :allow-drag="allowDrag" >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">
            新增
          </el-button>
          <el-button v-if="node.childNodes.length == 0" type="text" size="mini" @click="() => remove(node, data)">
            删除
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog title="提示" :visible.sync="dialogVisible" width="30%" >
      <el-form :model="category">
        <el-form-item label="分类名称" >
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCategory()">确 定</el-button>
      </div>
    </el-dialog>
  </div>

</template>



<script>
import http from '../../../utils/httpRequest';

export default {
  data() {
    return {
      maxLevel: 0,
      category:{
        name: "",
        parentCid:1,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: ""
      },
      menus: [],
      dialogVisible: false,
      expandedKey: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    };
  },
  methods: {
    handleNodeClick(data) {

    },
    getMenues() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list"),
        method: 'get'
      }).then(({ data }) => {
        console.log("数据" + data);
        this.menus = data.data;
      })
    },
    append(data) {
      console.log("新增按钮", data);
      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = ++data.catLevel;
    },
    //添加三级分类，点击对话框填入数据后
    addCategory(){
      console.log("三级分类数据",this.category);
      this.$http({
         url: this.$http.adornUrl("/product/category/save"),
          method: "post",
          data: this.$http.adornData(this.category, false)
      }).then(({data}) => {
         this.$message({
            showClose: true,
            message: `菜单添加成功`,
            type: 'success'
          });
          //关闭对话框
          this.dialogVisible = false;
          this.getMenues();
          //设置需要展开的节点
          this.expandedKey = [this.category.parentCid];
      })
    },
    remove(node, data) {
      this.$confirm(`是否删除${data.name}菜单`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        var ids = [data.catId];
        this.$http({
          url: this.$http.adornUrl("/product/category/delete"),
          method: "post",
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
          this.$message({
            showClose: true,
            message: `菜单删除成功`,
            type: 'success'
          });
          this.getMenues();
          //设置需要展开的节点
          this.expandedKey = [node.parent.data.catId];
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },
    allowDrop(draggingNode, dropNode, type){
      //被拖拽的节点和拖拽后的父节点总层数不能为大于三
      this.countLevel(draggingNode.data);
      let deep = this.maxLevel - draggingNode.data.catLevel + 1;
      if (type === "inner"){
        return deep + dropNode.data.catLevel <= 3;
      } else {
        return deep + dropNode.data.catLevel-1  <= 3 ;
      }
      
    
      
    },
    countLevel(data){
      //找到所有子节点，递归查询最大深度
      if (data.children != null && data.children.length > 0 ) {
          for (let i = 0 ; i < data.children.length ; i++) {
              if (data.children[i].catLevel > this.maxLevel) {
                this.maxLevel = data.children[i].catLevel;
              }
              this.countLevel(data.children[i])
          }
           
      }                               
    },

    //判断是否可以拖拽
    allowDrag(node){
      return true;
    }
    

  },
  created() {
    this.getMenues();
  }
}
</script>

<style>
</style>
