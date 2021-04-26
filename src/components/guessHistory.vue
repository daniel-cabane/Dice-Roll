<template>
  <v-dialog v-model="showDialog" width="650">
      <template v-slot:activator="{ on, attrs }">
        <v-btn class='mx-2' fab dark small color="primary" v-bind="attrs" v-on="on">
            <v-icon dark>
               mdi-format-list-bulleted
            </v-icon>
        </v-btn>
      </template>
      <v-card>
        <v-card-title class='text-h4'>
            Guess history
        </v-card-title>
        <v-card-text>
            <v-data-table :headers="headers" :items="history" :items-per-page="20"/>
        </v-card-text>
      </v-card>
    </v-dialog>
</template>

<script>
export default {
    props: { history: Array },
    data() {
        return {
            showDialog: false,
            headers: [
                { text: 'Correct guesses', value: 'correct', align: 'center', cellClass:'font-weight-bold text-h5' },
                { text: 'Out of', value: 'outOf' },
                { text: 'After', value: 'after' },
                { text: 'Bias level', value: 'bias' },
            ]
        }
    }
}
</script>
