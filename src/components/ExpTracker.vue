<template>
<div class="backscreen">
  <div class="control">
    <input
      class="labelInput"
      type="text"
      placeholder="username#1234"
      @create="InitBar"
      @click="ResetInput"
      @keyup.enter="GetMembershipData"
      @keyup="LiveSearch"
      v-model="username"
    />
    <button class="buttonInput" @click="GetMembershipData">
      <img src="brain.png">
    </button>
  </div>

  <br />
  <div class="ackzent">
  <div class="infoblock">
    <div class="infolabels">
      <ul class="list">
        <li>
          <label class="infolabel" id="progressToNextLevel">Towards next Level:  </label>
          <label class="valuelabel" id="progressToNextLevel" style="text-align:right">{{ expData.progressToNextLevel }}</label>
        </li>
        <li>
          <label class="infolabel" id="currentProgress">currentProgress:</label>
          <label class="valuelabel" id="currentProgress" style="text-align:right">{{ expData.currentProgress }}</label>
          </li>
        <li>
          <label class="infolabel" id="weeklyProgress">weeklyProgess:</label>
          <label class="valuelabel" id="weeklyProgress">{{ expData.weeklyProgress }}</label>
          </li>
        <li>
          <label class="infolabel" id="dailyProgress">dailyProgress:</label>
          <label class="valuelabel" id="dailyProgress"> {{ expData.dailyProgress }}</label>
          </li>
        <li>
          <label class="infolabel" id="nextLevelAt">nextLevelAt: </label>
          <label class="valuelabel" id="nextLevelAt"> {{ expData.nextLevelAt }}</label>
          </li>
        <li>
          <label class="infolabel" id="SeasonRank">Season Pass Level:  </label>
          <label class="valuelabel" id="SeasonRank">{{ expData.currentSeasonPassLevel }}</label>
          </li>
      </ul>
    </div>
    
    <div class="progressBars">
      <div id="seasonalRankBar" data-type="fill" data-fill-background-extrude="0" class="ldBar label-center" data-img="icon.png"><p class="infotext">Seasonal Rank</p></div>
      <div id="powerBonusBar" data-type="fill" data-fill-background-extrude="0" class="ldBar label-center" data-img="icon.png"><p class="infotext">Power Bonus</p></div>
    </div>
  </div>

  <div class="functionality" v-if="isVisible"> 
    <div class="autoRefresher">
      <span class="toggle-wrapper" role="checkbox" :aria-checked="autoRefresh.toString()" tabindex="0" @click="toggle" @keydown.space.prevent="toggle">
        <span class="toggle-background" :class="backgroundStyles" />
        <span class="toggle-indicator" :style="indicatorStyles" />
      </span>
      <p>Auto Refresh</p>
    </div>
    <!--div class="loader" v-if="messurementRunning"><label>I'm a loader</label></div>
    <div class="loader" v-if="autoRefresh"></div-->
    <div class="lds-ring" v-if="messurementRunning">EXPMessurement<div></div><div></div><div></div><div></div></div>
    <div class="lds-ring" v-if="autoRefresh">autoRefresh<div></div><div></div><div></div><div></div></div>
    <div class="Messurement">
      <button id="startMessung" @click="ExpMessurement">{{ messureStatus }}</button>
      <label>{{ expPerHour }}</label>
    </div>

  </div>
  </div>
</div>
</template>

