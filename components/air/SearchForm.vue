<template>
  <div class="search_form">
    <div class="search_title">
      <div
        class="title_item"
        v-for="(item,index) in ['单程', '往返']"
        :key="index"
        :class="currentIndex===index?'active':''"
      >{{item}}</div>
    </div>
    <div class="search_content"> 
      <!--
        fetch-suggestions 当输入框的值改变的时候 会被调用
        querySearchAsync函数
        @select 输入框的下拉列表 点击事件，handleSelect 可以获取到被点击 元素的值
      -->
      <el-form label-width="80px">
        <el-form-item label="出发城市">
          <el-autocomplete
            :fetch-suggestions="querySearchAsync"
            @select="handleSelect1"
            v-model="form.departCity"
            placeholder="请搜索出发城市"
          ></el-autocomplete>
        </el-form-item>
        <!-- 换 -->
        <div class="city_change" @click="handleCityChange">换</div>
        <!-- 换 -->
        <el-form-item label="到达城市">
          <el-autocomplete             
          :fetch-suggestions="querySearchAsync"
            @select="handleSelect2"
            v-model="form.destCity"
          placeholder="请搜索到达城市"></el-autocomplete>
        </el-form-item>
        <el-form-item label="出发时间">
          <el-date-picker
            type="date"
            placeholder="选择日期"
            v-model="form.departDate"
            value-format="yyyy-MM-dd"
            style="width: 100%;"
          ></el-date-picker>
        </el-form-item>
        <el-form-item>
          <el-button style="width:100%;" type="primary"
          @click="handleGetTicket">搜索</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      currentIndex: 0,
      form: {
        // 出发城市
        departCity: "广州",
        // 出发城市代码
        departCode: "CAN",
        // 目标城市
        destCity: "上海",
        // 目标城市代码
        destCode: "SHA",
        // 出发时间
        departDate:"2019-11-29"
      }
    };
  },
  methods: {
    // 1. 搜索建议的输入 事件
    querySearchAsync(queryString, callback) {
      // 1. queryString = 当前输入框的值
      // 2. 把关键字 “广” 发送请求
      // callback([{}]) callback里面放 对象数组
       if (queryString) {
        this.$axios
          .get("/airs/city", { params: { name: queryString } })
          .then(res => {

            let cityArr=res.data.data;
            cityArr.forEach(v=>{
              // 把广州市的 "市" 字移除 因为 后台数据不含有"市"
              v.name=v.name.replace("市","")
              v.value=v.name;
            });
            callback(cityArr)
          });
      }
    },
    // 2. 点击 出发城市
    handleSelect1(item) {
      console.log(item);
      this.form.departCode=item.sort
    },
    // 3. 点击目标城市
    handleSelect2(item) {
      console.log(item);
      this.form.destCode=item.sort
    },
    // 4. 点击换，交换城市,编码
    handleCityChange(){
      let form = JSON.parse(JSON.stringify(this.form))
      // this.form.departCity=form.destCity;
      // this.form.departCode=form.destCode;
      // this.form.destCity=form.departCity;
      // this.form.destCode=form.departCode;
      // -----------LOW!!!----------------
      [this.form.departCity,this.form.departCode,this.form.destCity,this.form.destCode]=[this.form.destCity,this.form.destCode,this.form.departCity,this.form.departCode]
    },
    // 5点击搜索
    handleGetTicket(){
      // 要做一个表单验证
      this.$router.push({path:"/air/flights",query:this.form})
    }

  }
};
</script>

<style lang="less" scoped>
.search_form {
}

.search_title {
  height: 50px;
  background-color: #fff;
  display: flex;
  .title_item {
    flex: 1;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: #eee;
  }
  .active {
    background-color: #fff;
    position: relative;
    &::before {
      content: "";
      position: absolute;
      width: 100%;
      height: 3px;
      top: 0;
      left: 0;
      background-color: orange;
    }
  }
}
.search_content {
  padding: 20px 50px 20px 25px;
  position: relative;
  .city_change {
    position: absolute;
    background-color: #666;
    width: 22px;
    height: 22px;
    display: flex;
    justify-content: center;
    align-items: center;
    color: #fff;
    font-size: 13px;
    right: 11px;
    top: 59px;
    cursor: pointer;
    &::before {
      content: "";
      width: 30px;
      height: 19px;
      border: 1px solid #ccc;
      border-left: none;
      border-bottom: none;
      position: absolute;
      top: -20px;
      left: -20px;
    }
    &::after {
      content: "";
      width: 30px;
      height: 19px;
      border: 1px solid #ccc;
      border-left: none;
      border-top: none;
      position: absolute;
      bottom: -20px;
      left: -20px;
    }
  }
}
</style>