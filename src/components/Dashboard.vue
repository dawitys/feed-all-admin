<template>
<div id="Dashboard">
  <v-container fluid class="bg">
    <div class="flex-wrapper white--text">
        <div id="left" class="box">
    <div class="box__header">
      <h3 class="box__header-title">today's stats</h3>
    </div>
    <div class="box__body">
      <div class="stats stats--main">
        <div class="stats__amount">5</div>
        <div class="stats__caption">new registrations</div>
        <div class="stats__change">
          <div class="stats__value stats__value--positive">+6%</div>
          <div class="stats__period">this week</div>
        </div>
      </div>
      <div class="stats">
        <div class="stats__amount">354</div>
        <div class="stats__caption">plates served</div>
        <div class="stats__change">
          <div class="stats__value">+3</div>
          <div class="stats__period">this week</div>
        </div>
      </div>
  
      <div class="stats">
        <div class="stats__amount">23</div>
        <div class="stats__caption">paid clients</div>
        <div class="stats__change">
          <div class="stats__value stats__value--negative">-12%</div>
          <div class="stats__period">this week</div>
        </div>
      </div>
  
      <div class="stats">
        <div class="stats__amount">15</div>
        <div class="stats__caption">unpaid clients</div>
        <div class="stats__change">
          <div class="stats__value">+78</div>
          <div class="stats__period">this week</div>
        </div>
      </div>
    </div>
  </div>
  <div id="right"class="box">
    <div class="box__header">
      <h3 class="box__header-title">All time stats</h3>
    </div>
    <div class="box__body">
      <div class="stats stats--main">
        <div class="stats__amount">12698</div>
        <div class="stats__caption">plates served</div>
        <div class="stats__change">
          <div class="stats__value stats__value--positive">+6%</div>
          <div class="stats__period">all time</div>
        </div>

      <div class="stats">
        <div class="stats__amount">6302</div>
        <div class="stats__caption">peoples</div>
        <div class="stats__change">
          <div class="stats__value">+78</div>
          <div class="stats__period">all time</div>
        </div>
      </div>  
      <div class="stats">
        <div class="stats__amount">268</div>
        <div class="stats__caption">client food centers</div>
        <div class="stats__change">
          <div class="stats__value">+3</div>
          <div class="stats__period">all time</div>
        </div>
      </div>
    </div>  
    <div class="stats">
      <div class="stats__amount">87359</div>
      <div class="stats__caption">visitors</div>
      <div class="stats__change">
        <div class="stats__value stats__value--negative">-12%</div>
        <div class="stats__period">all time</div>
      </div>
    </div>
    </div>
  </div>
        <!-- <v-btn id="linkToLota" color="pink darken-2" round large dark href="https://www.apartmani-lota.com" target="_blank">Lota Home <v-icon right>fa-external-link</v-icon></v-btn> -->
    </div>
  </v-container>
</div>
</template>

<script>
export default {
  name: 'Dashboard',
  data () {
    return {
      
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#Dashboard, .container, .layout, .flex{
  height:100%;
}
/*.bg{
  background-image: url('../assets/img/dashboard.jpg');
  background-color: rgba(10, 99, 80, 0.685);
  background-size:cover;
  background-blend-mode:multiply ;
}*/
.flex-wrapper{
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  justify-content: center;
  height: 100%;
  padding: 50px;
}
#linkToLota{
  margin-right: 0;
}
#left {
    left: 0;
    margin-left: 25rem;
}
#right{
    right: 0;
    margin-right: 5rem;
}

.box {
	background: linear-gradient(#2b9966, #083110);
	margin: 15px auto 0;
	border-radius: 5px;
    width: 350px;
    position: fixed;
}

.box__header {
	padding: 15px 25px;
	position: relative;
}

.box__header-title {
	color: #fff;
	font-size: 18px;
}

.box__body {
	padding: 0 25px;
}

/* STATS */

.stats {
	color: #fff;
	position: relative;
	padding-bottom: 25px;
}

.stats__amount {
	font-size: 42px;
	font-weight: bold;
	line-height: 1.2;
}

.stats__caption {
	font-size: 18px;

}

.stats__change {
	position: absolute;
	top: 10px;
	right: 0;
	text-align: right;
	color: #B1B7C8;
}

.stats__value {
	font-size: 18px;
}

.stats__period {
	font-size: 14px;
}

.stats__value--positive {
	color: #AEDC6F;
}

.stats__value--negative {
	color: #FB5055;
}

.stats--main .stats__amount {
	font-size: 54px;
}
</style>
