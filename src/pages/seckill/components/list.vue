<template>
  <div>
    <el-table
      :data="list"
      style="width: 100%;margin-bottom: 20px;"
      row-key="id"
      border
      :tree-props="{children: 'children'}"
    >
      <el-table-column prop="goodsname" label="商品名称" sortable ></el-table-column>
      <el-table-column label="状态">
        <template slot-scope="scope">
          <el-button type="primary" v-if="scope.row.status===1">启用</el-button>
          <el-button type="info" v-else>禁用</el-button>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="180">
        <template slot-scope="scope">
          <el-button type="primary" @click="edit(scope.row.id)">编辑</el-button>
          <del-btn @confirm="del(scope.row.id)"></del-btn>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>
  <script>
import { mapActions, mapGetters } from "vuex";
import {  reqgoodsDel } from "../../../utils/http";
import { successalert } from "../../../utils/alert";
export default {
  data() {
    return {};
  },
  computed: {
    ...mapGetters({
      list: "seckill/list",
    })
  },
  methods: {
    ...mapActions({
      reqList: "seckill/reqList",
    }),
    //删除
    del(id) {
      reqgoodsDel({ id: id }).then(res => {
        if (res.data.code == 200) {
          successalert(res.data.msg);
          this.reqList();
          //总数
          this.reqTotal()
        }
      });
    },
    //编辑
    edit(id) {
      this.$emit("edit", id);
    }
  },
  mounted() {
    //获取列表
    this.reqList();

    //获取总数
    this.reqTotal()
  }
};
</script>

  <style scoped>
  img{
    width: 80px;
    height: 80px;
  }
</style>