<template>
  <div class="container-fluid no-padding">
    <button class="btn btn-light" @click="addnewTimer">Add</button>

    <div class="row" v-bind:key="i" v-for="(timer,i) in groupedTimers">
      <div v-for="(timer,index) in groupedTimers[i]" v-bind:key="index" class="col-lg">
        <span class="float-right" style="cursor:pointer" @click="removeTimer()">X</span>
        <h4 class="card-title"> Test {{i*3+index+1}}</h4>
        <div v-if="timers[i*3+index].time >= 120000 && timers[i*3+index].time < 121000 && timers[i*3+index].section === 0" class="alert alert-danger blink_me" role="alert">
          Testlösung auftragen!</div>
        <h6 v-if="timers[i*3+index].time > 0 && timers[i*3+index].time < 120000">Tupfer in Testlösung</h6>
        <h6 v-if="timers[i*3+index].time > 120000 && timers[i*3+index].time < 1020000">Test läuft</h6>
        <div v-if="timers[i*3+index]._2time >= 900000" class="alert alert-success blink_me" role="alert">
        Ergebnis ablesen!</div>
        <div class="timer-form">
          <div class="progress">
            <div class="progress-bar pg-beginning" role="progressbar"  v-bind:style="{ width: timers[i*3+index].time/1000*0.0980392156862742 + '%'}" v-bind:aria-valuenow="0" aria-valuemin="0" aria-valuemax="100"></div>
            <div class="progress-bar bg-success" role="progressbar"  v-bind:style="{width: ((timers[i*3+index]._2time/1000*0.0980392156862742)) + '%'}" v-bind:aria-valuenow="0" aria-valuemin="0" aria-valuemax="100"></div>
          </div>
          <div class="display:inline" v-if="timer.section === 0">{{formattedElapsedTime(timer.time)}}</div>
          <div class="display:inline" v-if="timer.section === 1">{{formattedElapsedTime(timer._2time)}}</div>
          <button class="btn btn-success" @click="start(i*3+index)" v-if="timers[i*3+index].time === 0">start</button>
          <button class="btn btn-success" @click="start(i*3+index)" v-if="timers[i*3+index].time >= 120000 && timers[i*3+index]._2time <= 0 ">Weiter</button>
          <button class="btn btn-danger"  @click="reset(i*3+index)">Reset</button>
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
      timer:[{
        timer: undefined
      }
      ],
      timers:[
        {
          Name: '',
          start: 0,
          _2Start: 0,
          run: 0,
          section: 0,
          _2time: 0,
          time: 0
        }
      ]
    }
  },
  computed:{
    groupedTimers() {
      return _.chunk(this.timers,3)
    }
  },
  methods: {
    removeTimer(){
      this.timer.pop()
      this.timers.pop()
    },
    addnewTimer() {
      this.timer.push({
        timer: undefined
      })
      this.timers.push({
        Name: '',
        start: 0,
        _2Start: 0,
        run: 0,
        section: 0,
        _2time: 0,
        time: 0
      })
    },
    start(index) {
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
      this.timer[index] = setInterval(() => {
        if(this.timers[index].section === 0 ){
          this.timers[index].time = Date.now()-this.timers[index].start;
        }else{
          this.timers[index]._2time = Date.now()-this.timers[index]._2Start;
        }
        if(this.timers[index].time >= 120000 && this.timers[index].time < 121000 && this.timers[index].section !== 1 ){
          clearInterval(this.timer[index].timer);
          this.timers[index].run =0;
          const audio = new Audio(require('./234524_foolboymedia_notification-up-1 (online-audio-converter.com).mp3'));
          audio.play();
        }
        if(this.timers[index]._2time >= 900000){
          clearInterval(this.timer[index].timer);
          this.timers[index].run =0;
          const audio = new Audio(require('./234524_foolboymedia_notification-up-1 (online-audio-converter.com).mp3'));
          audio.play();
        }

      }, 1000);
    },
    reset(index) {

      clearInterval(this.timer[index].timer);
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

</style>
