<template>
  <div class="search">
    <h1 class="text-center" v-show="false">{{ msg }}</h1>

    <b-modal 
      id="modalCenter" 
      :title="p.name" 
      no-close-on-backdrop 
      ok-only 
      header-bg-variant="primary" 
      header-text-variant="light"
    >
      <b-container fluid class="text-capitalize">
        <b-row>
          <b-col cols="6" class="text-right">altura</b-col>
          <b-col cols="6">{{ p.height }}</b-col>
        </b-row>
        <b-row>
          <b-col cols="6" class="text-right">peso</b-col>
          <b-col cols="6">{{ p.mass }}</b-col>
        </b-row>
        <b-row>
          <b-col cols="6" class="text-right">año nacimiento</b-col>
          <b-col cols="6">{{ p.birth_year }}</b-col>
        </b-row>
        <b-row>
          <b-col cols="6" class="text-right">genero</b-col>
          <b-col cols="6">{{ p.gender }}</b-col>
        </b-row>
        <b-row>
          <b-col cols="6" class="text-right">planeta</b-col>
          <b-col cols="6">{{ p.planeta }}</b-col>
        </b-row>
        <b-row>
          <b-col cols="6" class="text-right">films</b-col>
          <b-col cols="6" v-show="toggles.showFilms">
            <ul>
              <li v-for="(z, index) in p.filmsName" :key="index">{{ z }}</li>
            </ul>                    
          </b-col>
        </b-row>
        <b-row>
          <b-col cols="6" v-for="(img, index) in p.imagenes" :key="index">
            <div class="card border-secondary">
              <img class="card-img" v-bind:src="img.thumbnailUrl" alt="Card image">
            </div>
          </b-col>
        </b-row>  
      </b-container>
    </b-modal>

    <b-overlay :show="toggles.loading" rounded="sm" variant="white">
      <b-container>
        <b-row class="sb">
          <b-col></b-col>
          <b-col cols="auto">
            <p>
              <b-input-group>
                <b-form-input type="text" v-model="name" placeholder="Nombre" @keyup="buscar()"></b-form-input>

                <b-input-group-append>
                  <b-button variant="outline-primary" @click="buscar()">Buscar</b-button>
                </b-input-group-append>
              </b-input-group>
            </p>
          </b-col>
          <b-col></b-col>
        </b-row>

        <ul v-if="errors && errors.length">
          <li v-for="(error, index) of errors" :key="index">
            {{error.message}}
          </li>
        </ul>

        <b-row>
          <b-col cols="3" class="cc" v-for="(x, index) in swp" :key="index">
            <b-card border-variant="primary" text-variant="black" :title="x.name">
              <b-card-text>
                
              </b-card-text>
              <b-button href="#" variant="primary" @click="getP(x)">Ir al detalle</b-button>
            </b-card>
          </b-col>
        </b-row>
      </b-container>
    </b-overlay>
    
    <p v-show="false">{{msg}}</p>

  </div>
</template>

<script lang="ts">
  import Vue from 'vue'
  import { ModalPlugin } from 'bootstrap-vue';
  import Component, { mixins } from 'vue-class-component';
  import axios from 'axios';

  Vue.use(ModalPlugin);

  interface Base{
    name:  string,
    height:  string,
    mass:  string,
    birth_year:  string,
    gender: string,
    planeta: string,
    imagenes: [
      {thumbnailUrl: string}
    ],
    filmsName: string[],
  }

  const SearchProps = Vue.extend({
    props: {
      msg: String
    }
  });
  @Component({})
  export default class Search extends mixins(SearchProps) {

    name = '';
    swp: string[] = [];
    p: Base = {
      name: '',
      height: '',
      mass: '',
      birth_year: '',
      gender: '',
      planeta: '',
      imagenes: [
        {thumbnailUrl: 'https://upload.wikimedia.org/wikipedia/commons/6/6c/Star_Wars_Logo.svg'}
      ],
      filmsName: [],
    };
    baseUrl = 'https://swapi.dev/api/';
    errors = [];
    toggles = {
      showFilms: false,
      loading: false,
    };

    mounted() {
      this.buscar();
    }

    public async buscar() {
      try {
        const response = await axios.get(this.baseUrl + 'people/?search=' + this.name);
        this.swp = response.data.results;
      } catch (error) {
        console.log("Error: buscar()" + error);
        //this.errors.push(error);
      }
    }

    public async getP(e: any){
      this.toggles.loading = true;

      this.p.name = e.name;
      this.p.height = e.height;
      this.p.mass = e.mass;
      this.p.birth_year = e.birth_year;
      this.p.gender = e.gender;
      this.addPlanetName(e, this.p);
      this.p.filmsName = await this.addFilmsName(e);
      //addImages($scope.p);
      // eslint-disable-next-line
      //$('#modalCenter').modal('show');
      this.toggles.loading = false;
      this.$bvModal.show('modalCenter');
    }

    public async addFilmsName(e: {films: string[]}) {
      let filmsNames = [];
      this.toggles.showFilms = false;

      try{
        const filmsSize =  e.films.length;
        for (let i = 0; i < filmsSize; i++) {
          const response = await axios.get(e.films[i]);
          filmsNames.push(response.data.title);

          if (filmsSize === i+1) {
            console.log(i);
            this.toggles.showFilms = true;
          }
        }
      } catch (error) {
        console.log("Error: addFilmsName()" + error);
        filmsNames.push('Nada');
        //this.errors.push(error);
      }

      return filmsNames;
    }

    public async addPlanetName(e: any, p: Base) {
      try {
        const response = await axios.get(e.homeworld);
        p.planeta = response.data.name;
      } catch (error) {
        console.log("Error: addPlanetName()" + error);
        p.planeta = 'Nada';
        //this.errors.push(error);
      }
    }
    
  };

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .sb {
    padding-top: 2em;
  }
  .cc {
    padding-bottom: 1em;
  }
</style>