<script>
import "../loading-bar.js";
import $ from "../jquery-3.6.0.min.js";
export default {
  name: "ExpTracker",
  data: function () {
    return {
      username: "",
      membershipId: "",
      membershipType: "",
      apiKey: "338fc46b3cf140b79315df6993eb2d7e",
      messureStatus: "Start Tracker",
      messurementRunning: false,
      timer: 0,
      startExp: 0,
      expPerHour: 0,
      expData: {
        progressToNextLevel: 0,
        currentProgress: 0,
        weeklyProgress: 0,
        dailyProgress: 0,
        nextLevelAt: 0,
        currentPercentage: 0,
        currentPercentageSP: 0,
        currentSeasonPassLevel: 0,
      },
      autoRefresh: false,
      isVisible: false,
    };
  },
  computed: {
    backgroundStyles() {
      return {
        "gold-mid": this.autoRefresh,
        "gray-lighter": !this.autoRefresh,
      };
    },
    indicatorStyles() {
      return {
        transform: this.autoRefresh ? "translateX(14px)" : "translateX(0)",
      };
    },
  },
  methods: {
    toggle() {
      this.autoRefresh = !this.autoRefresh;
      if (this.autoRefresh) {
        window.aR = setInterval(this.GetExpData, 3000);
      } else {
        clearInterval(window.aR);
      }
    },

    ResetInput: function () {
      this.username = "";
    },

    InitBar: function () {
      var pbBar = document.getElementById("powerBonusBar").ldBar;
      var seasonalRankBar = document.getElementById("seasonalRankBar").ldBar;
      if (this.expData.currentPercentage != null) {
        pbBar.set(this.expData.currentPercentage);
      } else {
        pbBar.set(0);
      }

      if (this.expData.currentPercentageSP != null) {
        seasonalRankBar.set(this.expData.currentPercentageSP);
      } else {
        seasonalRankBar.set(0);
      }
    },
    //
    FillExpData: function (expArray) {
      for (var key in expArray) {
        if (Object.prototype.hasOwnProperty.call(expArray, key)) {
          console.log(key + " -> " + expArray[key]);
          if (Object.prototype.hasOwnProperty.call(this.expData, key)) {
            this.expData[key] = expArray[key];
          }
        }
      }
    },

    //
    GetExpData: function () {
      var ref = this;
      $.ajax({
        url:
          "https://www.bungie.net/Platform/Destiny2/" +
          this.membershipType +
          "/Profile/" +
          this.membershipId +
          "/?components=104%2C202%2C1100%2C100",
        headers: {
          "X-API-Key": this.apiKey,
        },
        type: "GET", // added data type
        contentType: "application/json",
        dataType: "json",
        success: function (expRes) {
          console.log(expRes);
          //1787069365 -> Prestige 
          //2069932355 -> SP bis 100
          var profileProgression = expRes.Response.profileProgression.data;
          ref.FillExpData(profileProgression.seasonalArtifact.powerBonusProgression);
          var expData = ref.expData;
          var characterProgression = expRes.Response.characterProgressions.data[Object.keys(expRes.Response.characterProgressions.data)[0]];
          var seasonPassLevel = characterProgression.progressions['2069932355'].level;

          if(seasonPassLevel >= 100)
          {
            seasonPassLevel += characterProgression.progressions['1787069365'].level;
            expData.currentPercentageSP =  characterProgression.progressions['1787069365'].progressToNextLevel / 1000;
          }
          else
          {
            expData.currentPercentageSP =  characterProgression.progressions['2069932355'].progressToNextLevel / 1000;
          }

          expData.currentSeasonPassLevel = seasonPassLevel;
         
          expData.currentPercentage = (expData.progressToNextLevel / expData.nextLevelAt) * 100;
          ref.isVisible = true;
          ref.InitBar();
        },
      });
    },
    //get general Memebership Data and store it, call GetExpData
    GetMembershipData: function () {
      var userIdEscaped = this.username.replace("#", "%23");
      var ref = this;
      $.ajax({
        url:
          "https://www.bungie.net/Platform/Destiny2/SearchDestinyPlayer/All/" +
          userIdEscaped +
          "/",
        headers: {
          "X-API-Key": this.apiKey,
        },
        type: "GET", // added data type
        contentType: "application/json",
        dataType: "json",
        success: function (accRes) {
          ref.membershipId = accRes.Response[0].membershipId;
          ref.membershipType = accRes.Response[0].membershipType;
          ref.GetExpData();
        },
      });
    },
    //
    ExpMessurement: function () {
      var ref = this;
      if (!this.messurementRunning) {
        this.messurementRunning = true;
        this.messureStatus = "Stop Tracker";
        console.log(this.messurementRunning);
        window.timer = 0;
        this.GetExpData();
        this.startExp = this.expData.currentProgress;
        window.eM = setInterval(function () {
          window.timer = window.timer + 100;
          if(window.timer % 10000 == 0){
            ref.GetExpData();
          }
        }, 100);
      } else {
        clearInterval(window.eM);
        this.GetExpData();
        this.messurementRunning = false;
        setTimeout(function(){
          console.log("gathering Data");
          ref.messureStatus = "Start Tracker";
          var endExp = ref.expData.currentProgress;
          var hour = window.timer / 3600000;
          var expHr = (endExp - ref.startExp) / hour;
          window.timer = 0;
          ref.startExp = 0;
          console.log(expHr); 
        }, 5000);
               
        //Ausgabe Exp/hr
      }
    },
    LiveSearch: function(){
       $.ajax({
        url:
          "https://www.bungie.net/Platform/User/Search/GlobalName/0/",
        headers: {
          "X-API-Key": this.apiKey,
        },
        type: "POST", // added data type
        contentType: "application/json",
        dataType: "json",
         data: JSON.stringify({ "displayNamePrefix": this.username }),
        success: function (response) {
          var res = response.Response.searchResults
          var counter = 0;
          for (var key in res) {
            if(counter <=5){
              console.log(res[key].bungieGlobalDisplayName + '#' + res[key].bungieGlobalDisplayNameCode);
              counter++;
            }           
          }
        },
      });
    },
  },
};
</script>

<style>

/*/////////Toggles///////////*/
.gold-mid {
  background-color: #5dce59;
}

.gray-lighter {
  background-color: #949494;
}

.toggle-wrapper {
  display: inline-block;
  position: relative;
  cursor: pointer;
  width: 32px;
  height: 18px;
  border-radius: 9999px;
}

.toggle-wrapper:focus {
  outline: 0;
}

