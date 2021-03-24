<template>
  <div class="container" id="home">
    <div class="d-flex align-items-center justify-content-end m-3 lead">
      <h1 class="text-center border-bottom mr-auto">Handling JSON</h1>
      <div class="mr-2">
        <select class="form-control mr-1 selectStatus clickable" v-model="pageSize" @click="updateVisibleUsers" >
          <option v-bind:value="50">50 per page</option>
          <option v-bind:value="100">100 per page</option>
          <option v-bind:value="200">200 per page</option>
          <option v-bind:value="500">500 per page</option>
          <option v-bind:value="1000">1000 per page</option>
          <option v-bind:value="5000">5000 per page</option>
        </select>
      </div><!-- mr-2 -->
      <Pagination 
          v-bind:stateUsers="stateUsers"
          v-on:page-update="updatePage"
          v-bind:currentPage="currentPage"
          v-bind:pageSize="pageSize" />
    </div><!-- d-flex align-items-center justify-content-end m-3 lead -->
    <div class="d-flex mb-2">
      <input type="text" class="form-control mr-1 filterName" v-model="searchName" placeholder="Name exp: Allyson">
      <input type="text" class="form-control mr-1 filterBalance" v-model="searchBalance" placeholder="Balance exp: 2,972.88">
      <select class="form-control mr-1 selectStatus" v-model="isActive" >
        <option v-bind:value="'all'">Both</option>
        <option v-bind:value="true">Active</option>
        <option v-bind:value="false">Not Active</option>
      </select>
      <input type="date" class="form-control mr-1 filterDate" v-model="searchDate" placeholder="Reg. exp: 2014-02-22T12:35:59">
      <input type="text" class="form-control mr-1 filterState" v-model="searchState" placeholder="State exp: Colorado">
      <input type="text" class="form-control filterCountry" v-model="searchCountry" placeholder="State exp: Cyprus">
    </div><!-- d-flex -->
    
    <div class="row">
      <div class="col-12">
        <table class="table">
          <thead class="thead-dark">
            <tr>
              <th scope="col">#</th>
              <th scope="col" class="clickable" @click="sortByName">Full Name <i class=" fas" :class="{'fa-sort-alpha-down' : !sortOrderName,'fa-sort-alpha-up': sortOrderName }"></i></th>
              <th scope="col" class="clickable" @click="sortByBalance" >Balance <i class=" fas " :class="{'fa-sort-amount-down' : !sortOrderBalance,'fa-sort-amount-up': sortOrderBalance }"></i></th>
              <th scope="col" class="clickable" @click="sortByActive">Active <i class=" fas " :class="{'fa-toggle-on' : !sortOrderActive,'fa-toggle-off': sortOrderActive }"></i></th>
              <th scope="col" class="clickable" @click="sortByDate">Registered <i class=" fas " :class="{'fa-sort-numeric-up' : !sortOrderDate,'fa-sort-numeric-down': sortOrderDate }"></i></th>
              <th scope="col" class="clickable" @click="sortByState">State <i class=" fas " :class="{'fa-sort-alpha-down' : !sortOrderState,'fa-sort-alpha-up': sortOrderState }"></i></th>
              <th scope="col" class="clickable" @click="sortByCountry">Country <i class=" fas " :class="{'fa-sort-alpha-down' : !sortOrderCountry,'fa-sort-alpha-up': sortOrderCountry }"></i></th>
            </tr>
          </thead>
          <tbody>
            <router-link class="clickable" tag="tr" :to="{name: 'User', params: {user_id: item.id , stateUsers: stateUsers } }" @click="userInfo(item)" v-for="(item,index) in filteredUser" :key="item.id">
              <th scope="row">{{ indexShowInTable + index +1}}</th>
              <td>{{item.fullName}}</td>
              <td>{{item.balance}}</td>
              <td>{{item.isActive}}</td>
              <td>{{item.registered}}</td>
              <td>{{item.name}}</td>
              <td>{{item.country}}</td>
            </router-link>
          </tbody>
        </table>
      </div><!-- col-12 -->
    </div><!-- row -->
    <!-- Back to top of page -->
    <a href="#home" id="toTop" v-smooth-scroll><i class="fas fa-arrow-alt-circle-up fa-4x"></i></a>
  </div>
