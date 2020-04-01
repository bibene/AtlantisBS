<template>
  <v-data-table dense fixed-header :height="height"
    :headers="computedHeaders"
    :items="items || []"
    :disable-pagination="true"
    :hide-default-footer="true"
  >

    <template v-slot:top>
      <v-toolbar flat color="white" dense>
        {{title}}
        <v-spacer></v-spacer>
        <v-dialog v-model="itemDialog" max-width="500px">
          <template v-slot:activator="{ on }">
            <v-btn small dark class="mb-2" v-on="on" :disabled="disableAdd">Add</v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">{{ itemDialogTitle }}</span>
            </v-card-title>
            <v-card-text>
              <v-container>
                <v-form ref="form" v-model="formValid">
                  <v-row>
                    <v-col v-for="field in headers" :key="field.value" cols="12" sm="6" md="6">
                      <v-checkbox v-if="field.type==='boolean'" v-model="editedItem[field.value]" :label="field.text" color="dark"></v-checkbox>
                      <v-select v-else-if="field.hasOwnProperty('choices')" v-model="editedItem[field.value]" :items="field.choices" :label="field.text" ></v-select>
                      <v-textarea v-else-if="field.type==='text'" v-model="editedItem[field.value]" :items="field.choices" :label="field.text"></v-textarea>
                      <v-text-field v-else :rules="field.rules" v-model="editedItem[field.value]" :label="field.text"></v-text-field>
                    </v-col>
                  </v-row>
                </v-form>
              </v-container>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="dark" text @click="closeItemDialog">Cancel</v-btn>
              <v-btn color="dark" text @click="saveItem">Save</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>     

    <template v-slot:item="{ item }">
      <tr @click="selectItem(item)" :class="{'grey lighten-2': item.id===selectedItemId}">

        <td v-for="(field, index) in headers" :key="field.value" class="text-xs-right">
          <v-simple-checkbox :ripple="false" v-if="field.type==='boolean'" v-model="item[headers[index].value]" color="dark"></v-simple-checkbox>
          <div v-else-if="field.type==='text'"> {{ (item[headers[index].value] || "").replace(/(?:\r\n|\r|\n)/g, ' ') }} </div>
          <div v-else>{{ item[headers[index].value] }}</div>
        </td>

        <td class="text-xs-right"> 
          <v-icon small class="mr-2" @click="editItem(item)">mdi-pencil</v-icon>
          <v-icon small class="mr-2" @click="deleteItem(item)">mdi-delete</v-icon>
        </td>
      </tr>
    </template>

    <template v-slot:no-data>
      {{noDataText}}
    </template>

  </v-data-table>
</template>

<script>
  export default {
    name: 'absTable',
    props: ['title', 'newDialogTitle', 'editDialogTitle', 'headers', 'items', 'selectable', 'noDataText', 'disableAdd', 'height', 'defaultNewItem'],
    events: ['selectedItem'],
    data: () => ({
      itemDialog: false,
      formValid: true,
      selectedItemId: -1,
      editedItem: {},
      defaultItem: {},
    }),

    computed: {
      itemDialogTitle () {
        return (typeof this.editedItem.id === "undefined") ? this.newDialogTitle : this.editDialogTitle
      },
      computedHeaders () {
        //add collumn for edit and delete actions
        var head = [...this.headers];
        head.push({ text: 'Action', value: 'actions', sortable: false, width: 80  })
        return head
      }
    },

    watch: {
      itemDialog (val) {
        val || this.closeItemDialog()
      },
    },

    created () {
      this.initialize()
    },

    methods: {
      initialize () {
        this.editedItem = []
        for (var i = 0; i < this.headers.length; i++) {
          if (this.headers[i].type === 'boolean') {
            this.editedItem[this.headers[i].value] = false 
            this.defaultItem[this.headers[i].value] = false
          }
          else {
            this.editedItem[this.headers[i].value] = ''
            if (typeof this.headers[i].choices==='undefined')
              this.defaultItem[this.headers[i].value] = ''
            else 
              this.defaultItem[this.headers[i].value] = this.headers[i].choices[0]
          }
        }
      },

      selectItem: function (item) {
        if (this.selectable) {
          if (this.selectedItemId === item.id) {
            this.selectedItemId = -1
            this.selectedItem = null
            this.$emit('selectedItem', null)
          }
          else {
            this.selectedItemId = item.id
            this.selectedItem = item 
            this.$emit('selectedItem', item)
          }
        }
      },

      editItem (item) {
        this.editedItem = Object.assign({}, item)
        this.itemDialog = true
      },

      deleteItem (item) {
        const index = this.items.indexOf(item)
        this.items.splice(index, 1)
      },

      closeItemDialog () {
        this.itemDialog = false
        setTimeout(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          //this.editedStrucutreIndex = -1
          this.$refs.form.resetValidation()
        }, 300)
      },

      saveItem () {
        this.$refs.form.validate()
        if (this.formValid) {
          if (typeof this.editedItem.id === "undefined") {
            var maxid = 0;
            if (this.items)
              this.items.map(function(obj){     
                  if (obj.id > maxid) maxid = obj.id    
              })
            else this.items = []
            this.editedItem.id = maxid + 1
            this.items.push(this.editedItem)
          } else {
            const struct_id = this.editedItem.id;
            const index = this.items.findIndex(function(i){return i.id === struct_id})
            Object.assign(this.items[index], this.editedItem)
          }
          this.closeItemDialog()
        }
      },
    },
  }
</script>
