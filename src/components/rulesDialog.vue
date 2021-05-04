<template>
  <v-dialog v-model="showDialog" width="650">
      <template v-slot:activator="{ on, attrs }">
        <v-btn fab dark small color="primary" v-bind="attrs" v-on="on">
            <v-icon dark>
               mdi-help
            </v-icon>
        </v-btn>
      </template>
      <v-card>
        <v-card-title class="headline grey lighten-2">
          Rules
        </v-card-title>
        <v-card-text class='px-4 py-2 subtitle-1'>
            <p>First, choose the number of dice, the bias level and click on <v-btn small style='width:100px' color='primary'>Set up</v-btn> to start.</p>
            <p>You can then <v-btn small style='width:80px' color='primary'>Roll</v-btn> the dice as many times as you want.</p>
            <p>For every roll, getting 6 is considered a success. You can choose to keep track of the number of success or the average result.</p>
            <p>Some dice will be biased (probability of success will be higher or lower) and others won't. The aim is to guess which dice are biased and wich aren't...</p>
            <div class='text-center text-h6'>The probability of success with a balanced die is 1÷6 &nbsp; &#8776; &nbsp;<span class='text-h5 font-weight-bold'>16.7%</span></div>
            <div class='text-center text-h6'>The average result with a balanced die is <span class='text-h5 font-weight-bold'>3.5</span></div>
        </v-card-text>
        <v-divider></v-divider>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" text @click="showDialog = false" >
            Got it !
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
</template>

<script>
export default {
    data() {
        return {
            showDialog: true
        }
    }
}
</script>