</template>

<script>
import axios from 'axios';
import Pagination from './Pagination.vue'

export default {
  name: 'Home',
  data () {
    return {
      info:null,
      countryState:[],
      stateUsers:[],
      searchName:'',
      searchBalance:'',
      searchState:'',
      searchCountry:'',
      isActive: 'all',
      searchDate:'',
      sortOrderName: null,
      sortOrderBalance: null,
      sortOrderState: null,
      sortOrderCountry: null,
      sortOrderActive: null,
      sortOrderDate: null,
      currentPage: 0,
      pageSize: 200,
      visibleUsers: [],
      indexShowInTable:0
    }
  },
  components:{
    Pagination
  },
  watch : {
    searchName:function(val) {
      this.searchBalance = '';
      this.isActive = 'all';
      this.searchDate = '';
      this.searchState = '';
      this.searchCountry = '';
    },
    searchBalance:function(val) {
      this.searchName = '';
      this.isActive = 'all';
      this.searchDate = '';
      this.searchState = '';
      this.searchCountry = '';
    },
    isActive:function(val) {
      this.searchName = '';
      this.searchBalance = '';
      this.searchDate = '';
      this.searchState = '';
      this.searchCountry = '';
    },
    searchDate:function(val) {
      this.searchName = '';
      this.isActive = 'all';
      this.searchBalance = '';
      this.searchState = '';
      this.searchCountry = '';
    },
    searchState:function(val) {
      this.searchName = '';
      this.isActive = 'all';
      this.searchDate = '';
      this.searchBalance = '';
      this.searchCountry = '';
    },
    searchCountry:function(val) {
      this.searchName = '';
      this.isActive = 'all';
      this.searchDate = '';
      this.searchState = '';
      this.searchBalance = '';
    }
  },
  computed: {
    filteredUser(){
      var pickSearch = this.searchOption();

      if(pickSearch.option == 'fullName'){
        // console.log(pickSearch.name.toLowerCase());
        return this.visibleUsers.filter((name) => {
          return name.fullName.toLowerCase().match(pickSearch.name)
        });
      } else if(pickSearch.option == 'balance'){
        return this.visibleUsers.filter((name) => {
          return name.balance.match(pickSearch.name)
        });
      } else if(pickSearch.option == 'state'){
        return this.visibleUsers.filter((name) => {
          return name.name.toLowerCase().match(pickSearch.name)
        });
      } else if(pickSearch.option == 'country'){
        return this.visibleUsers.filter((name) => {
          return name.country.toLowerCase().match(pickSearch.name)
        });
      } else if(pickSearch.option == 'registered'){
        return this.visibleUsers.filter((name) => {
          return name.registered.match(pickSearch.name)
        });
      } else if(pickSearch.option == 'isActive'){
        if(pickSearch.name == 'all'){
          return this.visibleUsers.filter((name) => {
            return name
          });
        } else {
          return this.visibleUsers.filter((name) => {
            return name.isActive == pickSearch.name
          });
        }
      }
    }
  },
  methods:{
    // Here JSON is filter and sorted in one array of Objects that is easier to loop over
    sortingJSON(){
      this.info.forEach(element => {
          for(let i = 0; i < element.state.length; i++){
            let users = element.state[i];
            users.country = element.country;
            // Add all country is one array
            this.countryState.push(users);
          }
      });
      // Reset all users
      this.stateUsers = [];

      this.countryState.forEach(element =>{
        for(let i = 0; i < element.users.length;i++){
          var fixTime = element.users[i].registered.replace("T"," ")
          let user = element.users[i]
          user.name = element.name;
          user.country = element.country;
          user.registered = fixTime;
          // Add all users in one array
          this.stateUsers.push(user)
        }
      });
      
      this.updateVisibleUsers();

    },
    getData() {
      // Geting data from API - FWW
      return axios.get('https://fww-demo.herokuapp.com/').then(response => (this.info = response.data))
    },
    searchOption(){
      if(this.searchName != ''){
        return {
          name: this.searchName.toLowerCase(),
          option: 'fullName'
        };
      } else if(this.searchBalance != ''){
        return {
          name: this.searchBalance,
          option: 'balance'
        };
      } else if(this.searchState != ''){
        return {
          name: this.searchState.toLowerCase(),
          option: 'state'
        };
      } else if(this.searchCountry != ''){
        return {
          name: this.searchCountry.toLowerCase(),
          option: 'country'
        };
      } else if(this.searchDate != ''){
        return {
          name: this.searchDate,
          option: 'registered'
        };
      } else if(this.isActive == 'all' || this.isActive == true || this.isActive == false){
        return {
          name: this.isActive,
          option: 'isActive'
        };
      }
    },
    sortByName(){
      var order = !this.sortOrderName;
      this.sortOrderName = !this.sortOrderName;
      this.visibleUsers.sort(function (a, b) {
        var nameA = a.fullName.toUpperCase(); 
        var nameB = b.fullName.toUpperCase(); 
        if(order){
          if (nameA < nameB) {
            return -1;
          } else {
            return 1
          }
        } else {
          if (nameA < nameB) {
            return 1;
          } else {
            return -1
          }
        }
        return 0;
      });
    },
    sortByBalance(){
      var order = !this.sortOrderBalance;
      this.sortOrderBalance = !this.sortOrderBalance;
      this.visibleUsers.sort(function (a, b) {
        var balanceA = parseFloat(a.balance.substring(1).replace(",","").replace(".",""))
        var balanceB = parseFloat(b.balance.substring(1).replace(",","").replace(".",""))
        if(order){
          return balanceA - balanceB
        } else {
          return balanceB - balanceA
        }
      });
    },
    sortByActive(){
      var order = !this.sortOrderActive;
      this.sortOrderActive = !this.sortOrderActive;
      this.visibleUsers.sort(function(x, y) {
        if(order){
          return (x.isActive === y.isActive) ? 0 : x.isActive ? 1 : -1;
        } else {
          return (x.isActive === y.isActive) ? 0 : x.isActive ? -1 : 1;
        }
      });
    },
    sortByState(){
      var order = !this.sortOrderState;
      this.sortOrderState = !this.sortOrderState;
      this.visibleUsers.sort(function (a, b) {
        var stateA = a.name.toUpperCase(); 
        var stateB = b.name.toUpperCase(); 
        if(order){
          if (stateA < stateB) {
            return -1;
          } else {
            return 1
          }
        } else {
          if (stateA < stateB) {
            return 1;
          } else {
            return -1
          }
        }
        return 0;
      });
    },
    sortByCountry(){
      var order = !this.sortOrderCountry;
      this.sortOrderCountry = !this.sortOrderCountry;
      this.visibleUsers.sort(function (a, b) {
        var countryA = a.country.toUpperCase(); 
        var countryB = b.country.toUpperCase(); 
        if(order){
          if (countryA < countryB) {
            return -1;
          } else {
            return 1
          }
        } else {
          if (countryA < countryB) {
            return 1;
          } else {
            return -1
          }
        }
        return 0;
      });
    },
    sortByDate(){
      var order = !this.sortOrderDate;
      this.sortOrderDate = !this.sortOrderDate;
      this.visibleUsers.sort(function (a, b) {
        var registeredA = a.registered; 
        var registeredB = b.registered; 
        if(order){
          return new Date(registeredB) - new Date(registeredA);
        } else {
          return new Date(registeredA) - new Date(registeredB);
        }
      });
    },
    updatePage(pageNumber) {
      this.currentPage = pageNumber;
      this.updateVisibleUsers();
    },
    updateVisibleUsers() {
      this.visibleUsers = this.stateUsers.slice(this.currentPage * this.pageSize, (this.currentPage * this.pageSize) + this.pageSize);

      // if we have 0 visible todos, go back a page
      if (this.visibleUsers.length == 0 && this.currentPage > 0) {
        this.updatePage(this.currentPage -1);
      }
    },
    userInfo(user){
      console.log(user);
    }
  },
  created () {
    this.getData().then((result) => {
      this.sortingJSON()
    })
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
a#toTop {
  position: fixed;
  bottom: 5vh;
  right: 20vw;
}
.clickable {
  cursor: pointer;
}
</style>
