/* eslint-disable */
<template>
  <div>
    <h1>Module Recommendations</h1>
    <br>
    <p>Type the number of MCs you wish to take for each of the electives and click on 'GET MODULES' to get going!</p>
    <p>
      <b>*Do note that NUS requires students to take minimally 18 MCs per semester</b>
    </p>

    <v-container grid-list-md text-xs>
      <v-layout row wrap class="justify-center" >
        <v-flex xs3 sm6 md3>
          <label class="label">
            <b>Programme Electives:</b>
          </label>
          <v-text-field v-model="PE_MCs" label="Number of MCs" outline></v-text-field>
        </v-flex>
        <v-flex xs4 sm6 md3>
          <label class="label">
            <b>General Electives:</b>
          </label>
          <v-text-field v-model="GE_MCs" label="Number of MCs" outline></v-text-field>
        </v-flex>
        <v-flex xs4 sm6 md3>
          <label class="label">
            <b>Unrestricted Electives:</b>
          </label>
          <v-text-field v-model="UE_MCs" label="Number of MCs" outline></v-text-field>
        </v-flex>
        <v-flex xs4 sm4 text-xs>
          <div>
            <!-- {{this.loading}}<br> -->
            <v-btn block v-on:click="getModules()" :loading="loading">Get Modules</v-btn>
          </div>
        </v-flex>
      </v-layout>
    </v-container>

    <h3 class="header-text">
      <strong>Analysis</strong>
    </h3>

    <div class="analysis-text">
      <p>
        The following module recommendations are based on the past modules
        <b>Business Analytics</b> students
        took in their
        <b>2nd year</b>.
      </p>
      <h3>
        <b>Recommended Programme Elective(s):</b>
      </h3>
      <div>
        <b-list-group-item class="list" v-for="module in filtered_mods_PE" :key="module.id">
          <a href="#" @click="getDesc(module[0])">{{ module[0] }}</a>
        </b-list-group-item>
      </div>
      <h3>
        <b>Recommended General Elective(s):</b>
      </h3>
      <div>
        <b-list-group-item class="list" v-for="module in filtered_mods_GE" :key="module.id">
          <a href="#" @click="getDesc(module[0])">{{ module[0] }}</a>
        </b-list-group-item>
      </div>

      <h3>
        <b>Recommended Unrestricted Elective(s):</b>
      </h3>
      <div>
        <b-list-group-item class="list" v-for="module in filtered_mods_UE" :key="module.id">
          <a href="#" @click="getDesc(module[0])">{{ module[0] }}</a>
        </b-list-group-item>
      </div>
      <br>
      <div class="statistics">
        <div class="statistics-header">
          <h3 class="statistics-text">
            <strong>Brief Module Description</strong>
          </h3>
        </div>
        <p>
          <strong>{{recoMods.title}}</strong>
        </p>
        <p>Department: {{recoMods.Department}}</p>
        <p>{{recoMods.Description}}</p>
      </div>
    </div>
    <div class="text-xs-center"></div>
     <v-btn color="primary" to="/choose">back </v-btn>
  </div>
</template>

