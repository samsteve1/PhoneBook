<template>
  <div class="">
    <nav class="panel column is-offset-2 is-8">
    <p class="panel-heading">
      Vuejs PhoneBook
      <button class="button is-primary is-outlined" @click="openAdd">
      Add New
      </button>
      <span class="is-pulled-right" v-if="loading">
        <i class="has-text-primary fa fa-refresh fa-spin fa-2x fa-fw"></i>
      </span>
    </p>
    <div class="panel-block">
      <p class="control has-icons-left">
        <input class="input is-small" type="text" placeholder="search" v-model="searchQuery">
        <span class="icon is-small is-left">
          <i class="fa fa-search"></i>
        </span>
      </p>
    </div>

  <a class="panel-block" v-for="item,key in temp">
    <span class="column is-9">
    {{item.name}}
    </span>
    <span class="panel-icon column is-1">
      <li class="has-text-danger fa fa-trash" @click="del(key, item.id)"></li>
    </span>

    <span class="panel-icon column is-1">
      <li class="has-text-info fa fa-edit" aria-hidden="true" @click="openUpdate(key)"></li>
    </span>

    <span class="panel-icon column is-1">
      <li class="has-text-primary fa fa-eye" aria-hidden="true" @click="openShow(key)"></li>
    </span>
  </a>
  </nav>
  <Add :openmodal='addActive' @closeRequest="close"></Add>
  <Show :openmodal='showActive' @closeRequest="close"></Show>
  <Update :openmodal="updateActive" @closeRequest="close"> </update>
  </div>

</template>
<script>
  let Add = require('./Add.vue');
  let Show =require('./Show.vue');
  let Update = require('./Update.vue');
  export default{
    components: {Add, Show, Update},
    data(){
      return{
        addActive: '',
        showActive: '',
        updateActive:'',
        phoneList:{},
        loading:false,
        searchQuery: '',
        temp:'',
        errors:{}
      }
    },
    watch:{
      searchQuery(){
        if(this.searchQuery.length > 0){
            this.temp = this.phoneList.filter((item) => {
            return Object.keys(item).some((key)=>{
              let string = String(item[key])
              return string.toLowerCase().indexOf(this.searchQuery.toLowerCase())>-1
            })
            });
        }
        else{
          this.temp = this.phoneList
        }
      }
    },
    mounted() {
      //do something after mounting vue instance
      axios.post('/getData')
      .then((response) => this.phoneList = this.temp = response.data)
      .catch((error)   => this.errors = error.response.errors)
    },
    methods: {
      openAdd() {
        this.addActive = 'is-active';
      },
      openShow(key){
        this.$children[1].list = this.temp[key]
        this.showActive = 'is-active';
      },
      openUpdate(key){
        this.$children[2].list =  this.temp[key]
        this.updateActive = 'is-active';
      },
      del(key, id){
        if(confirm("Are you sure ?")){
          this.loading = !this.loading
          axios.delete(`/phonebook/${id}`)
            .then((response) => {this.phoneList.splice(key,1); this.loading = !this.loading})
            .catch((error) => this.errors = error.response.data.errors)
        }
      },
      close(){
        this.addActive = this.updateActive = this.showActive = ''
      }
    }
  }
</script>
