<template>
  <div class="container">
    <div class="dashboard">
      <div class="title">
        <span>空氣品質指標 (AQI)</span>
        <select v-model="currentLocation">
          <option value="Loading" v-if="!location.length">Loading...</option>
          <option v-for="item in location" :key="item" :value="item">
            {{ item }}
          </option>
        </select>
      </div>
      <div class="readme">
        <table>
          <thead>
            <tr>
              <td class="level_0">0～50</td>
              <td class="level_1">51～100</td>
              <td class="level_2">101～150</td>
              <td class="level_3">151～200</td>
              <td class="level_4">201～300</td>
              <td class="level_5">301～400</td>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>良好</td>
              <td>普通</td>
              <td>對敏感族群<br />不健康</td>
              <td>對所有族群<br />不健康</td>
              <td>非常不健康</td>
              <td>危害</td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="banner">
        <div style="font-size: 36px;">
          {{ currentLocation }}
        </div>
        <div class="dot"></div>
        <div style="font-size: 16px;">
          {{ lastUpdate }} &nbsp;&nbsp;&nbsp;&nbsp;更新
        </div>        
      </div>
      <div class="detail">
        <table class="city" v-if="currentSite">
          <tbody>
            <tr>
              <td class="name">{{ currentSite.SiteName }}</td>
              <td class="aqi-value" :class="AQIClass(currentSite.Status)">
                {{ currentSite.AQI }}
              </td>
            </tr>
            <tr>
              <td colspan="2" class="value">
                <div class="value-detail bottom-line">
                  <div class="title">
                    <span class="c-name">臭氧 </span>
                    <span class="e-name">O3 (ppb)</span>                    
                  </div>
                  <div class="data">
                    {{ currentSite.O3 }}
                  </div>
                </div>
                <div class="value-detail bottom-line">
                  <div class="title">
                    <span class="c-name">懸浮微粒 </span>
                    <span class="e-name">PM10 (μg/m³)</span>                    
                  </div>
                  <div class="data">
                    {{ currentSite.PM10 }}
                  </div>
                </div>
                <div class="value-detail bottom-line">
                  <div class="title">
                    <span class="c-name">細懸浮微粒 </span>
                    <span class="e-name">PM2.5 (μg/m³)</span>                    
                  </div>
                  <div class="data">
                    {{ currentSite['PM2.5'] }}
                  </div>
                </div>
                <div class="value-detail bottom-line">
                  <div class="title">
                    <span class="c-name">一氧化碳 </span>
                    <span class="e-name">CO (ppm)</span>                    
                  </div>
                  <div class="data">
                    {{ currentSite.CO }}
                  </div>
                </div>
                <div class="value-detail bottom-line">
                  <div class="title">
                    <span class="c-name">二氧化硫 </span>
                    <span class="e-name">SO2 (ppb)</span>                    
                  </div>
                  <div class="data">
                    {{ currentSite.SO2 }}
                  </div>
                </div>
                <div class="value-detail">
                  <div class="title">
                    <span class="c-name">二氧化氮 </span>
                    <span class="e-name">NO2 (ppb)</span>                    
                  </div>
                  <div class="data">
                    {{ currentSite.NO2 }}
                  </div>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="other-city">
        <table class="city link" @click="currentSite = item" v-for="(item, index) in filterSite"
          :class="{ 'mr': index % 2 === 0, 'ml': index % 2 === 1 }"
          :key="index">
          <tbody>
            <tr>
              <td class="name">
                {{ item.SiteName }}
              </td>
              <td class="aqi-value" :class="AQIClass(item.Status)">{{ item.AQI }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <footer class="footer"></footer>
  </div>
</template>

<script>
import Vue from 'vue';
import axios from 'axios';
import VueAxios from 'vue-axios';

Vue.use(VueAxios, axios);

export default {
  data() {
    return {
      site: [],
      location: [],
      currentLocation: 'Loading',
      currentSite: [],
      lastUpdate: 'Loading',
    }
  },
  methods: {
    getAQI() {
      const api = 'https://cors-anywhere.herokuapp.com/http://opendata2.epa.gov.tw/AQI.json';
      const vm = this;
      this.$http.get(api).then((res) => {
        // eslint-disable-next-line
        console.log(res.data);
        vm.site = res.data;
        vm.lastUpdate = vm.site[0].PublishTime;
        vm.currentLocation = vm.site[0].County;
        vm.location = res.data.map(function (item) {
          return item.County;
        }).filter(function(item, pos, self) {
          return self.indexOf(item) == pos;
        });
      });
    },
    AQIClass(status) {
      switch (status) {
        case '良好':
          return 'level_0';
        case '普通':
          return 'level_1';
        case '對敏感族群不健康':
          return 'level_2';
        case '對所有族群不健康':
          return 'level_3';
        case '非常不健康':
          return 'level_4';
        case '危害':
          return 'level_5';
      }
    },
  },
  computed: {
    filterSite() {
      const vm = this;
      const filter =  vm.site.filter((item) => {
        return item.County === vm.currentLocation;
      });
      vm.currentSite = filter[0];
      return filter;
    }
  },
  created() {
    document.title = '新手 JS 地下城 5F - 全台空氣指標儀表板';
    this.getAQI();
  },
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css?family=Noto+Sans+TC:700|Open+Sans:400,700');
$bg: #F0F0F0;
$level_0: #95F084;
$level_1: #FFE695;
$level_2: #FFAF6A;
$level_3: #FF5757;
$level_4: #9777FF;
$level_5: #AD1774;
$colors: $level_0, $level_1, $level_2, $level_3, $level_4, $level_5;
@for $i from 1 through length($colors) {
  $value: ($i - 1);
  .level_#{$value} {
      background: nth($colors, $i) !important;
  }
}
body {
  box-sizing: border-box;
  display: flex;
  justify-items: center;
  align-items: center;
  height: 98vh;
  font-family: 'Open Sans', 'Noto Sans TC', sans-serif;
  font-weight: bold;
}
select {
  width: 350px;
  height: 46px;
  border: 3px solid black;
  position: relative;
  top: -9px;
  font-size: 16px;
  font-family: 'Noto Sans TC';
  text-indent: 10px;
}
table {
  border-collapse: collapse;
}
.container {
  width: 1280px;
  height: 1080px;
  background: $bg;
  margin: 0 auto;
}
.dashboard {
  height: calc(1080px - 34px - 80px);
  padding: 40px 85px;
  display: grid;
  grid-template-columns: 350px 350px 350px;
  grid-column-gap: 30px;
  grid-template-rows: 110px 54px 535px;
  grid-row-gap: 32px;
  .title {
    font-size: 40px;
    font-family: 'Noto Sans TC';
  }
  .readme {
    grid-area: 1 / 2 / 2 / 4;
    table {
      width: 100%;
      height: 100%;
      tr td {
        border: 3px solid black;
        height: 50px;
        width: 120px;
        text-align: center;
        background: white;
      }
    }  
  }
  .banner {
    grid-area: 2 / 1 / 3 / 4;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .detail {
    grid-area: 3 / 1 / 4 / 2;
  }
  .other-city {
    grid-area: 3 / 2 / 4 / 4;
  }
}
.footer {
  height: 34px;
  background: black;
}
.dot {
  height: 3px;
  width: 768px;
  border-bottom: 3px dotted #000000;
}
.city {
  width: 350px;
  background: white;
  display: inline-block;
  margin-bottom: 16px;
  .name {
    width: 190px;
    height: 97px;
    border: 3px solid black;
    font-size: 36px;
    text-align: center;
  }
  .aqi-value {
    width: 160px;
    height: 97px;
    border: 3px solid black;
    font-size: 48px;
    font-weight: bold;
    text-align: center;
  }
  .value {
    width: 100%;
    height: 438px;
    border: 3px solid black;
    padding: 30px;
    .value-detail {
      display: flex;
      justify-content: space-between;
      align-items: baseline;
      padding-bottom: 12px;
      .c-name {
        font-size: 24px;
      }
      .e-name {
        font-size: 16px;
        font-weight: normal;
        font-family: 'Open Sans';
      }
      .data {
        font-size: 24px;
      }
    }
  }
}
.mr {
  margin-right: 12px;
}
.ml {
  margin-left: 12px;
}
.link {
  cursor: pointer;
  &:hover {
    background: #CCC;
    box-shadow: 5px 5px 3px #666;
  }
}
.bottom-line {
  border-bottom: 2px solid black;
}
</style>
