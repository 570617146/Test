<template>
  <div class="wrapper" :style="{width:width+'px'}">
    <!-- 搜索 -->
    <div class="search">
      <span>组件名称</span>
      <el-input
        size="medium"
        placeholder="请输入"
        suffix-icon="iconfont el-icon-tickets"
        v-model="moduleName"
      ></el-input>
      <span>分类</span>
      <el-select size="medium" clearable v-model="classifyValue" filterable placeholder="请选择">
        <el-option
          v-for="item in classifyOptions"
          :key="item.value"
          :label="item.label"
          :value="item.label"
        ></el-option>
      </el-select>
      <span>状态</span>
      <el-select size="medium" clearable v-model="stateValue" filterable placeholder="请选择">
        <el-option
          v-for="item in stateOptions"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        ></el-option>
      </el-select>
      <el-button size="medium" type="primary" @click="query">查询</el-button>
      <el-button size="medium" @click="reset">重置</el-button>
    </div>
    <!-- 顶部按钮 -->
    <div class="top">
      <el-button size="small" type="primary">新增</el-button>
      <el-button size="small" @click="Delete" type="danger">删除</el-button>
    </div>
    <!-- 内容 -->
    <div class="content">
      <div class="item">
        <el-checkbox @change="select('all')" v-model="checked"></el-checkbox>
        <span>组件名称</span>
        <span>分类</span>
        <span>版本</span>
        <span>开发语言</span>
        <span>最后更新时间</span>
        <span>状态</span>
        <span>操作</span>
      </div>
      <div class="item" v-for="(item,i) of list" :key="i">
        <el-checkbox @change="select" v-model="checkeds[i]"></el-checkbox>
        <span v-text="item.name"></span>
        <span v-text="item.classify"></span>
        <span v-text="item.versions"></span>
        <span v-text="item.development"></span>
        <span v-text="item.time"></span>
        <span :class="item.state==1?'putaway':''" v-text="item.state==1?'上架':'下架'"></span>
        <span>
          <el-button size="mini" type="primary">查看</el-button>
          <el-button size="mini">编辑</el-button>
          <el-button @click="soldOut(i)" size="mini">下架</el-button>
        </span>
      </div>
    </div>
    <!-- 分页控件 -->
    <div class="paging">
      <span>共{{list.length}}条</span>
      <el-select size="medium" v-model="pagingValue" style="width:110px;">
        <el-option
          v-for="item in pagingOptions"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        ></el-option>
      </el-select>
      <el-pagination layout="prev, pager, next" :total="10"></el-pagination>
      <span>前往</span>
      <el-input size="medium" style="width:50px" v-model="page"></el-input>
      <span>页</span>
    </div>
    <!-- 增加数据 -->
    <div class="mask"></div>
    <div class="add">
      <el-form label-position="right" label-width="80px" :model="formLabelAlign">
        <el-form-item label="组件名称">
          <el-input v-model="formLabelAlign.name"></el-input>
        </el-form-item>
        <el-form-item label="分类">
          <el-input v-model="formLabelAlign.classify"></el-input>
        </el-form-item>
        <el-form-item label="版本">
          <el-input v-model="formLabelAlign.versions"></el-input>
        </el-form-item>
        <el-form-item label="开发语言">
          <el-input v-model="formLabelAlign.development"></el-input>
        </el-form-item>
        <el-form-item label="状态">
          <el-input v-model="formLabelAlign.state"></el-input>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      width: 1000,
      moduleName: "",
      classifyOptions: [],
      classifyValue: "",
      stateOptions: [
        {
          value: "1",
          label: "上架"
        },
        {
          value: "0",
          label: "下架"
        }
      ],
      stateValue: "",
      // 控制全选
      checked: false,
      // 子选择
      checkeds: [],
      pagingOptions: [
        {
          value: "选项1",
          label: "10条/页"
        },
        {
          value: "选项2",
          label: "20条/页"
        }
      ],
      pagingValue: "10条/页",
      page: 1,
      list: [],
      formLabelAlign: {
        id: 0,
        name: "",
        classify: "",
        versions: "",
        development: "",
        time: "",
        state: ""
      }
    };
  },
  created() {
    this.width = window.screen.availWidth - 30;
    this.load();
  },
  methods: {
    // 加载数据
    load() {
      this.axios.get("/subassembly").then(result => {
        // console.log(result.data);
        this.list = result.data;
        this.checkeds = [];
        for (let i = 0; i < result.data.length; i++) {
          this.checkeds.push(false);
        }
        this.classifyOptions = [];
        let classifyArr = this.deteleObject(result.data, "classify");
        for (let i = 0; i < classifyArr.length; i++) {
          this.classifyOptions.push({
            value: `选项${i + 1}`,
            label: classifyArr[i].classify
          });
        }
      });
    },
    // 去除数组里重复的有重复名称的对象
    deteleObject(arr, name) {
      var hash = {};
      return arr.reduce(function(item, next) {
        hash[next[name]] ? "" : (hash[next[name]] = true && item.push(next));
        return item;
      }, []);
    },
    // 选中 支持全选与取消
    select(msg) {
      // console.log(this.checkeds);
      let checkeds = this.checkeds;
      if (msg == "all") {
        for (let i = 0; i < checkeds.length; i++) {
          checkeds[i] = this.checked;
        }
      } else {
        let s = 0;
        for (let i = 0; i < checkeds.length; i++) {
          if (checkeds[i] == false) {
            s++;
          }
        }
        if (s == 0) {
          this.checked = true;
        } else {
          this.checked = false;
        }
      }
    },
    // 重置
    reset() {
      this.moduleName = "";
      this.classifyValue = "";
      this.stateValue = "";
      this.load();
    },
    // 查询
    query() {
      let condition = "";
      if (this.moduleName != "") {
        condition += "name=" + this.moduleName;
      }
      if (this.classifyValue != "") {
        condition += "&classify=" + this.classifyValue;
      }
      if (this.stateValue != "") {
        condition += "&state=" + this.stateValue;
      }
      this.axios.get(`/subassembly?${condition}`).then(result => {
        // console.log(result.data);
        if (condition == "") {
          this.list = [];
        } else {
          this.list = result.data;
        }
      });
    },
    // 删除
    Delete() {
      let checkeds = this.checkeds;
      for (let i = 0; i < checkeds.length; i++) {
        if (checkeds[i] == true) {
          this.axios.delete("/subassembly/" + this.list[i].id).then(result => {
            console.log(result);
            this.load();
          });
        }
      }
    },
    // 下架
    soldOut(i) {
      if (this.list[i].state == 0) return;
      let time = new Date();
      let year = time.getFullYear();
      let moth = time.getMonth() + 1;
      moth = moth < 10 ? "0" + moth : moth;
      let date = time.getDate() < 10 ? "0" + time.getDate() : time.getDate();
      this.axios
        .patch(`/subassembly/${this.list[i].id}`, {
          time: year + "-" + moth + "-" + date,
          state: 0
        })
        .then(result => {
          console.log(result);
          this.load();
        });
    }
  }
};
</script>

