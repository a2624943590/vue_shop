<template>
   <div>
     <el-button type="primary" @click="exitBtn">退出</el-button>
   </div>
</template>

<script>
export default {
  name: 'Home',
  methods: {
    exitBtn () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    }
  }
}
</script>

<style scoped>

</style>
