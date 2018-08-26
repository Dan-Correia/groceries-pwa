<template>
  <div>
    <h1>Week Meals</h1>
    <datalist id="ingredients">
      <option v-for="(ingredient, index) in ingredients" :key="index" :value="ingredient"/>
    </datalist>
    <b-card class="m-5">
      <div v-if="addingARecipe === false">
        <select class=" mr-3" v-model="selectedRecipe" @change="addToWeekRecipes(selectedRecipe)">
          <option disabled selected>Pick a recipe</option>
          <option v-for="(recipe, index) in recipes" v-bind:value="recipe" :key="index">
            {{ recipe.name }}
          </option>
        </select>
        OR
        <button class="btn btn-primary px-5 ml-3" @click="addingARecipe = true;addIngredientToRecipe()">Add recipe</button>
      </div>
      <div v-if="addingARecipe">
        <h1>Adding a recipe</h1>
        <b-row>
          <b-col lg="12" xl="4">
            <b-row>
              <b-col lg="6">
                <b-form-group label="Name" label-for="name">
                  <b-form-input type="text" id="name" v-model="recipe.name"/>
                </b-form-group>
              </b-col>
              <b-col>
                <b-form-group label="Doses" label-for="doses">
                  <b-form-input type="number" id="doses" v-model="recipe.doses"/>
                </b-form-group>
              </b-col>
            </b-row>
          </b-col>

          <b-col>
            <b-table responsive hover :items="recipe.ingredients" v-if="recipe.ingredients.length" :fields="['name', 'quantity', 'unitOfMeasure', 'actions']">
              <template slot="name" slot-scope="data">
                <input type="text" list="ingredients" v-model="data.item.name">
              </template>
              <template slot="quantity" slot-scope="data">
                <input type="number" v-model="data.item.quantity">
              </template>
              <template slot="unitOfMeasure" slot-scope="data">
                <input type="text" v-model="data.item.unitOfMeasure">
              </template>
              <template slot="actions" slot-scope="data">
                <a @click.stop="removeFromNewRecipe(data)"><i style="color:red; cursor:pointer" class="fas fa-times"></i> </a>
              </template>
            </b-table>
          </b-col>
        </b-row>
        <button class="btn btn-primary mr-2" @click="addIngredientToRecipe()">Add ingredient</button>
        <button class="btn btn-primary ml-2" :disabled="!recipe.ingredients.length" @click="addRecipeToRecipeList()">Add recipe</button>
      </div>
      <br/>
      <b-row>
        <b-col>
          <div v-if="weekRecipes.length">
            <h1>List of recipes</h1>
            <b-table hover responsive :items="weekRecipes" :fields="['id','name','doses','actions']" @row-clicked="addToRecipeIngredients">
              <template slot="actions" slot-scope="data">
                <a @click.stop="removeFromWeekRecipes(data)"><i style="color:red; cursor:pointer" class="fas fa-times"></i> </a>
              </template>
            </b-table>
          </div>
        </b-col>
        <b-col>
          <div v-if="recipeIngredients.length">
            <h1>Selected recipe ingredients</h1>
            <b-table responsive :items="recipeIngredients" />
          </div>
        </b-col>
      </b-row>
      <br/>
      <button v-if="weekRecipes.length" class="btn btn-primary" @click="getTotalIngredients">Get total ingredients</button>
      <br/>
      <br/>
      <b-table responsive v-if="totalIngredients.length" :items="totalIngredients" />
    </b-card>
  </div>
</template>

<script>

/* eslint-disable */
const recipesJson = require('../assets/recipes.json');
const axios = require('axios');
import { Typeahead } from 'uiv';

export default {
  name: 'Orders',
  components: { Typeahead },
  data: function () {
    return {
      selectedRecipe: '',
      recipes: recipesJson,
      ingredients: [],
      weekRecipes: [],
      recipeIngredients: [],
      totalIngredients: [],
      addingARecipe: false,
      recipe: {
        name: '',
        doses: 0,
        ingredients: []
      }
    }
  },
  created () {
    axios.get('https://json-server-heroku-qippsfioeh.now.sh/recipes').then(response => {
      this.recipes = response.data;
      response.data.forEach(r => {
        this.ingredients.push(...r.ingredients.map(i => i.name));
      });
      this.ingredients = this.ingredients.filter((v, i, a) => a.indexOf(v) === i);
    });
  },
  methods: {
    addToWeekRecipes: function (recipe) {
      this.weekRecipes.push(recipe);
    },
    removeFromWeekRecipes: function (recipe) {
      this.weekRecipes = this.weekRecipes.filter(r => r.id !== recipe.item.id);
      this.recipeIngredients = {};
    },
    addIngredientToRecipe: function () {
      this.recipe.ingredients.push({name: '', quantity: 0, unitOfMeasure: ''});
    },
    removeFromNewRecipe: function (ingredient) {
      this.recipeIngredients = this.recipeIngredients.filter(r => r.id !== recipe.item.id);
    },
    addToRecipeIngredients: function (recipe, index) {
      this.recipeIngredients = recipe.ingredients;
    },
    getTotalIngredients: function () {
      this.totalIngredients = [];
      this.weekRecipes.forEach(r => {
        r.ingredients.forEach(i => {
          if (!this.totalIngredients.some(ti => ti.name === i.name)) {
            this.totalIngredients.push(i);
          } else {
            this.totalIngredients.filter(ti => ti.name === i.name).forEach(ti => ti.quantity += i.quantity);
          }
        })
      })
    },
    addRecipeToRecipeList: function () {
      axios.post('http://localhost:3000/recipes', this.recipe).then(() => {
        this.recipes.push(this.recipe);
        alert('recipe added');
      });
      this.addingARecipe = false;
    }
  }
}
</script>
<style>
.dropdown-menu {
  display: table;
}
</style>