.toggle-background {
  display: inline-block;
  border-radius: 9999px;
  height: 100%;
  width: 100%;
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.1);
  transition: background-color 0.4s ease;
}

.toggle-indicator {
  position: absolute;
  height: 14px;
  width: 14px;
  left: 2px;
  bottom: 2px;
  background-color: #ffffff;
  border-radius: 9999px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  transition: transform 0.4s ease;
}

/*////////////////////////*/

.backscreen {
  background-color: #383838;
}

.ackzent{
   margin: 3%;
  padding: 1%;
  width:92%;
  height:80vh;
  background-color: #505050;
  border-style: hidden;
  border-radius: 3pt;
  font-family: "Open Sans", sans-serif;
}

.infoblock{
  display: block;
}



.list{
  padding-left: 0%;
  list-style: none;
  font-size: 1.5rem;
  color: white;
}

.infoblock .infolabels .valuelabel{
  float: right;
}

.functionality{
  margin-top: 25%;
    align-items: center;
  display: flex;
}

.functionality .autoRefresher{
   margin: auto;
  display: inline-block;
  align-items: center;
}

.functionality .Messurement{
    margin: auto;
  display: inline-block;
  align-items: center;
}

.functionality .loader{
    margin: auto;
  display: inline-block;
  align-items: center;
}



@media only screen and (min-width: 800px) {

.ackzent{
   margin: 3%;
  padding: 1%;
  width:92%;
  height: 80vh;
  background-color: #505050;
  border-style: hidden;
  border-radius: 3pt;
  font-family: "Open Sans", sans-serif;
}

.infoblock {
  margin-left: 3%;
  padding: 1%;
  width:92%;
  height: 40%;
  background-color: #505050;
  border-style: hidden;
  border-radius: 3pt;
  font-family: "Open Sans", sans-serif;
  display: inline-block;
}

.list{
  list-style: none;
  font-size: 2rem;
  color: white;
}

.infovalues{
  float: left;
  font-size: 2rem;
}

.infoblock .infolabels{
  float: left;
  font-size: 2rem;
}

.functionality{
  margin: 3%;
  margin-top: 5%;
}

}

.progressBars {
  align-items: center;
  display: flex;
  font-size:1.5rem;
}

@media only screen and (max-width:800px){
.infoblock .progressBars{
  margin-top:10%;
}
}

.progressBars .ldBar {
  margin: auto;
  display: inline-block;
  align-items: center;
}

.progressBars .infotext{
  font-family: "Open Sans", sans-serif;
  font-size: 2rem;
  color: white;
  vertical-align: middle;
  text-align: center;
}

.control {
  margin-top: 0%;
  width: 100%;
  font-family: "Open Sans", sans-serif;
  display: flex;
  /*background-image: url("../assets/header.png");*/
}

.control .labelInput{
  border: none;
  padding: 0;
  margin-left: 3%;
  height: 100%;
  opacity: 0.5;
  width:100%;
  font-size: 2.7rem;
  padding-top:0.5%;
  padding-bottom: 0.5%;
}

.control .labelInput:focus {
  border: none;
  outline: none;
}

.control .buttonInput{
  border: none;
  opacity: 0.5;
  margin-right:3%;
  float: right;
}

.control .buttonInput:hover {
  opacity: 0.8;
}


.ldBar {
  position: relative;
}

.ldBar.label-center > .ldBar-label {
  position: absolute;
  top: 50%;
  left: 50%;
  -webkit-transform: translate(-50%, -50%);
  transform: translate(-50%, -50%);
  text-shadow: 0 0 3px #fff;
}

.ldBar-label:after {
  content: "%";
  display: inline;
}

.ldBar.no-percent .ldBar-label:after {
  content: "";
}

.loader {
  border: 16px solid #f3f3f3;
  border-radius: 50%;
  border-top: 16px solid #2adf33;
  width: 80px;
  height: 80px;
  -webkit-animation: spin 2s linear infinite; /* Safari */
  animation: spin 2s linear infinite;
}

/* Safari */
@-webkit-keyframes spin {
  0% { -webkit-transform: rotate(0deg); }
  100% { -webkit-transform: rotate(360deg); }
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}


.lds-ring {
  display: inline-block;
  position: relative;
  width: 80px;
  height: 80px;
}
.lds-ring div {
  box-sizing: border-box;
  display: block;
  position: absolute;
  width: 64px;
  height: 64px;
  margin: 8px;
  border: 8px solid #dfc;
  border-radius: 50%;
  animation: lds-ring 1.2s cubic-bezier(0.5, 0, 0.5, 1) infinite;
  border-color: #dfc transparent transparent transparent;
}
.lds-ring div:nth-child(1) {
  animation-delay: -0.45s;
}
.lds-ring div:nth-child(2) {
  animation-delay: -0.3s;
}
.lds-ring div:nth-child(3) {
  animation-delay: -0.15s;
}
@keyframes lds-ring {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}


</style>
