<template>
  <div v-if="$route.name === 'Users'">
    <section class="content-header">
      <button type="button" class="btn btn-sm btn-default" @click="back()"><i class="fa fa-arrow-left"></i> Back</button>
      <h1>Users</h1>
    </section>
    <section class="content">
      <div class="row">
        <div class="col-xs-12">
          <div class="box">
            <div class="box-header">
              <h3 class="box-title">All users</h3>
              <FBButton buttonClass="btn btn-success right-position" buttonText="Add user" :onClick="addUser">
                      <i slot="icon" class="fa fa-user-plus"></i>
              </FBButton>
              <select v-model="selected" v-on:change="onChangeSort" class="form-control sort">
                <option v-for="value in sorts" :value="value" :key="value">
                  {{value}}
                </option>
              </select>
            </div>
            <div style="display:none;">
            <FBDataTable ref="userTable" :css="css.table" :fields="fields" dataPath="items"
                 :muti-sort="true" multi-sort-key="ctrl" :query = "query"
                 :methodName="methodName" pagination-path="pagination" @fbdatatable:pagination-data="onPaginationData" item-key="id">
              <FBPagination slot="pagination" :css="css.pagination"></FBPagination>
            </FBDataTable>
            </div>
            <!-- /.box-header -->

            <div class="box-body table-responsive no-padding">

              <table class="table table-hover">
                <tbody>
                <tr>
                  <th><input type="checkbox"></th>
                  <th>ID <i class="fa fa-caret-down"></i></th>
                  <th>
                    <div class="input-group input-group-sm" style="width: 150px;">
                      <input type="text" name="table_search" class="form-control pull-right" placeholder="Search"
                             v-model="searchQuery">
                      <div class="input-group-btn">
                        <FBButton type="submit" :onClick="searchUser" buttonClass="btn btn-default">
                          <i slot="icon" class="fa fa-search"></i>
                        </FBButton>
                      </div>
                    </div>
                  </th>
                  <th>Subscriber <i class="fa fa-caret-down"></i></th>
                  <th>Status <i class="fa fa-caret-down"></i></th>
                  <th>Type <i class="fa fa-caret-down"></i></th>
                  <th>Created <i class="fa fa-caret-down"></i></th>
                  <th>Modified by <i class="fa fa-caret-down"></i></th>
                </tr>

                <tr v-for="user in users.items" :key="user.id">
                  <td><input name="deleting[]" type="checkbox" :value="user.id" v-model="selectedUsers"></td>
                  <td>{{user.id}}</td>
                  <td>
                    <div class="user-block">
                      <img class="img-circle" src="../assets/img/user3-128x128.jpg"
                           :alt="user | fullUserName">
                      <span class="username">
                         <router-link
                           :to="{path: '/users/profile/'+user.id}" style="color:red"
                           v-if="$lodash.toNumber(user.is_deleted) === 1">{{user | fullUserName}}</router-link>
                        <router-link v-else-if="user.is_deleted !== 1"
                                     :to="{path: '/users/profile/'+user.id}">{{user | fullUserName}}</router-link>
                        <router-link style="color:black" v-else
                                     :to="{path: '/users/profile/'+user.id}">{{user | fullUserName}}</router-link>
                      </span>
                      <span class="description" v-if="user.lastLogin">
                        Last login - {{user.lastLogin.created_at | toDate}}
                      </span>
                    </div>
                  </td>
                  <td>
                    <span class="label label-success">Subscribed</span>
                  </td>
                  <td v-if="user.status === 1">Active</td>
                  <td v-else>In-active</td>
                  <td>
                    {{user.role}}
                  </td>
                  <td>
                    <div class="text-muted" v-if="user.created_at">
                      {{ user.created_at | toDate }}
                    </div>
                  </td>
                  <td><span v-if="user.modifiedBy"> {{ user.modifiedBy | fullUserName }} </span>
                    <div class="text-muted" v-if="user.modified_at !== null">
                      {{ user.modified_at | toDate }}
                    </div>
                  </td>
                  <td>
                    <div class="btn-group">
                      <button type="button" class="btn btn-default">Profile</button>
                      <button type="button" class="btn btn-default dropdown-toggle" data-toggle="dropdown"
                              aria-expanded="false">
                        <span class="caret"></span>
                        <span class="sr-only">Toggle Dropdown</span>
                      </button>
                      <ul class="dropdown-menu" role="menu">
                        <li><a href="#">Edit</a></li>
                        <li><a href="#">Reset Password</a></li>
                        <li><a href="#" @click.prevent="deleteUserData($event)" :data-user-ID="user.id">Delete User Data</a>
                        </li>
                        <li class="divider"></li>
                        <li><a href="#">Make in-active</a></li>
                      </ul>
                    </div>
                  </td>
                </tr>
                </tbody>
              </table>
              <div class="box-footer clearfix">
                <div class="pull-left">
                  <FBButton :onClick="deleteUsers" buttonClass="btn btn-default" buttonText="Delete Selected Users">
                    <i slot="icon" class="fa fa-user-times"></i>
                  </FBButton>
                  <FBButton :onClick="deleteUsersData" buttonClass="btn btn-default" buttonText="Deleted Selected User's Datas">
                    <i slot="icon" class="fa fa-user-times"></i>
                  </FBButton>
                </div>
                <div class="box-footer clearfix">
              <FBPagination @fbdatatable-pagination:change-page="onChangePage" ref="pagination" :css="css.pagination"></FBPagination>
            </div>
              </div>
            </div>
            <!-- /.box-body -->

          </div>
          <!-- /.box -->
        </div>
      </div>
    </section>
    <AddUserModal ref="addUserModal"></AddUserModal>
    <LoadingModal></LoadingModal>
  </div>
  <div v-else>
    <router-view></router-view>
  </div>
