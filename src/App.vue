<template>
  <div class="container-fluid no-padding .container">
    <button class="btn btn-light" @click="addnewTimer">Add</button>
    <button class="btn btn-light mx-1" @click="removeTimer()">Remove</button>
    <div class="row no-padding" v-bind:key="i" v-for="(row,i) in groupedTimers">
      <div v-for="(timer_col,index) in groupedTimers[i]" v-bind:key="index" class="col-sm no-padding">
        <div class="card-body no-padding">
        <h6 class="card-title no-padding"> Test {{i*per_collum+index+1}}</h6>
        <div v-if="timers[i*per_collum+index].time >= 120000" class="alert alert-danger no-padding .alert" role="alert">
          Testlösung auftragen!</div>
        <h6 v-if="timer_col.time > 0 && timer_col.time < 120000">
          Tupfer in Testlösung</h6>
        <h6 v-if="timer_col._2time > 0 && timer_col._2time < 900000">
          Test läuft</h6>
        <div v-if="timer_col._2time >= 900000" class="alert alert-success .alert" role="alert">
        Ergebnis ablesen!</div>
        <div class="timer-form">
          <div class="progress">
            <div class="progress-bar pg-beginning" role="progressbar"  v-bind:style="{ width: timers[i*per_collum+index].time/1000*0.0980392156862742 + '%'}" v-bind:aria-valuenow="0" aria-valuemin="0" aria-valuemax="100"></div>
            <div class="progress-bar bg-warning" role="progressbar"  v-bind:style="{width: ((timers[i*per_collum+index]._2time/1000*0.0980392156862742)) + '%'}" v-bind:aria-valuenow="0" aria-valuemin="0" aria-valuemax="100"></div>
          </div>
          <div class="display:inline" v-if="timer_col.section === 0">{{formattedElapsedTime(timer_col.time)}}</div>
          <div class="display:inline" v-if="timer_col.section === 1">{{formattedElapsedTime(timer_col._2time)}}</div>
          <button class="btn  btn-success "  @click="start(i*per_collum+index)" v-if="timers[i*per_collum+index].time === 0">Start</button>
          <button class="btn  btn-success "  @click="start(i*per_collum+index)" v-if="timers[i*per_collum+index].time >= 120000 && timers[i*per_collum+index]._2time <= 0 ">Weiter</button>
          <button class="btn btn-danger "  @click="reset(i*per_collum+index)">Reset</button>
        </div>
      </div>
    </div>
  </div>
  </div>
</template>

<script>

export default {
  name: 'app',
  comments:{

  },
  data() {
    return {
      per_collum: 5,
      audio: new Audio(),
      timers:[
        {
          Name: '',
          start: 0,
          _2Start: 0,
          run: 0,
          section: 0,
          _2time: 0,
          time: 0,
          timer: undefined
        }
      ]
    }
  }
  ,
  computed:{
    groupedTimers() {
      return _.chunk(this.timers,5)
    }
  },
  methods: {
    removeTimer(){
      clearInterval(this.timers[this.timers.length-1].timer);
      this.timers.pop()
    },
    addnewTimer() {
      this.timers.push({
        Name: '',
        start: 0,
        _2Start: 0,
        run: 0,
        section: 0,
        _2time: 0,
        time: 0,
        timer: undefined
      })
    },
    start(index) {
      this.audio.play();
      this.audio.src = require('./test.mp3');
      if(this.timers[index].time <=120000){
        this.timers[index].start = Date.now()
      }
      if(this.timers[index].time >=120000){
        this.timers[index]._2Start = Date.now()
      }
      this.timers[index].run = 1;
      if(this.timers[index].time >= 120000){
        this.timers[index].section = 1;
      }
      this.timers[index].timer = setInterval(() => {
        if(this.timers[index].run === 0){
          clearInterval(this.timers[index].timer)
        }
        if(this.timers[index].section === 0 ){
          this.timers[index].time = Date.now()-this.timers[index].start;
        }else{
          this.timers[index]._2time = Date.now()-this.timers[index]._2Start;
        }
        if(this.timers[index].time >= 120000 && this.timers[index].section !== 1 ){
          clearInterval(this.timers[index].timer);
          this.timers[index].run =0;
          this.audio.play();
        }
        if(this.timers[index]._2time >= 900000){
          clearInterval(this.timers[index].timer);
          this.timers[index].run =0;
          this.audio.play();
        }

      }, 1000);
    },
    reset(index) {
      clearInterval(this.timers[index].timer);
      this.timers[index].time = 0;
      this.timers[index].section = 0;
      this.timers[index].run = 0;
      this.timers[index]._2time = 0;
      this.timers[index]._2Start = 0;
      this.timers[index].start = 0;
    },
    formattedElapsedTime(elapsedTimee) {
      const date = new Date(null);
        date.setSeconds(elapsedTimee / 1000);
      const utc = date.toUTCString();
      return utc.substr(utc.indexOf(":") - 2, 8);
    }
  }
}

</script>

<style>
.alert {
  font-size: 70%;
}
</style>
