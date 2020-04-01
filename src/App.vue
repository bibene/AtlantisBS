<template>
  <v-app>

    <v-app-bar dark app>
      <v-toolbar dense short flat dark>
        <v-menu offset-y>
          <template v-slot:activator="{ on }">
            <v-app-bar-nav-icon v-on="on"></v-app-bar-nav-icon>
          </template>
          <v-list>
            <v-list-item @click="null;">
              <v-list-item-title><v-icon left>mdi-folder-upload</v-icon>Load {{tabNames[tab]}}</v-list-item-title>
            </v-list-item>
            <v-list-item @click="null;"> 
              <v-list-item-title><v-icon left>mdi-content-save</v-icon>Save {{tabNames[tab]}}</v-list-item-title>
            </v-list-item>
            <v-list-item @click="null;"> 
              <v-list-item-title><v-icon left>mdi-eraser</v-icon>Clear {{tabNames[tab]}}</v-list-item-title>
            </v-list-item>
            <v-list-item @click="null;">
              <v-list-item-title><v-icon left>mdi-cog</v-icon>Settings</v-list-item-title>
            </v-list-item>
          </v-list>
        </v-menu>
        <v-toolbar-title>Atlantis Battle Simulator</v-toolbar-title>
        <v-spacer></v-spacer>
      </v-toolbar>
    </v-app-bar>

    <v-content>
    
    <v-tabs v-model="tab" dark centered>
      <v-tab v-for="(tabName, index) in tabNames" :key="index">{{tabName}}</v-tab>
    </v-tabs>

      <v-container fluid>
        <v-row dense>
          <v-col cols="12">
            <v-row align="start" justify="center">
              <v-card class="ma-1 pa-1 elevation-6" outlined width="40%">
                <ABSTable 
                  :height="200"
                  :headers="structureHeadersWithTypesAndRules"
                  :items="simStructures" 
                  :title="'Structures'"
                  :newDialogTitle="'New Structure'"
                  :editDialogTitle="'Edit Structure'"
                  >
                </ABSTable>
              </v-card>
              <v-card class="ma-1 pa-1 elevation-1" outlined width="50%">
                <ABSTable 
                  :height="200"
                  v-on:selectedItem="handleUnitSelection"
                  :headers="unitHeadersWithStructuresAndRules"
                  :items="activeUnits" 
                  :selectedItem="selectedUnit"
                  :title="'Units'"
                  :newDialogTitle="'New Unit'"
                  :editDialogTitle="'Edit Unit'" 
                  :selectable="true" 
                  :defaultNewItem="{name: '', building: '', behind: false, orders: '', skills: [], items: []}"
                  >
                </ABSTable>
              </v-card>
            </v-row>            
          </v-col>
        </v-row>
        <v-row dense>
          <v-col cols="12">
            <v-row align="start" justify="center">
              <v-card class="ma-1 pa-1 elevation-1" outlined width="45%">
                <ABSTable 
                  :height="200"
                  :headers="itemHeadersWithTypesAndRules"
                  :items="activeItems" 
                  :title="'Items'"
                  :newDialogTitle="'New Item'"
                  :editDialogTitle="'Edit Item'" 
                  :noDataText="selectedUnit ? 'Add Items' : 'Select Unit'" 
                  :disableAdd="selectedUnit ? false : true" 
                  >
                </ABSTable>
              </v-card>
              <v-card class="ma-1 pa-1 elevation-1" outlined width="45%" >
                <ABSTable 
                  :height="200"
                  :headers="skillHeadersWithTypesAndRules"
                  :items="activeSkills" 
                  :title="'Skills'"
                  :newDialogTitle="'New Skill'"
                  :editDialogTitle="'Edit Skill'" 
                  :noDataText="selectedUnit ? 'Add Skills' : 'Select Unit'"
                  :disableAdd="selectedUnit ? false : true"
                  >
                </ABSTable>
              </v-card>
            </v-row>            
          </v-col>
        </v-row>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
import ABSTable from './components/ABSTable.vue';

