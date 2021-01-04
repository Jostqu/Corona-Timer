<template>
  <div class="container-fluid no-padding .container">
    <button class="btn btn-light m-2" @click="addnewTimer">Add</button>
    <button class="btn btn-light mx-1m-1" @click="removeTimer()">Remove</button>
    <button class="btn btn-light mx-1 float-right m-2" @click="openNav()">Settings</button>
    <div ref="myNav" class="overlay">

      <!-- Button to close the overlay navigation -->
      <a class="closebtn" @click="closeNav()">&times;</a>

      <!-- Overlay content -->
      <div class="overlay-content">
      Phase 1 in Sekunden: <input v-model="Phase_1_Sek" type="number"><br><br>
      Phase 2 in Sekunden: <input v-model="Phase_2_Sek" type="number"><br><br>
      <button class="btn btn-light" @click="resetToDefaults">Reset</button>
      </div>

    </div>
    <div class="row no-gutters" v-bind:key="i" v-for="(row,i) in groupedTimers">
      <div v-for="(timer_col,index) in groupedTimers[i]" v-bind:key="index" class="col-sm nopadding">
        <button class="btn btn-danger float-right btn-circle btn-md mr-3"  @click="reset(timer_col)">Reset</button>
        <div class="card-body no-padding no-gutters">
        <h6 class="card-title no-padding"> Test {{i*per_collum+index+1}}</h6>
        <div v-if="timer_col.time >= Phase_1 && timer_col.section === 0" class="alert alert-danger no-padding .alert" role="alert">
          Testlösung auftragen!</div>
        <h7 v-if="timer_col.run === 1 && timer_col.section === 0 ">
          Tupfer in Testlösung</h7>
        <h7 v-if="timer_col.run === 1 && timer_col.section=== 1">
          Test läuft</h7>
        <div v-if="timer_col._2time >= Phase_2" class="alert alert-success .alert" role="alert">
        Ergebnis ablesen!</div>
        <div class="timer-form">
          <div class="progress">
            <div class="progress-bar pg-beginning" role="progressbar"  v-bind:style="{ width: timers[i*per_collum+index].time/1000*0.0980392156862742 + '%'}" v-bind:aria-valuenow="0" aria-valuemin="0" aria-valuemax="100"></div>
            <div class="progress-bar bg-warning" role="progressbar"  v-bind:style="{width: ((timers[i*per_collum+index]._2time/1000*0.0980392156862742)) + '%'}" v-bind:aria-valuenow="0" aria-valuemin="0" aria-valuemax="100"></div>
          </div>
          <div class="display:inline" v-if="timer_col.section === 0">{{formattedElapsedTime(timer_col.time)}}</div>
          <div class="display:inline" v-if="timer_col.section === 1">{{formattedElapsedTime(timer_col._2time)}}</div>
          <button class="btn  btn-success "  @click="start(timer_col)" v-if="timer_col.run !== 1 && timer_col.time < Phase_1">Start</button>
          <button class="btn  btn-success "  @click="weiter(timer_col)" v-if="timer_col.time >= Phase_1 && timer_col.run !== 1 && timer_col._2time === 0">Weiter</button>

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

      Phase_1: 120000,
      Phase_2: 900000,
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
    },
    Phase_1_Sek:{
      get: function(){
        return this.Phase_1/1000
      },
      set: function(val){
        let comp = val*1000
        this.Phase_1 = comp
      }
    },
    Phase_2_Sek:{
      get: function(){
        return this.Phase_2/1000
      },
      set: function(val){
        let comp = val*1000
        this.Phase_2 = comp
      }
    }
  },
  methods: {
    resetToDefaults(){
    this.Phase_1 = 120000;
    this.Phase_2 = 900000;
    },
    closeNav(){
      /* Close when someone clicks on the "x" symbol inside the overlay */
      this.$refs.myNav.style.width = "0%";
    },
    openNav(){
      /* Open when someone clicks on the span element */
      this.$refs.myNav.style.width = "100%";
    },
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
        if(timer.time >= this.Phase_1 && timer._2Start === 0){
          clearInterval(timer.timer);
          timer.run =0;
          this.audio.src = require('./test.mp3');
          this.audio.currentTime = 0;
          this.audio.play();
        }
      }, 1000);
    },
    weiter(timer){
      timer._2Start = Date.now()
      timer.section = 1;
      timer.run = 1;
      timer.timer = setInterval(() => {
        if(timer._2time >= this.Phase_2){
          clearInterval(timer.timer);
          timer.run =0;
          this.audio.src = require('./eventually-590.mp3');
          this.audio.currentTime = 0;
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
/* The Overlay (background) */
.overlay {
  /* Height & width depends on how you want to reveal the overlay  */
  height: 100%;
  width: 0;
  position: fixed; /* Stay in place */
  z-index: 1; /* Sit on top */
  left: 0;
  top: 0;
  background-color: white; /* Black fallback color */
  background-color: rgba(255,255,255, 0.8); /* Black w/opacity */
  overflow-x: hidden; /* Disable horizontal scroll */
  transition: 0.5s; /* 0.5 second transition effect to slide in or slide down the overlay (height or width, depending on reveal) */
}

/* Position the content inside the overlay */
.overlay-content {
  position: relative;
  top: 25%; /* 25% from the top */
  width: 100%; /* 100% width */
  text-align: center; /* Centered text/links */
  margin-top: 30px; /* 30px top margin to avoid conflict with the close button on smaller screens */
}

/* The navigation links inside the overlay */
.overlay a {
  padding: 8px;
  text-decoration: none;
  font-size: 36px;
  color: #818181;
  display: block; /* Display block instead of inline */
  transition: 0.3s; /* Transition effects on hover (color) */
}

/* When you mouse over the navigation links, change their color */
.overlay a:hover, .overlay a:focus {
  color: #f1f1f1;
}

/* Position the close button (top right corner) */
.overlay .closebtn {
  cursor:pointer;
  position: absolute;
  top: 20px;
  right: 45px;
  font-size: 60px;
}

/* When the height of the screen is less than 450 pixels, change the font-size of the links and position the close button again, so they don't overlap */
@media screen and (max-height: 450px) {
  .overlay a {font-size: 20px}
  .overlay .closebtn {
    font-size: 40px;
    top: 15px;
    right: 35px;
  }
}
</style>
