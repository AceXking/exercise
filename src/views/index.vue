<template>
  <div class="home">
    <el-button type="primary" plain @click="go(1)">顺序练习</el-button>
    <el-button type="primary" plain @click="go(2)">随机练习</el-button>
    <el-button type="primary" plain @click="go(2)">错题库</el-button>
  </div>
</template>

<script>
export default {
  name: "",
  data() {
    return {

    }
  },
  created() {

  },
  methods: {
    go(type) {
      this.$router.push({
        path: '/exercise',
        query: {
          type
        }
      })
    }
  }
}
</script>
<style lang="less" scoped>
.home {
  padding: 10px;
}
</style>