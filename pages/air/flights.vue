<template>
  <div class="flights">
    <!-- 正文 开始 -->
    <div class="flights_main">
      <!-- 1 筛选模块 开始 -->
      <FlightsFilter
        v-if="flightsData.flights.length"
        :info="flightsData.info"
        :options="flightsData.options"
        @filterChange="filterChange"
      />
      <!-- 1 筛选模块 结束 -->

      <!-- 2 表单的头部 开始 -->
      <FlightsHead />
      <!-- 2 表单的头部 结束 -->

      <!-- 3 机票列表 开始 -->
      <div class="air_list">
        <FlightsItem v-for="(item) in currentFlights" :key="item.id" :data="item" />
      </div>
      <!-- 3 机票列表 结束 -->

      <!-- 4 分页组件 开始 -->
      <div>
        <!-- 
          :current-page 当前的页码
          :page-sizes  页容量 数组
          :page-size  当前 页容量
          @size-change  页容量改变事件
          @current-change  页码改变事件
        -->
        <el-pagination
          :current-page="page.currentPage"
          :page-sizes="page.pageSizes"
          :page-size="page.pageSize"
          :total="page.total"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          layout="total, sizes, prev, pager, next, jumper"
        ></el-pagination>
      </div>
      <!-- 4 分页组件 结束 -->
    </div>
    <!-- 正文 结束 -->

    <!-- 侧边栏 开始 -->
    <div class="flights_aside">2</div>
    <!-- 侧边栏 结束 -->
  </div>
</template>

<script>
import FlightsFilter from "@/components/air/FlightsFilter";
import FlightsHead from "@/components/air/FlightsHead";
import FlightsItem from "@/components/air/FlightsItem";
export default {
  components: {
    FlightsFilter,
    FlightsHead,
    FlightsItem
  },
  data() {
    return {
      flightsData: {
        // 机票列表数组
        flights: [],
        info: {},
        options: {}
      },

      // 筛选后的数据源 第一次加载页面的时候 值===总的数据源
      filterList: [],

      // 被分页后的 机票列表
      currentFlights: [],

      // 分页对象
      page: {
        // 当前页码
        currentPage: 1,
        // 页容量数组
        pageSizes: [1, 10, 20, 50, 100],
        // 当前 页容量
        pageSize: 10,
        // 总条数
        total: 1
      }
    };
  },
  methods: {
    // 1.获取机票数据
    getList(isFirst) {
      if (isFirst) {
        let form = this.$route.query;
        this.$axios.get("/airs", { params: form }).then(res => {
          // console.log(res);
          // 定义 所有的数据源
          this.flightsData = res.data;
          // 定义 过滤后 数组  数据源
          this.filterList=this.flightsData.flights
          // 定义 总条数
          this.page.total=this.filterList.length
          // 需要实现 数组分页 this.flightsData.flights
          // 记公式
          // let list = totalList.slice() 使用它
          // let list = totalList.slice(
          // (当前的页码-1)*当前的页容量，当前的页码*当前的页容量
          // )
          // 分页使用
          this.currentFlights = this.filterList.slice(
            (this.page.currentPage - 1) * this.page.pageSize,
            this.page.currentPage * this.page.pageSize
          );
        });
      } else {
        // 过滤后的总条数(属于过滤后的 总条数)
        this.page.total=this.filterList.length
        // 分页
        this.currentFlights = this.filterList.slice(
          (this.page.currentPage - 1) * this.page.pageSize,
          this.page.currentPage * this.page.pageSize
        );
      }
    },
    // 2. 页容量改变事件
    handleSizeChange(value) {
      // 选择 页容量
      // console.log(value);
      this.page.pageSize = value;
      this.getList();
    },
    // 3. 当前页码改变事件
    handleCurrentChange(value) {
      // 选择后的当前页码
      this.page.currentPage = value;
      this.getList();
    },
    filterChange(filterObj){
      // console.log(filterObj);
      // console.log(filterObj)出来后 > 数据的展示
      // {airport: "首都机场", flightTimes: "6|12", company: "东航", sizes: "S"}
      // {airport: "", flightTimes: "", company: "东航", sizes: "S"}

      // 1.先过滤 第一个条件 航空公司
      // 2. 当 航空公司 等于 空字符串的时候 表示  不用过滤
      let filterList=this.flightsData.flights.filter(v=>{
        // if (filterObj.company==="") {
        //   return true
        // }

        // 1. 航空公司的条件
        let isOkay1 = filterObj.company==="" || v.airline_name===filterObj.company
        return isOkay1
        // if (isOkay1) {
        //   return true
        // }else{
        //   return false
        // }
      })
      this.filterList=filterList
      this.getList()


    }
  },
  mounted() {
    this.getList(true);
  }
};
</script>

<style lang="less" scoped>
.flights {
  width: 1000px;
  margin: 0 auto;
  display: flex;
}
.flights_main {
  flex: 5;
}
.flights_aside {
  flex: 2;
}
</style>