<script>
/*eslint-disable */
import { db } from "../firebase";
export default {
  name: "Choices",
  data: () => ({
    data: null,
    loading: false,
    descData: null,
    count: 0,
    PE_MCs: null,
    GE_MCs: null,
    UE_MCs: null,
    filtered_mods_PE: [],
    filtered_mods_GE: [],
    filtered_mods_UE: [],
    recoMods: { title: "", Department: "", MC: "", Description: "" },
    mods_taken: [
      "BT1101",
      "CS1010S",
      "MA1101R",
      "EC1301",
      "GER1000",
      "MA1521",
      "MKT1705X",
      "GEQ1000",
      "BT2101",
      "CS1020"
    ],
    prereq : {'BT2101':['CS1010S', 'MA1521', 'BT1101'], 'BT2102':['CS1010S', 'BT1101'], 'CS2010': ['CS1020'], 'ST2334': ['MA1521'], 'EC2101': ['EC1301']},
    preclu : {'GET1004': ['GEK1531'], 'GEH1025':['GEK1522'], 'CS2010':['CS2020', 'CS2030', 'CS2040', 'CS2040C'], 'IS2102': ['ES2002', 'ES2007D', 'CS2101', 'CS2103T', 'ES1601'], 
    'ST2334':['ST1131', 'ST1232', 'ST2131', 'MA2216', 'CE2407', 'EC2231', 'EC2303', 'PR2103', 'DSC2008'], 'MKT1705X': ['MKT1003'] }
  }),
  props: {
    source: String
  },
  methods: {
    async getData() {
      this.data = await db
        .ref("/shannon/sorted")
        .once("value")
        .then(function(snapshot) {
          var d = snapshot.val();
          
          return d;
        });
    },
    async getModules() {
      this.loading = true;
      await this.getData();
      this.loading = false;
      //console.log(this.data);
      var datakeys = Object.keys(this.data);
      //console.log(datakeys);
      console.log("here")
      for(var i=0; i<this.data['PE'].length; i++){
        if(!(this.data['PE'][i] in this.mods_taken)){
          var met = false;
          if(this.prereq[this.data['PE'][i][0]] != null){
            for(var j=0; j<this.prereq[this.data['PE'][i][0]].length; j++){
              if (this.mods_taken.includes(this.prereq[this.data['PE'][i][0]][j])){
                met = true;
              }else{
                met = false;
              }
            }
            if(met == true){
              if(this.preclu[this.data['PE'][i][0]] != null){
                for(var k=0; k< this.preclu[this.data['PE'][i][0]].length; k++){
                  if (this.mods_taken.includes(this.preclu[this.data['PE'][i][0]][k])){
                    met = false;
                  }else{
                    met = true;
                  }
                }
              }
            }
          }else{
            if(this.preclu[this.data['PE'][i][0]] != null){
              for(var k=0; k< this.preclu[this.data['PE'][i][0]].length; k++){
                if (this.mods_taken.includes(this.preclu[this.data['PE'][i][0]][k])){
                  met = false;
                }else{
                  met = true;
                }
              }
            }
          } 
          if(met == true){
            this.filtered_mods_PE.push(this.data['PE'][i])
          }
        }
      }
      for(var i=0; i<this.data['GE'].length; i++){
        console.log("did it enter GE")
        if(!(this.data['GE'][i] in this.mods_taken)){
          console.log("not taken")
          var met = false;
          if(this.prereq[this.data['GE'][i][0]] != null){
            for(var j=0; j<this.prereq[this.data['GE'][i][0]].length; j++){
              console.log("then what is wrong")
              if (this.mods_taken.includes(this.prereq[this.data['GE'][i][0]][j])){
                met = true;
              }else{
                met = false;
              }
            }
            if(met == true){
              if(this.preclu[this.data['GE'][i][0]] != null){
                for(var k=0; k< this.preclu[this.data['GE'][i][0]].length; k++){
                  if (this.mods_taken.includes(this.preclu[this.data['GE'][i][0]][k])){
                    met = false;
                  }else{
                    met = true;
                  }
                }
              }
            }
          }else{
            if(this.preclu[this.data['GE'][i][0]] != null){
              for(var k=0; k< this.preclu[this.data['GE'][i][0]].length; k++){
                if (this.mods_taken.includes(this.preclu[this.data['GE'][i][0]][k])){
                  met = false;
                }else{
                  met = true;
                }
              }
            }
          }  
          if(met == true){
            console.log("did it come here?")
            this.filtered_mods_GE.push(this.data['GE'][i])
          }
        }
      }for(var i=0; i<this.data['UE'].length; i++){
        if(!(this.data['UE'][i] in this.mods_taken)){
          var met = false;
          if(this.prereq[this.data['UE'][i][0]] != null){
            for(var j=0; j<this.prereq[this.data['UE'][i][0]].length; j++){
              if (this.mods_taken.includes(this.prereq[this.data['UE'][i][0]][j])){
                met = true;
              }else{
                met = false;
              }
            }
            if(met == true){
              if(this.preclu[this.data['UE'][i][0]] != null){
                for(var k=0; k< this.preclu[this.data['UE'][i][0]].length; k++){
                  if (this.mods_taken.includes(this.preclu[this.data['UE'][i][0]][k])){
                    met = false;
                  }else{
                    met = true;
                  }
                }
              }
            }
          }else{
            if(this.preclu[this.data['UE'][i][0]] != null){
              for(var k=0; k< this.preclu[this.data['UE'][i][0]].length; k++){
                if (this.mods_taken.includes(this.preclu[this.data['UE'][i][0]][k])){
                  met = false;
                }else{
                  met = true;
                }
              }
            }
          } 
          if(met == true){
            this.filtered_mods_UE.push(this.data['UE'][i])
          }
        }
      }
      this.filtered_mods_PE = await this.filtered_mods_PE.slice(0, this.PE_MCs / 4);
      this.filtered_mods_GE = await this.filtered_mods_GE.slice(0, this.GE_MCs / 4);
      console.log("filtered_mods_GE")
      console.log(filtered_mods_GE)
      this.filtered_mods_UE = await this.filtered_mods_UE.slice(1, 1 + this.UE_MCs / 4);
      console.log(this.filtered_mods_UE)
      this.loading = false;

      
    },
    async getDescData() {
      this.descData = await db
        .ref("/mods_info/data")
        .once("value")
        .then(function(snapshot) {
          // takes a snapshot of the data at that time
          var d = snapshot.val();
          // console.log("whewww")
          /*eslint-disable */
          console.log(d);
          return d;
        });
    },
    async getDesc(code) {
      await this.getDescData();
      // check if the code did get passed into the function
      console.log(String(code));
      // datakeys is the index of each child node in the mod parent node
      var datakeys = Object.keys(this.descData);
      console.log(datakeys);
      // here i am testing for the first child and seeing if it produces the module code
      console.log(this.descData[datakeys[0]]["ModuleCode"]);
      //var modules = [];
      //console.log(modules);
      for (var i = 0; i < this.descData.length; i++) {
        var key = datakeys[i];
        //console.log("key: " + key);
        if (this.descData[key]["ModuleCode"] == String(code)) {
          console.log(this.descData[key]);
          console.log(this.descData[key].Department);
          //var modinfo = this.data[key].Department
          this.recoMods.Department = this.descData[key].Department;
          this.recoMods.title = code;
          console.log(this.recoMods.title);
          this.recoMods.Description = this.descData[key].ModuleDescription;
          console.log(this.recoMods.Description);

          //this.recoMods[0] = this.data[key];
        }
      }
      console.log(this.recoMods.Department);
      return this.recoMods;
    }
  }
};
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.header h3 {
  float: left;
}
.header button-right {
  float: right;
}
.reco {
  border: solid;
  border-color: black;
  margin-left: 50px;
  margin-right: 50px;
  display: inline;
  overflow: auto;
}
.list:hover {
  background-color: gray;
}

.analysis {
  border: solid;
  border-color: rgb(148, 148, 148);
  display: inline-block;
  vertical-align: top;
  float: left;
  width: 49%;
  height: 460px;
}

.statistics {
  border: solid;
  border-color: rgb(148, 148, 148);
  display: inline-block;
  vertical-align: top;
  float: center;
  text-align: center;
  width: 49%;
  height: 460px;
}

.statstics-text {
  display: inline-block;
}

.statistics-img {
  display: inline-block;
}

.header-image {
  display: inline-block;
}

.header-text {
  display: inline-block;
}
</style>