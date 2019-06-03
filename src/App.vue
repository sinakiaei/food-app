<template>
  <v-app>
    <v-form>
      <v-container>
        <v-layout>
          <v-flex xs6 md4>
            <v-select :items="genders" v-model="selectedGender" label="Gender" solo></v-select>
          </v-flex>
          <v-flex xs6 md4>
            <v-select :items="ages" v-model="selectedAge" label="Age" solo></v-select>
          </v-flex>
          <v-flex xs6 md4>
            <v-btn
              color="primary"
              @click="submit"
              :disabled="selectedAge === null || selectedGender === null"
            >Recommend</v-btn>
          </v-flex>
        </v-layout>
      </v-container>
    </v-form>
    <v-container>
      <v-layout row wrap>
        <v-flex xs6>
          <template v-for="(items, key, index) in recommendations">
            <v-card :key="index" color="blue-grey darken-2" class="white--text my-2">
              <v-card-title primary-title>
                <div>
                  <div class="headline">{{ key }}</div>
                  <template v-for="(item, idx) in items">
                    <div :key="idx" class="my-2">
                      <div class="subheading">{{ item.food }}</div>
                      <div class="caption">{{ item.srvg_sz }}</div>
                    </div>
                  </template>
                </div>
              </v-card-title>
              <v-divider light></v-divider>
              <v-card-actions>
                <v-btn flat dark @click.stop="getDirections(items[0].fgid)">Directions</v-btn>
              </v-card-actions>
            </v-card>
          </template>
        </v-flex>
      </v-layout>
      <v-layout row justify-center>
        <v-dialog v-model="dialog" max-width="400">
          <v-card>
            <v-card-title class="headline grey lighten-2" primary-title>Directions</v-card-title>
            <v-card-text>
              <div v-for="(item, index) in direction" :key="index">
                <p>{{ index + 1 }}. {{item}}</p>
              </div>
            </v-card-text>
            <v-divider></v-divider>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="primary" flat @click="dialog = false">OK</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-layout>
    </v-container>
  </v-app>
</template>

<script>
import axios from "axios";
import groupBy from "lodash/groupBy";

export default {
  data() {
    return {
      ages: null,
      selectedAge: null,
      genders: null,
      selectedGender: null,
      recommendations: null,
      directions: null,
      direction: null,
      dialog: false
    };
  },
  methods: {
    submit() {
      axios
        .get(
          `http://localhost:3000/api/recommendations?gender=${encodeURIComponent(
            this.selectedGender
          )}&ages=${encodeURIComponent(this.selectedAge)}`
        )
        .then(response => {
          this.recommendations = groupBy(
            response.data.servings,
            serving => serving.foodgroup
          );
          this.directions = groupBy(
            response.data.directions,
            direction => direction.fgid
          );
        })
        .catch(error => {
          console.log(error);
        });
    },
    getDirections(foodGroup) {
      this.direction = this.directions[foodGroup][0].directional_statement;
      this.dialog = true;
    }
  },
  mounted() {
    axios.get(`http://localhost:3000/api/initial-values`).then(response => {
      this.genders = response.data.genders;
      this.ages = response.data.ages;
    });
  }
};
</script>
