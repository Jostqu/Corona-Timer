<template>
  <div class="container-fluid no-padding .container">
    <button class="btn btn-light" @click="addnewTimer">Add</button>
    <button class="btn btn-light mx-1" @click="removeTimer()">Remove</button>
    <div class="row no-gutters" v-bind:key="i" v-for="(row,i) in groupedTimers">
      <div v-for="(timer_col,index) in groupedTimers[i]" v-bind:key="index" class="col-sm nopadding">
        <button class="btn btn-danger float-right btn-circle btn-md mr-3"  @click="reset(timer_col)">Reset</button>
        <div class="card-body no-padding no-gutters">
        <h6 class="card-title no-padding"> Test {{i*per_collum+index+1}}</h6>
        <div v-if="timer_col.time >= 120000 && timer_col.section === 0" class="alert alert-danger no-padding .alert" role="alert">
          Testlösung auftragen!</div>
        <h7 v-if="timer_col.run === 1 && timer_col.section === 0 ">
          Tupfer in Testlösung</h7>
        <h7 v-if="timer_col.run === 1 && timer_col.section=== 1">
          Test läuft</h7>
        <div v-if="timer_col._2time >= 900000" class="alert alert-success .alert" role="alert">
        Ergebnis ablesen!</div>
        <div class="timer-form">
          <div class="progress">
            <div class="progress-bar pg-beginning" role="progressbar"  v-bind:style="{ width: timers[i*per_collum+index].time/1000*0.0980392156862742 + '%'}" v-bind:aria-valuenow="0" aria-valuemin="0" aria-valuemax="100"></div>
            <div class="progress-bar bg-warning" role="progressbar"  v-bind:style="{width: ((timers[i*per_collum+index]._2time/1000*0.0980392156862742)) + '%'}" v-bind:aria-valuenow="0" aria-valuemin="0" aria-valuemax="100"></div>
          </div>
          <div class="display:inline" v-if="timer_col.section === 0">{{formattedElapsedTime(timer_col.time)}}</div>
          <div class="display:inline" v-if="timer_col.section === 1">{{formattedElapsedTime(timer_col._2time)}}</div>
          <button class="btn  btn-success "  @click="start(timer_col)" v-if="timer_col.run !== 1 && timer_col.time < 120000">Start</button>
          <button class="btn  btn-success "  @click="weiter(timer_col)" v-if="timer_col.time >= 120000 && timer_col.run !== 1 && timer_col._2time === 0">Weiter</button>

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
      audio: new Audio(require('./test.mp3')),
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
    start(timer) {
      timer.run = 1;
      this.audio.play();
      this.audio.pause();
      this.audio.currentTime = 0;
      timer.start = Date.now()
      timer.timer = setInterval(() => {
          timer.time = Date.now()-timer.start;
        if(timer.time >= 120000 && timer._2Start === 0){
          clearInterval(timer.timer);
          timer.run =0;
          this.audio.play();
        }
      }, 1000);
    },
    weiter(timer){
      timer._2Start = Date.now()
      timer.section = 1;
      timer.run = 1;
      timer.timer = setInterval(() => {
        if(timer._2time >= 900000){
          clearInterval(timer.timer);
          timer.run =0;
          this.audio.play();
        }
        timer._2time = Date.now()-timer._2Start;
      },1000)

      if(timer.run === 0){
        clearInterval(timer.timer)
      }

    },
    reset(timer) {
      clearInterval(timer.timer);
      timer.time = 0;
      timer.section = 0;
      timer.run = 0;
      timer._2time = 0;
      timer._2Start = 0;
      timer.start = 0;
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
.btn-circle.btn-md {
  width: 40px;
  height: 40px;
  padding: 0px 0px;
  border-radius: 20px;
  font-size: 14px;
  text-align: center;
}
.alert {
  font-size: 70%;
}
#right-panel-link {
  position: absolute;
  top: 0;
  right: 0;
}
</style>
