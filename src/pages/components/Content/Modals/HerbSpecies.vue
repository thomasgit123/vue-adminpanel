<template>
  <div class="modal fade" id="modalHerbSpecies" tabindex="-1" role="dialog" aria-labelledby="myModalLabel">
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <div class="modal-header">
          <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">×</span></button>
          <h4 class="modal-title" id="myModalLabel">Herb Species</h4>
        </div>
        <div class="modal-header no-padding">
          <form action="#" method="get" class="sidebar-form" @submit="search">
            <div class="input-group">
              <input type="text" name="q" class="form-control" placeholder="Search..." v-model="query">
              <span class="input-group-btn">
                <button type="submit" name="search" id="search-btn" class="btn btn-flat"><i class="fa fa-search"></i>
                </button>
              </span>
            </div>
          </form>
        </div>
        <div class="modal-body no-padding" style="overflow: auto; position:relative; height:460px;">
          <table class="table table-hover">
            <tbody v-if="species.items && species.items.length > 0">
            <tr v-for="(item, key) in species.items" :key="key">
              <td style="padding-top:12px;">
                <span>{{item.name}}</span>
              <td>
                <div class="btn-group pull-right">
                  <button type="button" class="btn btn-default" data-toggle="modal" data-target="#modalFormulaCreate" @click="editFormula(item)"><i class="fa fa-pencil"></i></button>
                  <button type="button" :class="selectedSpecieIds.indexOf(item.id) > -1 ? 'btn btn-success' : 'btn btn-default'" @click="toggleSpecie(item.id)"><i :class="selectedSpecieIds.indexOf(item.id) > -1 ? 'fa fa-check' : 'fa fa-plus'"></i></button>
                </div>
              </td>
            </tr>
            </tbody>
            <tbody v-else>
            <tr>
              <td colspan="2">
                No Species
              </td>
            </tr>
            </tbody>
          </table>
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-default pull-left" data-toggle="modal" data-target="#modalFormulaCreate" @click="addFormula()"><i class="fa fa-plus-circle"></i> Create new</button>
          <button type="button" class="btn btn-default" data-dismiss="modal" ref="dialogClose" @click="close">Close</button>
          <button type="button" class="btn btn-primary" @click="save">Save changes</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import { mapGetters } from 'vuex'
  export default {
    name: 'HerbSpecies',
    computed: {
      ...mapGetters({
        currentSpecieIds: 'getHerbSpecieIds',
        modalType: 'getModalType',
        tabName: 'getActiveTabName',
        loading: 'getLoadingBar'
      })
    },
    data () {
      return {
        query: '',
        selectedSpecieIds: [],
        selectedSpecies: [],
        species: [],
        result: '',
        flag: false,
        specie: {
          id: '',
          name: ''
        }
      }
    },
    created () {
      this.fetchHerbSpecies()
    },
    watch: {
      currentSpecieIds: {
        handler () {
          this.selectedSpecieIds = Object.assign([], this.currentSpecieIds)
        }
      },
      flag: {
        handler () {
          if (this.flag === true) this.$store.dispatch('setLoadingBar', 'none')
        }
      },
      modalType: {
        handler () {
          this.result = ''
          if (this.modalType === 0 && this.tabName === 'Species') {
            this.flag = false
            this.result = 'success'
            this.$store.dispatch('setLoadingBar', 'block')
            this.$store.dispatch('setLoadingBarText', 'Loading...')
            this.fetchHerbSpecies()
          } else if (this.modalType === 5 && this.tabName === 'Species') {
            if (this.flag === false) {
              this.$store.dispatch('setLoadingBar', 'block')
              this.$store.dispatch('setLoadingBarText', 'Loading...')
            }
            this.selectedSpecieIds = Object.assign([], this.currentSpecieIds)
          }
        }
      }
    },
    methods: {
      fetchHerbSpecies: function () {
        this.$store.dispatch('fetchSpeciesTotal', {query: 'query=' + this.query})
          .then((response) => {
            this.flag = true
            this.species = response
          })
      },
      toggleSpecie (id) {
        let index = this.selectedSpecieIds.indexOf(id)
        if (index > -1) {
          this.selectedSpecieIds.splice(index, 1)
        } else {
          this.selectedSpecieIds.push(id)
        }
      },
      save: function () {
        this.selectedSpecies = []
        this.species.items.map((item, ind) => {
          let index = this.selectedSpecieIds.indexOf(item.id)
          if (index > -1) {
            this.selectedSpecies.push(item)
          }
        })
        this.$store.dispatch('setHerbSpecies', this.selectedSpecies)
        this.result = 'success'
        const btn = this.$refs.dialogClose
        btn.click()
      },
      search: function (evt) {
        evt.preventDefault()
        this.$store.dispatch('setLoadingBar', 'block')
        this.$store.dispatch('setLoadingBarText', 'Searching...')
        this.$store.dispatch('fetchSpecieTotal', {query: 'query=' + this.query})
          .then((response) => {
            this.$store.dispatch('setLoadingBar', 'none')
            this.species = response
          }).catch((error) => {
            console.log(error)
          })
      },
      addFormula: function () {
        this.formula.pinyin = ''
        this.formula.english_name = ''
        this.formula.latin_name = ''
        this.formula.traditional_chinese = ''
        this.formula.simplified_chinese = ''
        this.formula.pinyin_ton = ''
        this.$store.dispatch('setModalType', 1)
        this.$store.dispatch('setFormulaValue', this.formula)
      },
      editFormula: function (item) {
        this.formula.id = item.id
        this.formula.pinyin = item.pinyin
        this.formula.english_name = item.english_name
        this.formula.latin_name = item.latin_name
        this.formula.traditional_chinese = item.traditional_chinese
        this.formula.simplified_chinese = item.simplified_chinese
        this.formula.pinyin_ton = item.pinyin_ton
        if (item.modifiedBy === null || item.modifiedBy === undefined) this.formula.modified_by_name = ''
        else this.formula.modified_by_name = item.modifiedBy.firstname + ' ' + item.modifiedBy.lastname
        this.formula.modified_time = item.modified_time
        if (item.createdBy === null || item.createdBy === undefined) this.formula.created_by_name = ''
        else this.formula.created_by_name = item.createdBy.firstname + ' ' + item.createdBy.lastname
        this.formula.created_time = item.created_time
        this.$store.dispatch('setModalType', 2)
        this.$store.dispatch('setFormulaValue', this.formula)
      },
      close: function () {
        if (this.result === 'success') {
          this.$store.dispatch('setModalType', 4)
        } else {
          this.$store.dispatch('setModalType', 6)
        }
      }
    }
  }
</script>
