<template>
  <div id="app">
    <!-- <colorPicker v-model="color" v-on:change="headleChangeColor"></colorPicker>{{color}} -->
    <cl-select v-model="selectVal" @change="headleChange">
      <cl-option v-for="item in arr" :key="item.value" :label="item.label" :value="item.value"></cl-option>
    </cl-select>{{selectVal}}
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      color: "#ff0000",
      arr: [],
      selectVal: 1
    }
  },
  methods: {
    headleChange (val) {
			console.log(val)
		}
  },
  created(){
    setTimeout(() => {
      this.arr = [
        {value:1, label: '1111111111111111111111111111111111111111111111111111111'},
        {value:2, label: '222'},
        {value:3, label: '333'},
      ]
      console.log(this.arr)
    }, 3000)
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
