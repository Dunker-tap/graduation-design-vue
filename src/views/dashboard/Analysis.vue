<template>
  <div>
<!--    <b-pdf :file="'60137d6395657f2fee283f13'"></b-pdf>-->
    <index-chart v-if="indexStyle==1"></index-chart>
    <index-bdc v-if="indexStyle==2"></index-bdc>
    <index-task v-if="indexStyle==3"></index-task>
    <div style="width: 100%;text-align: right;margin-top: 20px;display: none">
      请选择首页样式：
      <a-radio-group v-model="indexStyle">
        <a-radio :value="1">统计图表</a-radio>
        <a-radio :value="2">统计图表2</a-radio>
        <a-radio :value="3">任务表格</a-radio>
      </a-radio-group>
    </div>
  </div>
</template>

<script>
  import IndexChart from './IndexChart'
  import IndexTask from "./IndexTask"
  import IndexBdc from './IndexBdc'
  import FileMangement from "@/extends/FileMangement/fileMangement";
  export default {
    name: "Analysis",
    components: {
      FileMangement,
      IndexChart,
      IndexTask,
      IndexBdc
    },
    data() {
      return {
        indexStyle:3
      }
    },
    created() {

    },
    methods: {

    }
  }
</script>