export default {
  name: 'App',

  components: {
    ABSTable,
  },

  data: () => ({
    tab: 0,
    selectedUnit: null,
    tabNames: ['Attackers', 'Defenders'],

    structureTypes: ['Fleet', 'Tower', 'Fort', 'Castle', 'Citadel', 'Shaft', 'Lair', 'Ruin', 'Cave', 'Magical Tower', 'Magical Fortress', 'Magical Castle', 'Magical Citadel'],
    itemTypes: ['Sword', 'Axe', 'Wool'],
    skillTypes: ['Fire', 'Combat', 'Tactics'],

    structureHeaders: [
        { text: 'Name', value: 'name' },
        { text: 'Type', value: 'type' },
      ],
    unitHeaders: [
        { text: 'Name', value: 'name' },
        { text: 'Structure', value: 'structure' },
        { text: 'Behind', value: 'behind', type: 'boolean' },
        { text: 'Add. Orders', value: 'orders', type: 'text'},
      ],
    skillHeaders: [
        { text: 'Name', value: 'name' },
        { text: 'Study Days', value: 'studyDays' },
      ],
    itemHeaders: [
        { text: 'Name', value: 'name' },
        { text: 'Amount', value: 'amount' },
      ],

    simStructures: [
        { id: 1, name: 'Mags Tower', type: 'Tower' },
        { id: 2, name: 'Castle 1', type: 'Castle' },
      ],
    simData: [ //index 0 - attackers, index 1 - defenders, index matching tab variable 
      {units: [
        { id: 1, name: 'Dark Mage', structure: 'Mags Tower', behind: true,
          skills: [
            { id: 1, name: 'Combat', studyDays: 450 },
            { id: 2, name: 'Fireball', studyDays: 100 },
          ],
          items: [
            { id: 1, name: 'Swords', amount: 1 },
            { id: 2, name: 'Magical Staf', amount: 1 },
          ]
          },
        { id: 2, name: 'Meatline', structure: '', behind: true,
          skills: [
            { id: 1, name: 'Combat', studyDays: 360 },
          ],
          items: [
            { id: 1, name: 'XBows', amount: 100 },
          ]
          },
      ]},
      {units: [
        { id: 1, name: 'White Mage', structure: 'Castle 1', behind: true,
          skills: [
            { id: 1, name: 'Endurance', studyDays: 450 },
            { id: 2, name: 'Lightning', studyDays: 100 },
          ],
          items: [
            { id: 1, name: 'Swords', amount: 1 },
            { id: 2, name: 'Healing Staf', amount: 1 },
          ]
          },
        { id: 2, name: 'Frontline', structure: '', behind: false,
          skills: [
            { id: 1, name: 'Combat', studyDays: 180 },
          ],
          items: [
            { id: 1, name: 'Swords', amount: 100 },
          ]
          },
      ]},
    ],

  }),

  computed: {
    activeUnits () {
      return this.simData[this.tab].units
    },
    activeSkills () {
      if (this.selectedUnit) {        
        return this.selectedUnit.skills 
      } else {
        return []
      }
    },
    activeItems () {
      if (this.selectedUnit) {
        return this.selectedUnit.items
      } else {
        return []
      }
    }, 
    structureHeadersWithTypesAndRules () {
      var headers = [...this.structureHeaders]
      headers.find(s => s.value==='type').choices = this.structureTypes;
      headers.find(s => s.value==='type').rules = [this.requiredRule];
      headers.find(s => s.value==='name').rules = [this.requiredRule];
      return headers
    },
    unitHeadersWithStructuresAndRules () {
      var headers = [...this.unitHeaders]
      var buildings = ['']
      for (var i = 0; i < this.simStructures.length; i++) buildings.push(this.simStructures[i].name)
      headers.find(s => s.value==='structure').choices = buildings;
      headers.find(s => s.value==='name').rules = [this.requiredRule];
      return headers
    }, 
    itemHeadersWithTypesAndRules () {
      var headers = [...this.itemHeaders]
      headers.find(s => s.value==='name').rules = [this.requiredRule];
      headers.find(s => s.value==='name').choices = this.itemTypes;
      headers.find(s => s.value==='amount').rules = [this.requiredRule, this.posNumberRule];
      return headers
    },
    skillHeadersWithTypesAndRules () {
      var headers = [...this.skillHeaders]
      headers.find(s => s.value==='name').rules = [this.requiredRule];
      headers.find(s => s.value==='name').choices = this.skillTypes;
      headers.find(s => s.value==='studyDays').rules = [this.requiredRule, this.posNumberRule, this.numberTo450Rule];
      return headers
    },
  },
  methods: {
    initialize () {
    },
    requiredRule (value) {
      return !!value.trim() || 'Required' 
    },
    posNumberRule (value) {
      return /^\d+$/.test(value) || 'Must be a number'
    },
    numberTo450Rule (value) {
      return (parseInt(value) >= 1 && parseInt(value) <= 450) || 'Must be between 1 and 450'
    },
    handleUnitSelection (unit) {
      this.selectedUnit = unit;
    }

  }

  }
</script>