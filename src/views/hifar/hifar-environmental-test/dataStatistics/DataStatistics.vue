<template>
  <div class="dataStatistics">
    <iframe
      ref='iframe'
      :src='src'
      frameborder='0'
      height='100%'
      scrolling='auto'
      width='100%'
    />
  </div>
</template>

<script>
export default {
  name: "DataStatistics",
  data() {
    return {
      src: "http://192.168.3.85:3000/thirdTurn?t=eyJhbGciOiJIUzI1NiJ9.eyJpZE5hbWUiOiLotoXnuqfnrqHnkIblkZgiLCJ0ZW5hbnRJZCI6IjNjY2VmOWY3ZDY1ZTQ0NTc5OGRmNTczYTkyOTA0OTFlIiwibW9iaWxlIjoiIiwiZGVwdElkIjoiMGYwM2JlZWU2OWI4NDZiYjhjNzcyOGE5ZTJlOTBmMTkiLCJ1c2VyVHlwZSI6ImRldiIsImV4cCI6MzMyMDU4NzQwNTU5MTIsInVzZXJJZCI6IjNjY2VmOWY3ZDY1ZTQ0NTc5OGRmNTczYTkyOTA0OTFlIiwiaWF0IjoxNjY5ODc0MDU1OTEyLCJzZWNyZXRMZXZlbCI6M30=.nVEWiYE4OGOdIY6-utLR3HI0SN-vOkZd3WLp9ZBGWik=&p=df59d0fc89cf47188953bb6a0cc2a6c5"
    }
  },
}
</script>

<style lang="less" scoped>
.dataStatistics {
  width: 100%;
  height: 100%
}
</style>