</template>
<style>
  .pagination {
    margin-top: 0px !important;
  }
  .pull-left {
    margin-top: 10px !important;
  }
</style>
<script>
  import { mapGetters } from 'vuex'
  import FBButton from '../components/FBButton.vue'
  import datatableConfig from '../config/datatable'

  const AddUserModal = () => import(/* webpackChunkName: "UI Elements" */ './components/Users/AddUserModal.vue')
  const LoadingModal = () => import(/* webpackChunkName: "Modals" */ './components/Content/TCMModals/LoadingModal.vue')
  const FBDataTable = () => import(/* webpackChunkName: "UI Elements" */ '../components/FBDataTable.vue')
  const FBPagination = () => import(/* webpackChunkName: "UI Elements" */ '../components/FBPagination.vue')

  export default {
    name: 'Users',
    data () {
      return {
        css: datatableConfig,
        selected: 'All',
        sorts: [
          'All', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N',
          'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z'
        ],
        fields: [
          {
            title: 'Check',
            name: '__slot:check'
          },
          {
            title: 'Name',
            name: '__slot:name'
          },
          {
            title: 'Simplified',
            name: '__slot:simplified'
          },
          {
            title: 'Category',
            name: '__slot:category'
          },
          {
            title: 'Associated',
            name: '__slot:associated'
          },
          {
            title: 'Created',
            name: '__slot:created'
          },
          {
            title: 'Date',
            name: '__slot:date'
          },
          {
            title: '',
            name: '__slot:control'
          }
        ],
        searchQuery: '',
        methodName: 'fetchSearchUsers',
        query: '',
        selectedUsers: [],
        pagination: {
          page: 1
        }
      }
    },
    components: {
      FBButton,
      AddUserModal,
      LoadingModal,
      FBDataTable,
      FBPagination
    },
    methods: {
      back: function () {
        this.$router.go(-1)
      },
      onChangeSort () {
        this.$store.dispatch('setAlabetSort', this.selected)
        this.$store.dispatch('setLoadingBar', 'block')
        this.$store.dispatch('setLoadingBarText', 'Loading...')
        this.$store.dispatch('fetchSearchUsers', { query: this.searchQuery, page: this.pagination.page })
          .then((response) => {
            this.$store.dispatch('setLoadingBar', 'none')
            this.$refs.userTable.reload()
          })
          .catch((error) => {
            this.$store.dispatch('setLoadingBar', 'none')
            this.$swal(
              'Error!',
              error.message,
              'error'
            )
          })
      },
      onPaginationData (paginationData) {
        this.$refs.pagination.setPaginationData(paginationData)
      },
      onChangePage (page) {
        this.$refs.userTable.changePage(page)
      },
      nextPage: function (p) {
        this.pagination.page = this.pagination.page + p
      },
      prevPage: function (p) {
        this.pagination.page = this.pagination.page - p
      },
      searchUser () {
        this.methodName = 'fetchSearchUsers'
        this.query = this.searchQuery
      },
      addUser () {
        this.$refs.addUserModal.$refs.modal.showModal = true
      },
      deleteUserData ($event) {
        this.$swal({
          title: 'Are you sure?',
          text: 'You will not be able to recover this user!',
          type: 'warning',
          showCancelButton: true,
          confirmButtonText: 'Yes, delete it!',
          cancelButtonText: 'No, keep it!'
        }).then(function () {
          this.$store.dispatch('setLoadingBar', 'block')
          this.$store.dispatch('setLoadingBarText', 'Deleting User Data...')
          this.$store.dispatch('deleteUserData', $event.target.dataset.userId)
            .then((response) => {
              this.$store.dispatch('setLoadingBar', 'none')
              this.$swal(
                'Deleted!',
                response.message,
                'success'
              )
              // this.$store.dispatch('fetchAllUsers', this.$store.getters.getUserAccessToken)
            })
            .catch((error) => {
              this.$store.dispatch('setLoadingBar', 'none')
              this.$swal(
                'Error!',
                error.message,
                'error'
              )
            })
        }.bind(this), function (dismiss) {
          if (dismiss === 'cancel') {
            this.$swal(
              'Cancelled',
              'Cancelled',
              'error'
            )
          }
        }.bind(this))
      },
      deleteUsers () {
        this.$swal({
          title: 'Are you sure?',
          text: 'You will not be able to recover this user!',
          type: 'warning',
          showCancelButton: true,
          confirmButtonText: 'Yes, delete it!',
          cancelButtonText: 'No, keep it!'
        }).then(function () {
          this.$store.dispatch('setLoadingBar', 'block')
          this.$store.dispatch('setLoadingBarText', 'Deleting Selected Users...')
          this.$store.dispatch('deleteUser', {users: this.selectedUsers})
            .then((response) => {
              this.$store.dispatch('setLoadingBar', 'none')
              this.selectedUsers = []
              this.$refs.userTable.reload()
            })
            .catch((error) => {
              this.$store.dispatch('setLoadingBar', 'none')
              this.$swal(
                'Error!',
                error.message,
                'error'
              )
            })
        }.bind(this), function (dismiss) {
          if (dismiss === 'cancel') {
            this.$swal(
              'Cancelled',
              'Cancelled',
              'error'
            )
          }
        }.bind(this))
      },
      deleteUsersData () {
        this.$swal({
          title: 'Are you sure?',
          text: 'You will not be able to recover this user!',
          type: 'warning',
          showCancelButton: true,
          confirmButtonText: 'Yes, delete it!',
          cancelButtonText: 'No, keep it!'
        }).then(function () {
          this.$store.dispatch('setLoadingBar', 'block')
          this.$store.dispatch('setLoadingBarText', 'Deleting Selected Users\'s Data...')
          this.$store.dispatch('deleteUserData', {users: this.selectedUsers})
            .then((response) => {
              this.$store.dispatch('setLoadingBar', 'none')
              this.selectedUsers = []
              this.$swal(
                'Deleted!',
                response.message,
                'success'
              )
            })
            .catch((error) => {
              this.$store.dispatch('setLoadingBar', 'none')
              this.$swal(
                'Error!',
                error.message,
                'error'
              )
            })
        }.bind(this), function (dismiss) {
          if (dismiss === 'cancel') {
            this.$swal(
              'Cancelled',
              'Cancelled',
              'error'
            )
          }
        }.bind(this))
      }
    },
    beforeCreate () {
      this.$store.dispatch('setAlabetSort', '')
      if (!this.$store.getters.getCountries || this.$store.getters.getCountries.length === 0) {
        this.$store.dispatch('fetchCountries')
      }
      if (!this.$store.getters.getLanguages || this.$store.getters.getLanguages.length === 0) {
        this.$store.dispatch('fetchLanguages')
      }
      this.$store.dispatch('fetchRoles')
      this.$store.dispatch('fetchUserTypes')
    },
    created () {
      // this.$store.dispatch('setLoadingBar', 'block')
      // this.$store.dispatch('setLoadingBarText', 'Loading Users...')
      // this.$store.dispatch('fetchAllUsers', {token: this.$store.getters.getUserAccessToken, page: this.pagination.page})
      //   .then((response) => {
      //     this.$store.dispatch('setLoadingBar', 'none')
      //   })
    },
    watch: {
      searchQuery: function (query) {
        if (this.searchQuery === '') {
          this.methodName = 'fetchAllUsers'
          this.query = ''
          // this.$store.dispatch('fetchAllUsers', {token: this.$store.getters.getUserAccessToken})
        }
      }
    },
    computed: {
      ...mapGetters({
        'users': 'getUsers'
      })
    }
  }
</script>
