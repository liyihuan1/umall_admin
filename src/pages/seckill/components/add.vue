<template>
  <div>
    <el-dialog
      :title="info.isadd?'秒杀添加':'秒杀编辑'"
      :visible.sync="info.isshow"
      @closed="cancel"
      @opened="opened"
    >
      <!-- 2.v-model绑定user -->
      <el-form :model="user">
        <div>user:{{user}}</div>
        <el-form-item label="活动名称" label-width="100px">
          <el-input v-model="user.title"></el-input>
        </el-form-item>
        <el-form-item label="活动期限" label-width="100px">
          <template>
            <div class="block">
              <span class="demonstration"></span>
              <el-date-picker
                v-model="value1"
                type="daterange"
                range-separator="至"
                start-placeholder="开始日期"
                end-placeholder="结束日期">
              </el-date-picker>
            </div>
          </template>
        </el-form-item>



        <el-form-item label="一级分类" label-width="100px">
          <el-select v-model="user.first_cateid" @change="changeFirstCateId">
            <el-option label="--请选择--" value disabled></el-option>
            <el-option
              v-for="item in cateList"
              :key="item.id"
              :label="item.catename"
              :value="item.id"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="二级分类" label-width="100px">
          <el-select v-model="user.second_cateid"
          @change="changeSecondCateId">
            <el-option label="--请选择--" value disabled></el-option>
            <el-option
              v-for="item in secondCateList"
              :key="item.id"
              :label="item.catename"
              :value="item.id"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="商品" label-width="100px">
          <el-select v-model="user.goodsid">
            <el-option label="--请选择--" value disabled></el-option>
            <el-option label="sk2" value="1"></el-option>
            <el-option label="面膜" value="2"></el-option>
            <el-option label="水墨" value="3"></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="状态" label-width="100px">
          <el-switch v-model="user.status" :active-value="1" :inactive-value="2"></el-switch>
        </el-form-item>
      </el-form>

      <div slot="footer" class="dialog-footer">
        <el-button @click="cancel">取 消</el-button>
        <el-button type="primary" @click="add" v-if="info.isadd">添 加</el-button>
        <el-button type="primary" @click="update" v-else >修 改</el-button>
      </div>
    </el-dialog>
  </div>
</template>
  <script>
import { mapActions, mapGetters } from "vuex";
import {
  reqcatelist,
  reqgoodsAdd,
  reqgoodsDetail,
  reqgoodsUpdate
} from "../../../utils/http";
import { successalert } from "../../../utils/alert";
export default {
  props: ["info"],
  data() {
    return {
      // 1.初始化数据
      user: {
        title:"",
        begintime:"",
        endtime:"",
        first_cateid: "",
        second_cateid: "",
        goodsid:"",
        status: 1
      },
      //4.二级分类的列表
      secondCateList: [],
      //三级
      threeCateList:[]
    };
  },
  computed: {
    ...mapGetters({
      //3.1 分类的list
      seckillList: "seckill/list",
      //一级分类
      cateList:"cate/list",
    })
  },
  methods: {
    ...mapActions({
      //3.2分类列表的获取
      reqseckillList: "seckill/reqList",
      //7.2 规格列表的获取
      //二级
      reqSpecsList: "specs/reqList",
      reqList: "goods/reqList",
      reqTotal: "goods/reqTotal",
      //一级
      reqCateList: "cate/reqList",
    }),
    //5.修改了一级分类
    changeFirstCateId() {
      //二级分类选中的要清除
      this.user.second_cateid = "";

      this.getSecondList();
    },
    //根据一级分类，计算出二级分类的list
    getSecondList() {
      reqcatelist({ pid: this.user.first_cateid }).then(res => {
        if (res.data.code == 200) {
          this.secondCateList = res.data.list;
        }
      });
    },
    //根据一级、二级分类，计算出三级分类的list
    changeSecondCateId(){
      
    },
    //取消
    cancel() {
      if (!this.info.isadd) {
        this.empty();
      }
      this.info.isshow = false;
    },
    //11.清空
    empty() {
      this.user = {
        title:"",
        begintime:"",
        endtime:"",
        first_cateid: "",
        second_cateid: "",
        goodsid:"",
        status: 1
      };
      this.imgUrl = "";
      //二级分类的列表
      this.secondCateList = [];
    },
    //10.点了添加
    add() {
      //发请求
      reqgoodsAdd(data).then(res => {
        if (res.data.code == 200) {
          //1.弹框消失
          this.cancel();
          //2.数据清空
          this.empty();
          //3.弹成功
          successalert(res.data.msg);
          //4.刷新list
          this.reqList();
        }
      });
    },
    //获取一条数据
    getOne(id) {
      reqgoodsDetail({ id: id }).then(res => {
        if (res.data.code == 200) {
          this.user = res.data.list;
          //重新获取二级分类的list
          this.getSecondList();
          //补id
          this.user.id = id;
        }
      });
    },
    //修改
    update() {
      reqgoodsUpdate(data).then(res => {
        if (res.data.code == 200) {
          //1.弹框消失
          this.cancel();
          //2.数据清空
          this.empty();
          //3.弹成功
          successalert(res.data.msg);
          //4.刷新list
          this.reqList();
        }
      });
    },
  },
  mounted() {
    //3.3如果没有请过分类数据，就发一下请求
    if (this.cateList.length === 0) {
      this.reqCateList();
    }

    //7.3 请求规格list
    this.reqSpecsList(true);
  }
};
</script>

<style scoped>
/* 原生js的样式 */
.my-upload {
  width: 100px;
  height: 100px;
  border: 1px dashed #999;
  position: relative;
}
.my-upload .add {
  width: 100px;
  height: 100px;
  text-align: center;
  line-height: 100px;
  font-size: 30px;
  color: #666;
}
.my-upload .ipt {
  width: 100px;
  height: 100px;
  position: absolute;
  left: 0;
  top: 0;
  opacity: 0;
  cursor: pointer;
}
.my-upload .img {
  width: 100px;
  height: 100px;
  position: absolute;
  left: 0;
  top: 0;
}
</style>