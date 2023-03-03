<template>
  <main>
    <div class="countdown">
      <span class="active">3</span>
      <span>2</span>
      <span>1</span>
      <span>Start</span>
    </div>
    <div class="logo">
      Typathon
    </div>
    <div class="container">
      <div class="tip">Type as fast as you can!</div>
      <div class="timer">
        <span class="visual"></span>
        <div class="time-left">
          Time Left
        </div>
        <div class="time">
          <span class="seconds">60</span>
        </div>
        <span>Seconds</span>
      </div>
      <span v-show="showTest">Your Text:</span>
      <p v-show="showTest" class="data">Another productive way to use this tool to begin a daily writing routine. One way is to generate a random paragraph with the intention to try to rewrite it while still keeping the original meaning.</p>
      <form v-show="showTest" action="">
        <input @input="comparisonMethod()" type="text" class="user-input">
        <button v-show="showStartBtn" @click.prevent="start(),showStartBtn = false, showRestartBtn = true">Start! <span></span><span></span><span></span><span></span> </button>
        <button v-show="showRestartBtn" @click.prevent="reset()">Restart<span></span><span></span><span></span><span></span> </button>
      </form>
      <div v-show="showResults" class="results">
        <div class="speed"><h2>Speed</h2> <p>{{(((wordsCount)/userTime) * 60).toFixed(1)}}<span>words/min</span></p></div>
        <div class="percentage"><h2>Correct %</h2> <p>{{ correctLettersPercentage }}%<span>correct words</span></p></div>
        <div class="user-time"><h2>Your Time</h2> <p>{{userTime}}s<span>seconds</span></p></div>
        <button @click.prevent="showResults = false, showTest = true,reset()">Again? <span></span><span></span><span></span><span></span> </button>
      </div>
    </div>
  </main>
</template>

<script>
import { onMounted, onUpdated } from 'vue';

import axios from "axios";
axios.get("http://metaphorpsum.com/sentences/3")
  .then((response) => {
    console.log(response.data)
    document.querySelector(".data").textContent = response.data;                    
    let data = document.querySelector(".data").textContent;
      document.querySelector(".data").textContent = ''
      data.split("").forEach((letter) => {
        let span = document.createElement("span")
        span.textContent = letter;
        document.querySelector(".data").append(span)
        // word.style.cssText = `color: red`
      })
      console.log(document.querySelectorAll(".data span"))
  })
  .catch((error) => {
    console.log(error)
    this.errorMsg = "ERROR FETCHING DATA"
  })
