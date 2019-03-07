<template>
  <div class="city">
    <div class="search-wrap">
      <div class="search">
        <i class="fa fa-search"></i>
        <input type="text" placeholder="输入城市名" v-model="city_val">
      </div>
      <button @click="$router.push({ name: 'address', params: { city:city }})">取消</button>
    </div>
    <div style="height: 100%;" v-if="searchList.length == 0">
      <div class="location">
        <Location @click="selectCity({ name: city})" :address="city" />
      </div>
      <Alphabet @selectCity="selectCity" ref="allcity" :cityInfo="cityInfo" :keys="keys"/>
    </div>
    <div class="search-list" v-else>
      <ul>
        <li @click="selectCity(item)" v-for="(item, index) in searchList" :key="index">{{ item.name }}</li>
      </ul>
    </div>
  </div>
</template>

<script>
import Location from '../components/Location'
import Alphabet from '../components/Alphabet'
export default {
  name:'City',
  data() {
    return {
      city_val: '',
      cityInfo: null,
      keys: [],
      allCities:[],
      searchList:[]
    }
  },
  computed: {
    city(){
      return (
        this.$store.getters.location.addressComponent.city || 
        this.$store.getters.location.addressComponent.province
      );
    }
  },
  components:{
    Location, Alphabet
  },created() {
    this.getCityInfo();
  },
  watch: {
    city_val(){
      // console.log(this.city_val);
      this.searchCity();
    }
  },
  methods: {
    getCityInfo(){
      this.$axios('/api/posts/cities').then((res => {
        // console.log(res.data);
        this.cityInfo = res.data;
        // 处理key 计算key
        this.keys = Object.keys(res.data);
        // console.log(this.keys);

        // hotcities这个key移除掉
        this.keys.pop();
        // keys排序
        this.keys.sort();
        // console.log(this.keys);
        this.$nextTick(() => {
          this.$refs.allcity.initScroll();
        });

        // 存储所有城市，用来搜索过滤
        this.keys.forEach(key => {
          // console.log(key);
          // 根据key来遍历城市
          this.cityInfo[key].forEach( city => {
            // console.log(city);
            this.allCities.push(city);
          });
        });
      })).catch((err => {
        console.log(err);
      }));
    },
    selectCity(city){
      console.log(city);
      this.$router.push({ name: 'address', params: { city: city.name } });
    },
    searchCity(){
      if (!this.city_val) {
        // 如果搜索框为空
        this.searchList = [];
      }else{
        // 根据输入框的关键字检索城市，并存入到searchList数组中
        this.searchList = this.allCities.filter( item => {
          return item.name.indexOf(this.city_val) != -1;
        });
      }
    }
  },
}
</script>

<style scoped>
.city {
  width: 100%;
  height: 100%;
  overflow: auto;
  box-sizing: border-box;
  padding-top: 45px;
}
.search-wrap {
  position: fixed;
  top: 0;
  height: 45px;
  width: 100%;
  background: #fff;
  box-sizing: border-box;
  padding: 3px 16px;
  display: flex;
  justify-content: space-between;
}
.search {
  background-color: #eee;
  border-radius: 10px;
  line-height: 40px;
  width: 85%;
  box-sizing: border-box;
  padding: 0 16px;
}
.search input {
  background: #eee;
  outline: none;
  border: none;
  margin-left: 5px;
}
.search-wrap button {
  outline: none;
  color: #009eef;
  border: none;
  background-color: #fff;
}

.location {
  background: #fff;
  padding: 8px 16px;
  /* height: 65px; */
  box-sizing: border-box;
}

.search-list {
  background: #fff;
  padding: 5px 16px;
}
.search-list li {
  padding: 10px;
  border-bottom: 1px solid #eee;
}
</style>
