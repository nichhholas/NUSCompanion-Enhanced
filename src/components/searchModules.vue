<template>
  <div id="show-modules">
    <h1>Search NUS Modules</h1>
    <input type= "text" v-model="search" placeholder="search modules"/>
    <div>
      <!-- Using modifiers -->
      <b-button v-b-toggle.collapse-2 class="m-1">Advanced Search</b-button>

      <!-- Element to collapse -->
      <b-collapse id="collapse-2">
        <b-card>
          <div>
            Choose Rating:
            <b-form-select v-model="rating" :options="options_rating"></b-form-select>
            Choose Webcast Availability:
            <b-form-select v-model="webcast" :options="options_webcast"></b-form-select>
            Choose SU Option:
            <b-form-select v-model="SU" :options="options_SU"></b-form-select>
            Choose Project Work Availability:
            <b-form-select v-model="project" :options="options_project"></b-form-select>
            Department:
            <input type= "text" v-model="department" placeholder="search department"/>
          </div>
        </b-card>
      </b-collapse>
    </div>
    <div class="text-xs-center" v-if="loading">
      <v-progress-circular
        indeterminate
        color="primary"
      ></v-progress-circular>
    </div>
    <div v-for="module in filteredModules" :key="module.id" class="single-module">
      <!--  <router-link :to="'/module/'+module['.key']"> <h2>{{ module[".key"] }} </h2> </router-link> -->
      <router-link :to="'/Module/'+module.ModuleCode"> <h2>{{ module.ModuleCode }} </h2> </router-link>
        <h2>{{module.ModuleTitle}}</h2>
        <article>
        <table>
            <tr>
              <td class="p-left">
                Department: {{module.Department}}</td>
                <td class="p-right">Semester {{module['History']['0']['Semester']}}</td>
            </tr>
            <tr>
              <td class="p-left">
                Exam Date: {{module['History']['0']['ExamDate']}}</td>
              <td class="p-right">Rating: {{module.rating}}</td>
            </tr>
            <tr>
              <td class="p-left">{{getProject(module.project)}}</td>
              <td class="p-right">Module Credit: {{module.ModuleCredit}}</td>
            </tr>
            <tr> 
              <td class="p-left">Workload: {{module.Workload}}</td>
              <td class="p-right">{{getSU(module.su)}}</td>
            </tr>
            <tr> 
              <td class="p-left"> {{getWebcast(module.webcast)}} </td>
            </tr>
        </table>
        <p>{{module.ModuleDescription}}</p>
        <p>Preclusion: {{module.Preclusion}}</p>
        <p>Prerequisite: {{module.Prerequisite}}</p>
      </article>
    </div>
  </div>
</template>

<script>
//import HelloWorld from './HelloWorld'
import {db} from "@/firebase.js"
// import {seRef} from '@/firebase.js'
import {modsInfo} from '@/firebase.js'
export default {
  name: 'searchModules',
  components: {
  
  }, 
  firebase: {
    // modules: seRef
    modules: modsInfo
  },
  data () {
    return {
      loading: false,
      search: '',
      SU: '',
      department: '',
      test: 123,
      webcast: '',
      project: '',
      rating:'',
      options_webcast: [
          { value: '', text: 'Choose Webcast Availability' },
          { value: '1', text: 'Webcast Available' },
          { value: '0', text: 'No Webcast Available' },
      ],
      options_SU: [
          { value: '', text: 'Choose SU Option' },
          { value: '1', text: 'SU Option Available' },
          { value: '0', text: 'No SU Option Available' },
      ],
      options_project: [
          { value: '', text: 'Choose Project Option' },
          { value: '1', text: 'Module consists of Project Work' },
          { value: '0', text: 'Module does not consist of Project Work' },
      ],
      options_rating: [
          { value: 0, text: 'Choose Rating' },
          { value: 1, text: '>1' },
          { value: 2, text: '>2' },
          { value: 3, text: '>3' },
          { value: 4, text: '>4' },
      ],
    }
  },
  methods:{
        async getData(){
    var data = await modsInfo.once("value")
      .then(function(snapshot) {
        var d = snapshot.val();
        // console.log("whewww")
        //console.log(d)
        return d;
      } );  
    console.log(data);
    },
    getWebcast(input) {
      if (input == 1) {
        return "Webcast Available"
      }
      else {
        return "No Webcast Available"
      }
    },
    getSU(input) {
      if (input == 1) {
        return "SU Option Available"
      }
      else {
        return "No SU Option Available"
      }
    },
    getProject(input) {
      if (input == 1) {
        return "Module consists of Project Work"
      }
      else {
        return "Module does not consist of Project Work"
      }
    },
  },
  computed: {
    filteredModules: function () {
      // console.log(modsInfo)
      // this.getData();
      this.loading = true;
      if (this.rating != '') {
        return this.modules.filter((module) => {
        // return module['.key'].match(this.search.toUpperCase())
        if (module.rating > this.rating) {
          this.loading = false;
          return module.rating
          }
      })
      }

      if (this.webcast != '') {
        return this.modules.filter((module) => {
        // return module['.key'].match(this.search.toUpperCase())
        //console.log(this.webcast===module.webcast.toString())
        if (module.webcast.toString() ===this.webcast) {
          this.loading = false;
          return module.webcast.toString().match(this.webcast)
          }
      })
      }

      if (this.SU != '') {
        return this.modules.filter((module) => {
        // return module['.key'].match(this.search.toUpperCase())
        if (module.su.toString() === this.SU) {
          this.loading = false;
          return module.su.toString().match(this.SU)
          }
      })
      }

      if (this.project != '') {
        return this.modules.filter((module) => {
        // return module['.key'].match(this.search.toUpperCase())
        if (module.project.toString() === this.project) {
          this.loading = false;
          return module.project.toString().match(this.project)
          }
      })
      }

      if (this.department != '') {
        console.log(this.department)
        return this.modules.filter((module) => {
          // return module['.key'].match(this.search.toUpperCase())
          this.loading = false;
          return module.Department.toUpperCase().match(this.department.toUpperCase())
      })
      }
      
      if (this.search != ''){
        return this.modules.filter((module) => {
          // return module['.key'].match(this.search.toUpperCase())
          this.loading = false;
          return module.ModuleCode.match(this.search.toUpperCase())
      })
      }

      return this.modules.filter((module) => {
        // return module['.key'].match(this.search.toUpperCase())
        this.loading = false;
        return module
    })
  }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
#show-modules {
  max-width: 800px;
  margin:0 auto;
}
.single-module {
  padding: 20px;
  margin: 20px 0;
  box-sizing: border-box;
  background: #eee;
}
table {
    width: 100%;
}
td {
    vertical-align: top;
}
.p-left {
    text-align: left;
}
.p-right {
    text-align:right;
}
.copy {
    visibility: hidden;
}
.copy, .p-right {
    white-space: nowrap;
}
input[type=text] {
  width: 100%;
  box-sizing: border-box;
  border: 2px solid #ccc;
  border-radius: 4px;
  font-size: 16px;
  background-color: white;
  background-position: 10px 10px; 
  background-repeat: no-repeat;
  padding: 12px 20px 12px 40px;
}
</style>