export default {
  name: 'App',
  data(){
    return{
      testParagraph : '',
      correctLetters: 0,
      correctLettersPercentage: 0,
      inputLength : null,
      prevInputLength: null,
      wordsCount: 0,
      userTime: 0,
      stopUserTimeCount: false,
      stopTimer: false,
      showResults: false,
      showTest: true,
      showStartBtn: true,
      showRestartBtn: false,
      countdownFinished:false
    }
  },
  setup(){
    onMounted(() => {
      let data = document.querySelector(".data").textContent;
      document.querySelector(".data").textContent = ''
      data.split("").forEach((letter) => {
        let span = document.createElement("span")
        span.textContent = letter;
        document.querySelector(".data").append(span)
        // word.style.cssText = `color: red`
      })
      console.log(document.querySelectorAll(".data span"))
    })
  },
  methods:{
    focusInput(){

      document.querySelector(".user-input").focus();
    },
    comparisonMethod(){
      let userInputArray = document.querySelector(".user-input").value.split("");
      userInputArray = userInputArray.filter((ele) => {
        return ele !== null
      })
      this.inputLength = userInputArray.length;
      if(this.inputLength < this.prevInputLength){
        console.log("des")
        if(document.querySelectorAll(".data span")[userInputArray.length].textContent === " "){
          this.wordsCount--;
        }
        if(document.querySelectorAll(".data span")[userInputArray.length].classList.contains("correct")){
          this.correctLetters--
          console.log("class correct")
        }else if( document.querySelectorAll(".data span")[userInputArray.length].classList.contains("wrong")){
          console.log("class wrong")
        }
      }else{
        console.log("acs")
        if(userInputArray[userInputArray.length -1].toLowerCase().trim() === document.querySelectorAll(".data span")[userInputArray.length -1].textContent.toLowerCase().trim()){
            document.querySelectorAll(".data span")[userInputArray.length - 1].classList.add("correct")
            document.querySelectorAll(".data span")[userInputArray.length - 1].classList.remove("wrong")
            this.correctLetters++
            console.log("Correct", this.correctLetters)
            document.querySelectorAll(".data span")[userInputArray.length -1].style.backgroundColor = "yellowgreen";
          }else{
            console.log("wrong")
            document.querySelectorAll(".data span")[userInputArray.length - 1].classList.remove("correct")
            document.querySelectorAll(".data span")[userInputArray.length - 1].classList.add("wrong")
            document.querySelectorAll(".data span")[userInputArray.length -1].style.backgroundColor = "red";
          }
          if(document.querySelectorAll(".data span")[userInputArray.length - 1].textContent === " "){
            this.wordsCount++;
            console.log(this.wordsCount)
          }
      }
      let totalNoOfLetters = document.querySelectorAll(".data span").length;
      // document.querySelectorAll(".data span")[userInputArray.length -1].style.cssText = "color: black"
      
      document.querySelectorAll(".data span").forEach((span) => {
          span.style.color = "#333"
        })
        for(let i = 0; i <= userInputArray.length -1;i++){
          document.querySelectorAll(".data span")[i].style.color = "black"
          
        }
        this.correctLettersPercentage = (this.correctLetters * 100 / userInputArray.length).toFixed(1)
        console.log(this.correctLettersPercentage+"%")
        this.prevInputLength = userInputArray.length;
        if(userInputArray.length === document.querySelectorAll(".data span").length){
          this.stopUserTimeCount = true;
          this.wordsCount++
          this.showResults = true
          this.showTest = false
          this.stopTimer = true
          document.querySelector(".user-input").blur()
        }
        console.log(document.querySelector(".user-input").value)
        document.querySelector(".speed").style.cssText = `
          // border: 5px solid rgb(${255 -(6.25  * this.wordsCount)},${this.wordsCount * 6.25},0);
          background-color: rgb(${255 -(3  * (((this.wordsCount)/this.userTime) * 60).toFixed(1))},${(((this.wordsCount)/this.userTime) * 60).toFixed(1) * 6.25},0);
        `
        document.querySelector(".percentage").style.cssText = `
          // border: 5px solid rgb(${255 -(2.55  * this.correctLettersPercentage)},${this.correctLettersPercentage * 2.55},0);
          background-color: rgb(${255 -(2.55  * this.correctLettersPercentage)},${this.correctLettersPercentage * 2.55},0);
        `
    },
    startTimer(){
        let fullTime = document.querySelector(".seconds").textContent; 
        let timerCounter = setInterval(() => {
          document.querySelector(".seconds").textContent--;
          document.querySelector(".visual").style.cssText = `
            height: ${document.querySelector(".seconds").textContent * 100/fullTime}%;
            background-color: rgb(${0 + (4.25 * this.userTime)}, ${255 - (4.25 * this.userTime)}, 0);
            `;
          // if(document.querySelector(".seconds").textContent == "1"){
          //   this.stopUserTimeCount = true
          // }
          if(this.stopTimer){
            clearInterval(timerCounter)
          }
          if(document.querySelector(".seconds").textContent == "0"){
            clearInterval(timerCounter)
            this.stopUserTimeCount = true
            this.showResults = true
            this.showTest = false
            document.querySelector(".user-input").blur()
          }
        }, 1000);
    },
    startUserTime(){
      let userTimeCount = setInterval(() => {
        if(this.stopUserTimeCount){
          clearInterval(userTimeCount)
        }
        this.userTime++
        console.log(this.userTime)
        console.log(this.userTime/60)
      }, 1000);
    },
    start(){
      console.log("start")
      this.stopUserTimeCount = false;
      this.stopTimer = false;
      this.correctLetters = 0;
      this.correctLettersPercentage = 0;
      this.wordsCount = 0;
      this.userTime = 0;
      document.querySelector(".seconds").textContent = 60;
      this.countdown();
    },
    reset(){
      console.log("reset")
      this.correctLetters = 0;
      this.correctLettersPercentage = 0;
      this.wordsCount = 0;
      this.userTime = 0;
      this.stopUserTimeCount = true;
      this.stopTimer = true;
      this.showStartBtn = true;
      this.showRestartBtn = false;
      this.prevInputLength = 0;
      setTimeout(() => {
        document.querySelector(".seconds").textContent = 60
      },1000)
      document.querySelectorAll(".data span").forEach((span) => {
        span.style.backgroundColor = "transparent";
        span.style.color = "#333";
      })
      document.querySelector(".user-input").value = null;
      document.querySelector(".visual").style.cssText = `
          height: 100%;
          // background-color: rgb(${0 + (4.25 * this.userTime)}, ${255 - (4.25 * this.userTime)}, 0);
      `;
      this.getRandomParagraph();
      // window.location.reload()
    },
    countdown(){
      document.querySelector(".countdown").style.cssText = `
        opacity:1;
        z-index: 4;
      `;
      let i = 0;
      let countdown = setInterval(() => {
        document.querySelectorAll(".countdown span").forEach((span) =>{
          span.classList.remove("active")
        })
        document.querySelectorAll(".countdown span")[i].classList.add("active")
        i++;
        if(i === document.querySelectorAll(".countdown span").length - 1){
          setTimeout(() => {
            this.countdownFinished = true;
            this.focusInput();
            this.startTimer();
            this.startUserTime();
            document.querySelector(".countdown").style.cssText = `
              opacity:0;
              z-index: -1;
            `;
            clearInterval(countdown)
          }, 2000);
        }
      }, 1000);
    },
    getRandomParagraph(){
      axios.get("http://metaphorpsum.com/sentences/3")
      .then((response) => {
        console.log(response.data)
        document.querySelector(".data").textContent = response.data;                    
        this.setParagraphSpans()
      })
      .catch((error) => {
        console.log(error)
        this.errorMsg = "ERROR FETCHING DATA"
      })
    },
    setParagraphSpans(){
      let data = document.querySelector(".data").textContent;
      document.querySelector(".data").textContent = ''
      data.split("").forEach((letter) => {
        let span = document.createElement("span")
        span.textContent = letter;
        document.querySelector(".data").append(span)
        // word.style.cssText = `color: red`
      })
      console.log(document.querySelectorAll(".data span"))
    }
  },
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&family=Space+Mono:wght@400;700&display=swap');
input{
  /* visibility: hidden; */
  opacity: 0;
}
*{
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
  box-sizing: border-box;
  padding: 0;
  margin: 0;
}
:root{
  --main-color: #1F0640;
  --sec-color:#461E5C;
  --third-color: rgb(0, 144, 108);
  --main-transition: all 0.2s 0s linear;
  --main-gradient: linear-gradient(45deg,var(--main-color) 10%, var(--sec-color) 100%);
  --sec-gradient: linear-gradient(67deg,var(--third-color) 10%, var(--main-color) 37%);
}
body{
  background-image: var(--main-gradient);
  background-size: cover;
  background-repeat: no-repeat;
  background-position: center;
  min-height: 100vh;
  color: white;
  padding-bottom: 30px;
  font-family: 'Press Start 2P', cursive;
}
.container{
  padding-left: 20px;
  padding-right: 20px;
  margin-left: auto;
  margin-right: auto;
}
.logo{
  text-align: center;
  font-size: 30px;
  font-weight: bold;
  padding-top: 30px;
  padding-bottom: 30px;
}
@media (min-width:767px) {
  .container{
    width: 750px;
  }
}
@media (min-width:992px) {
  .container{
    width: 980px;
  }
}
@media (min-width:1200px) {
  .container{
    width: 1180px;
  }
}
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
}
main{
  position: relative;
}
main .countdown{
  position: absolute;
  left: 0%;
  top: 0%;
  z-index: -1;
  font-size: 80px;
  display: flex;
  justify-content: center;
  align-items: center;
  opacity:0;
  transition:var(--main-transition);
  height: 100vh;
  width: 100vw;
  background-color: #00000076;
}
main .countdown span{
  position: absolute;
  left: 50%;
  top: 50%;
  display: flex;
  border-radius: 50%;
  justify-content: center;
  align-items: center;
  /* background-image: var(--sec-gradient); */
  /* border: 5px solid var(--sec-color); */
  width: 150px;
  height: 150px;
  transform: translate(-50%,-50%);
  background-color: red;
  opacity: 0;
  transition: var(--main-transition);
}
main .countdown span:last-child{
  color:white;
  font-size: 50px;
  background-color: green;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background-image: none;
}
main .countdown .active{
  opacity: 1;
}
.container .timer{
  width: 150px;
  height: 150px;
  border-radius: 50%;
  margin: 30px auto;
  border: 5px solid var(--sec-color);
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  position: relative;
  overflow: hidden;
  font-size: 12px;
}
.container .timer .time{
  font-size: 30px;
  font-weight: bold;
  position: relative;
  transition: var(--main-transition);
}
.container .timer .time-left{
  position: absolute;
  left: 50%;
  top: 20px;
  transform: translateX(-50%);
}
.container .timer > span:not(.visual){
  position: absolute;
  left: 50%;
  bottom: 30px;
  transform: translateX(-50%);
}
.container .timer > span.visual{
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: var(--third-color);
  transition: var(--main-transition);
}
.container > p {
  margin: 20px auto;
  border: 5px solid var(--third-color);
  border-radius: 0px 20px 20px 20px;
  padding: 10px 20px;
  width: 700px;
  max-width: 100%;
  min-height: 200px;
  text-align: left;
  font-size: 18px;
  color: #333;
  font-family: 'Space Mono', monospace;
  background-color: var(--third-color);
  position: relative;
}
.container form{
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
.container .results{
  display: grid;
  grid-template-columns: repeat(3,150px);
  background-color: transparent;
  background-image: var(--sec-gradient);
  color: white;
  justify-content: center;
  gap: 20px;
  padding: 20px;
  border-radius: 6px;
  /* border: 5px solid var(--third-color); */
  position: relative;
  height: 300px;
}
.container .results::before{
  content: '';
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%,-50%);
  background-image: linear-gradient(293deg,var(--third-color) 10%, var(--main-color) 86%);;
  width: calc(100% + 10px);
  height: calc(100% + 10px);
  border-radius: 6px;
  z-index: -2;
}
.container .results > div{
  height: 150px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  border: 5px solid var(--third-color);
  position: relative;
  z-index: 2;
}
.container .results > div::before{
  content: '';
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%,-50%);
  /* background-image: linear-gradient(293deg,var(--third-color) 10%, var(--main-color) 80%);; */
  width: calc(100% + 10px);
  height: calc(100% + 10px);
  border-radius: 50%;
  z-index: 1;
}
.container .results > div h2{
  font-size: 12px;
  position: absolute;
  left: 50%;
  top: 20px;
  z-index: 2;
  width: 100px;
  /* max-width: 100%; */
  transform: translateX(-50%);
  line-height: 1.5;
}
.container .results > div p{
  font-size: 24px;
  font-weight: bold;
  z-index: 2;
  position: relative;
}
.container .results > div span{
  width: 100px;
  z-index: 2;
  /* max-width: 100%; */
  font-size: 12px;
  font-weight: normal;
  position: absolute;
  left: 50%;
  bottom: -20px;
  transform: translateX(-50%);
}
.container .results > div.percentage span{
  bottom: -30px;
}
.container .results button{
  grid-column-start: 2;
  grid-column-end: 3;
  cursor: pointer;
  margin-right: auto;
  margin-left: auto;
  align-self: center;
  /* width: 100px;
  height: 100px;
  border-radius: 50%;
  color: white;
  font-size: 22px;
  transition: var(--main-transition);
  background-image: linear-gradient(293deg,var(--third-color) 10%, var(--main-color) 86%);
  background-size: 200%; */
}
.container .results button:hover{
  background-size: 100%;
}
@media (max-width:600px) {
  .container .results{
    grid-template-columns: repeat(2,150px);
    height: 400px;
  }
}
.container button{
  width: 85px;
  height: 85px;
  border-radius: 50%;
  font-size: 18px;
  cursor: pointer;
  background-color: transparent;
  color: var(--third-color);
  font-weight: bold;
  border: 3px solid var(--third-color);
  transition:var(--main-transition);
  display: flex;
  align-items: center;
  justify-content: center;
  padding-right: 20px;
  font-family: 'Space Mono', monospace;
  padding-left: 20px;
  position: relative;
}
.container button span{
  position: absolute;
  width: 5px;
  height: 5px;
  transition: all 0.2s 0.1s linear, width 0.1s 0s linear, border-raduis 0.1s 0s linear;
  border-radius: 50%;
  background-color: var(--third-color);
  transition-duration: 0.1s;
 
  transform: rotate(45deg); 
}
.container button:hover{
  width: 95px;
  height: 95px;
  background-color: var(--third-color);
  color: var(--main-color);
  box-shadow: 0px 0px 10px 10px var(--sec-color);
}
.container button:hover span{
  width: 20px;
  border-radius: 5px;
}
.container button span:nth-of-type(1){
  left: -15px;
  top: -15px;
}
.container button:hover span:nth-of-type(1){
  left: 20px;
  top: 20px;
}
.container button span:nth-of-type(2){
  left: -15px;
  bottom: -15px;
  transform: rotate(135deg); 
}
.container button:hover span:nth-of-type(2){
  left: 20px;
  bottom: 20px;
}
.container button span:nth-of-type(3){
  right: -15px;
  top: -15px;
  transform: rotate(135deg); 
}
.container button:hover span:nth-of-type(3){
  right: 20px;
  top: 20px;
}
.container button span:nth-of-type(4){
  right: -15px;
  bottom: -15px;
}
.container button:hover span:nth-of-type(4){
  right: 20px;
  bottom: 20px;
}
</style>