<style scope>
.wrapper {
  /* border: 1px solid red; */
  overflow: hidden;
  /* position: relative; */
}
.search {
  /* height: 70px;
  line-height: 70px; */
  padding: 15px 0;
}
.search > .el-input {
  width: 200px;
}
.search > span {
  margin: 0 10px 0 10px;
}
.search > .el-button.el-button--primary {
  margin-left: 10px;
}
.top {
  height: 50px;
  line-height: 50px;
  padding-left: 10px;
}
.content {
  padding: 0 10px 0 10px;
  margin-top: 20px;
  color: #626469;
}
.content > .item {
  display: flex;
  width: 100%;
  height: 50px;
  line-height: 50px;
  border-bottom: 1px solid #ebeef5;
}
.content > .item > .el-checkbox {
  width: 3%;
  text-align: center;
}
.content > .item > span {
  /* width: 20%; */
  width: 16%;
  text-align: left;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
}
.content > .item > span:last-child {
  width: 19%;
}
.paging {
  /* float: right; */
  display: flex;
  justify-content: flex-end;
  height: 50px;
  line-height: 50px;
  margin-bottom: 20px;
  font-size: 16px;
  color: #626469;
}
.paging .el-pagination,
.paging .el-pager {
  display: flex;
  align-items: center;
}
.paging .el-button--text {
  font-size: 16px;
  color: #626469;
}
.paging > span {
  margin: 0 20px 0 20px;
}
.paging .el-input--medium .el-input__inner {
  font-size: 16px;
}
.putaway {
  color: #93cb7d;
}
.add {
  display: none;
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: #fff;
}
</style>