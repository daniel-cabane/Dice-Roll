<template>
  <v-container style='max-width:960px;margin:auto'>
    <div style='position:absolute;right:5px'>
      <guessHistory :history='history'/>
      <rulesDialog/>
    </div>
    <v-row class="text-center mt-3">
      <v-col cols="4" style='display:flex;justify-content:center;align-items:center'>
        <v-select :disabled='started' hide-details :items="[1,2,3,4]" v-model='nbOfDice' label="Nb of dice" outlined></v-select>
      </v-col>
      <v-col cols="4" style='display:flex;justify-content:center;align-items:center'>
        <v-select :disabled='started' hide-details :items="biasList" v-model='bias' label="Maximum Bias" outlined></v-select>
      </v-col>
      <v-col cols="4" style='display:flex;justify-content:center;align-items:center'>
        <v-btn x-large color='error' style='width:80%' v-if='started' outlined @click='init'>Restart</v-btn>
        <v-btn :disabled='nbOfDice == null || bias == null' x-large color='primary' style='width:80%' v-else @click='setUp'>Set up</v-btn>
      </v-col>
      <v-col cols="9">
        <div class='mt-2 text-h4'>
          Total nb of tries : <b>{{ tries }}</b>
        </div>
        <div class='d-flex justify-space-around mb-10'>
          <div class='ma-1' v-for='(die, index) in dice' :key='index'>
            <singleDice :result='die.result' :rolling='rolling'/>
            <div class='grey--text'>Nb of success : {{ die.success }}</div>
            <div class='text-h6'>Success ratio :<br/><span class='text-h4'><b>{{ die.ratio }}%</b></span></div>
            <div style='position:relative' v-if='bias!=0'>
              <v-select 
                hide-details solo dense
                v-model='die.presumed'
                :disabled='revealed'
                :background-color='presumeColor(die.presumed)' 
                :dark='presumeColor(die.presumed)!="white"'
                :items='["Balanced", "Biased"]' 
                style='width:150px'
                class='mt-3'
              />
              <v-img width='40px' src="../assets/tick.png" style='position:absolute;bottom:5px;right:20px;' v-if='revealed && (die.presumed == "Balanced" && die.bias == 0 || die.presumed == "Biased" && die.bias != 0)'/>
              <v-img width='40px' src="../assets/cross.png" style='position:absolute;bottom:0px;right:20px;' v-else-if='revealed'/>
            </div>
            <v-card large color='black' style='width:150px;height:50px' class='mt-3 d-flex justify-center align-center text-h5' dark v-if='bias == 0'>Balanced</v-card>
            <div v-if='revealed'>
              <v-card large color='primary' style='width:150px;height:50px' class='mt-3 d-flex justify-center align-center text-h5' dark v-if='die.bias == 0'>Balanced</v-card>
              <v-card large color='error' style='width:150px;height:50px' class='mt-3 d-flex justify-center align-center text-h5' dark v-else>Biased</v-card>
            </div>
          </div>
        </div>
        <div v-if='dice.length && bias!=0'>
          <div class='text-center text-h4' v-if='revealed'>
            Guessed after <b>{{ guessedAfter }}</b> tries
          </div>
          <v-btn color='success' x-large style='width:100%' @click='revealBias' outlined v-else>Reveal</v-btn>
        </div>
      </v-col>
      <v-col cols="3" v-if='started'>
        <div class='mt-5 mb-8' style='width:100%;display:flex;height:20px'>
          <v-card v-if='rolling' flat tile color='primary' style=';height:20px' :style='`flex:${rollRatio}`'/>
          <v-card v-if='rolling' flat tile color='primary' style='opacity:0.3;height:20px' :style='`flex:${100-rollRatio}`'/>
        </div>
        <v-select hide-details class='mb-6' :items="[1,20,50,100,1000]" v-model='nbOfRolls' label="Nb of rolls" outlined></v-select>
        <v-btn :disabled='rolling' color='primary' large style='width:200px' @click='rollDice'>Roll</v-btn>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  import singleDice from './singleDice';
  import rulesDialog from './rulesDialog';
  import guessHistory from './guessHistory';


  export default {
    name: 'mainDeck',
    components: { singleDice, rulesDialog, guessHistory },
    data: () => ({
      dice: [],
      nbOfDice: null,
      biasList: [
        {text: 'None', value: 0},
        {text: 'Subtle (+2% or -2%)', value: 0.02},
        {text: 'Visible (+5% or -5%)', value: 0.05},
        {text: 'Obvious (+10% or -10%)', value: 0.1},
        {text: 'Blatant (+20% or -20%)', value: 0.2}
      ],
      bias: null,
      started: false,
      rolling: false,
      tries: 0,
      nbOfRolls: 1,
      rollsToDo: 0,
      roll: null,
      revealed: false,
      guessedAfter: 0,
      history: []
    }),
    computed:{
      rollRatio(){
        return 100*this.rollsToDo/this.nbOfRolls;
      }
    },
    methods: {
      setUp() {
        this.dice = [];
        for(let i=0; i < this.nbOfDice ; i++){
          this.dice.push({result: 1, bias: Math.random()<0.5 ? Math.sign(Math.random()-0.5)*this.bias : 0, success: 0, ratio: 0});
        }
        this.started = true;
      },
      init() {
        this.nbOfDice = 1;
        this.dice = [];
        this.bias = 0;
        this.tries = 0;
        this.started = false;
        clearInterval(this.roll);
        this.rolling = false;
        this.nbOfRolls = 1;
        this.revealed = false;
      },
      rollOnce(){
        this.tries++;
        this.dice.forEach(die => {
          let result = Math.random() < 0.16667+die.bias ? 6 : Math.round(Math.random()*4+1);
          die.result = result;
          if(result == 6){
            die.success = die.success + 1;
          }
          die.ratio = (100*die.success/this.tries).toFixed(1);
        });
      },
      rollDice() {
        if(this.nbOfRolls == 1) return this.rollOnce();
        this.rolling = true;
        let delay = Math.max(10, 100 - this.nbOfRolls);
        this.rollsToDo = this.nbOfRolls;
        this.roll = setInterval(()=> {
          this.rollOnce();
          this.rollsToDo--;
          if(this.rollsToDo <= 0){
            clearInterval(this.roll);
            this.rolling = false;
          }
        }, delay);
      },
      presumeColor(presumed){
        return presumed == 'Biased' ? 'black' : 'white';
      },
      revealBias(){
        let nbCorrectGuesses = this.dice.reduce((acc, die) => (die.bias == 0 && die.presumed == "Balanced") || (die.bias != 0 && die.presumed == "Biased") ? ++acc : acc, 0);
        this.guessedAfter = this.tries;
        let biasItem = this.biasList.filter(b => b.value == this.bias)[0];
        this.history.push({
          correct: nbCorrectGuesses,
          outOf: this.nbOfDice==1 ? '1 die' : `${this.nbOfDice} dice`,
          after: `${this.guessedAfter} rolls`,
          bias: biasItem.text
        });
        this.revealed = true;
      }
    }
  }
</script>
