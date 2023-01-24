<template>
   <div class="main">
      <p class="h-text">Upload Your Excel File</p>
      <input type="file" class="excel" v-on:change="addFile($event)"
         accept=".csv,application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel">
      <br />
      <br />
      <div v-if="isLoading"><half-circle-spinner :animation-duration="1000" :size="60" color="#61876E" /> </div>
      <div class="file" v-show="show">
         <div>
            <p>Affected rows : {{ issueCount }}</p>
            <p>Not affected rows : {{ okCount }}</p>
         </div>
         <table>
            <thead>
               <tr>
                  <th v-for="i in arrayData[0]" :key="i + 1">{{ i }}</th>
               </tr>
            </thead>
            <tbody>
               <tr v-for="j in arrayData.length" :key="j">
                  <td v-for="i, index in currentData[j - 1]" :key="index">
                     <p>{{ i }} <br /><a class="error" v-on:click="edit(i, j)">{{ displayError(i, j) }}</a></p>
                  </td>
               </tr>
            </tbody>
         </table>
         <div class="options">
            <button v-for="i in pageNumber" :key="i" class="numbers" @click="paginate(i)" :disabled="current == i">
               <a>{{ i }}</a>
            </button>
         </div>
         <div class="buttDiv">
            <button class="export" v-on:click="downloadFile">Save</button>
         </div>
      </div>

      <div class="blur" v-if="isLoading1">
         <half-circle-spinner :animation-duration="1000" :size="60" color="#61876E" />
      </div>
 
      <div class="blur" v-on:click="False" v-show="popUp">
         <div class="popDiv" @click.stop="doSomething">
            Previous Data
            <p class="p">{{ temp }}</p>
            Add Correct Data
            <input type="text" class="editText" placeholder="Enter Name with (Dr.)" v-on:change="enter($event)"
               v-show="popUpName" required>
            <input type="email" class="editText" placeholder="Enter Email" v-on:change="enter($event)"
               v-show="popUpEmail" required>
            <input type="number" class="editText" placeholder="Enter Phone Number" v-on:change="enter($event)"
               v-show="popUpPhone" required maxlength="10">

            <select class="p" v-show="popUpCountry" v-model="data">
               <option v-for="(i, index) in countryDropArray" :key="index" :value='i.value'>{{ i.label }}</option>
            </select>

            <select class="p" v-show="popUpState" v-model="data">
               <option v-for="(i, ind) in stateObject[index - 1]" :key="ind" :value='i'>{{ i }}</option>
            </select>

            <select class="p" v-show="popUpCity" v-model="data">
               <option v-for="(i, ind) in cityObject[index - 1]" :key="ind" :value='i'>{{ i }}</option>
            </select>
            <div style="text-align: left;color: red;margin-bottom: 1em;width: 80%; font-size: 10px;">{{ error }}</div>
            <button class="save" v-on:click="saveData" type="submit">
               Save
            </button>
         </div>
      </div>
   </div>
</template>

<script>
import * as XLSX from "xlsx";
import { State } from 'country-state-city';
import { City } from 'country-state-city';
import CountryCodes from 'country-codes-list'
import { HalfCircleSpinner } from 'epic-spinners'
export default {
   name: 'UploadExcel',
   components: {
      HalfCircleSpinner,
   },
   data() {
      return {
         file: File,
         file1: File,
         arrayData: [],       //data in row
         show: false,         //shows and hide table
         isLoading: false,
         isLoading1: false,
         keyPair_Data: [],    //data in key-value
         checkData: [],       //all errors
         desc: [],            //all errors description
         checkInd: [],        //index of errors
         popUp: false,        // --\
         popUpName: false,    //    \
         popUpEmail: false,   //     \
         popUpPhone: false,   //      --  pop up
         popUpCity: false,    //     / 
         popUpState: false,   //    /
         popUpCountry: false, // --/
         temp: null,          //previous data before update
         data: null,          //input data to update
         error: null,
         errorCounter: 0,
         issueCount: 0,       //affected rows
         okCount: 0,          //not affected rows
         local: [],           //country calling code array ['91','376']
         index: 0,            //row no of updating element
         allStateList: [],    //state array
         stateArray: [],      //temp : state array names in object
         stateObject: [],     //all state array names in object
         stateCodeArray: [],  //temp : codes of state
         stateCodeObject: [], //all codes of state
         allCityList: [],     //city array
         key: [],             //country code array ['IN','US']
         stateNames: [],      //names of all state
         cityArray: [],       //temp : city array names in object
         cityObject: [],      //all city array names in object
         code: [],
         keyy: null,          //headers of row 
         countryDropArray: [],//dropdown array of country

         current: 1,          //current page
         dataPerPage: 10,     //no of data per page
         indexOfLastPage: 0,  //last index of data of page
         indexOfFirstPage: 0, //First index of data of page
         currentData: [],     //data on current page 
         pageNumber: [],      //all page numbers
      }
   },
   methods: {
      enter(e) {
         this.data = e.target.value
      },
      False() {
         this.popUp = false
         this.isLoading1 = true
         setTimeout(() => {
            this.isLoading1 = false
         }, 1000)
         this.popUpName = false
         this.popUpEmail = false
         this.popUpPhone = false
         this.popUpCity = false
         this.popUpState = false
         this.popUpCountry = false
         this.error = ''
      },
      addFile(e) {
         this.arrayData = []
         this.show = false;
         this.isLoading = true;
         setTimeout(() => {
            this.isLoading = false
            this.show = true
         }, 2000)

         this.file = e.target.files[0];
         //fileReader function
         const reader = new FileReader();
         reader.onload = (e) => {
            const data = e.target.result;
            //methods
            const wb = XLSX.read(data, { type: 'binary' });
            const sname = wb.SheetNames[0];
            const ws = wb.Sheets[sname];
            this.arrayData = XLSX.utils.sheet_to_json(ws, { header: 1 })
            
         }
         reader.readAsBinaryString(this.file);
      },
      snake_case_string(str) {
         return str && str.match(/[A-Z]{2,}(?=[A-Z][a-z]+[0-9]*|\b)|[A-Z]?[a-z]+[0-9]*|[A-Z]|[0-9]+/g)
            .map(s => s.toLowerCase())
            .join('_');
      },
      validate() {
         this.checkData = []
         this.desc = []
         this.checkInd = []
         this.stateCodeArray = [];

         this.keyPair_Data.forEach((value) => {
            Object.keys(value).map(i => {
               if (i == 'name') {
                  if (typeof (value[i]) != typeof ('a') || value[i].match(' ')) {
                     this.checkData.push(value[i])
                     this.desc.push(value[i] + " is invalid")
                     this.checkInd.push(value.no)
                  }
               }
               const myCountryCodesObject = CountryCodes.customList('countryCode', '{countryCallingCode}')
               let obj = {}
               obj = myCountryCodesObject;

               if (i == 'country_code') {
                  if (!Object.values(obj).includes(value[i].toString())) {
                     this.checkData.push(value[i])
                     this.desc.push(value[i] + " is not a country code")
                     this.checkInd.push(value.no)
                  }
               }


               if (i == 'state') {
                  this.stateObject = []

                  for (let j = 0; j < this.allStateList.length; j++) {
                     this.stateArray = [];
                     for (let k = 0; k < this.allStateList[j].length; k++) {
                        this.stateArray.push(this.allStateList[j][k].name)
                        if (!this.stateNames.includes(this.allStateList[j][k].name)) {
                           this.stateNames.push(this.allStateList[j][k].name)
                           this.code.push(this.allStateList[j][k].isoCode)
                        }
                     }
                     this.stateObject.push(this.stateArray)

                  }

                  for (let j = 0; j < this.stateNames.length; j++) {
                     if (value[i] === this.stateNames[j]) {
                        this.stateCodeArray.push(this.code[j])
                     }
                     else if (!this.stateNames.includes(value[i])) {
                        this.stateCodeArray.push('')
                        break;
                     }
                  }

                  if (!Object.values(this.stateObject[value.no - 1]).includes(value[i])) {
                     this.checkData.push(value[i])
                     this.desc.push(value[i] + " is not a state")
                     this.checkInd.push(value.no)
                  }

               }

               if (i == 'phone_number') {
                  if (value[i].toString().length != 10 || !/[0-9]+[0-9]+[0-9]+[0-9]+[0-9]+[0-9]+[0-9]+[0-9]+[0-9]+[0-9]/.test(value[i])) {
                     this.checkData.push(value[i])
                     this.desc.push(value[i] + " : only 10 digits are allowed")
                     this.checkInd.push(value.no)
                  }
               }
               if (i == 'email') {
                  if (!(value[i].match(/^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*$/))) {
                     this.checkData.push(value[i])
                     this.desc.push(value[i] + " : invalid email format")
                     this.checkInd.push(value.no)

                  }
               }
               if (i == 'username') {
                  if (!(value[i].match(/^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*$/))) {
                     this.checkData.push(value[i])
                     this.desc.push(value[i] + " : invalid username")
                     this.checkInd.push(value.no)
                  }
               }
            })
         })
      },
      displayError(j, ind) {
         let k = 10
         for (let j = 1; j <= this.currentData.length; j++) {
            if (this.current == j) {
               k = 10 * j - 10
               break;
            }
         }
         for (let i = 1; i <= 10; i++) {
            if (ind == i && this.current != 1) {
               ind = ind + k
               break;
            }
         }

         for (let c = 0; c < this.checkData.length; c++) {
            if (j == this.checkData[c] && ind == this.checkInd[c]) {
               return "  🔴" + this.desc[c]
            }
         }
      },
      cl(i, j) {
         this.popUp = false
         this.isLoading1 = true
         setTimeout(() => {
            this.isLoading1 = false
            this.popUp = true
            this.edit(i, j)
         }, 2000)
      }
      ,
      edit(i, index) {
         this.popUp = false
         this.isLoading1 = true
         setTimeout(() => {
            this.isLoading1 = false
            this.popUp = true
         }, this.temp != null)
         this.temp = i;
         this.index = index;

         let k = 10
         for (let j = 1; j <= this.currentData.length; j++) {
            if (this.current == j) {
               k = (10 * j) - 10
               break;
            }
         }
         for (let i = 1; i <= 10; i++) {
            if (index == i && this.current != 1) {
               this.index = index + k
               index = index + k
               break;
            }
         }

         for (let k = 0; k < this.keyPair_Data.length; k++) {
            Object.keys(this.keyPair_Data[k]).map(key => {
               if (this.keyPair_Data[k][key].toString() === i.toString()) {
                  if (this.keyPair_Data[k]['no'].toString() === index.toString()) {
                     this.keyy = key

                     if (key == 'name') {
                        this.popUpName = true
                     }
                     if (key == 'email') {
                        this.popUpEmail = true
                     }
                     if (key == 'phone_number') {
                        this.popUpPhone = true
                     }
                     if (key == 'city') {
                        this.popUpCity = true
                     }
                     if (key == 'state') {
                        this.popUpState = true
                     }
                     if (key == 'country_code') {
                        this.popUpCountry = true
                     }
                  }
               }
            });
         }

      },
      saveData() {
         if (this.keyy == 'name') {
            if (!this.data) {
               this.error = "Name is Required"
            }
            else if (typeof (this.data) != typeof ('a') || this.data.match(' ')) {
               this.error = 'Only First Name is required'
            }
            else if (this.data.match(/\d/)) {
               this.error = 'Name must be String'
            }
            else {
               this.error = ''
               this.save()
            }
         }
         else if (this.keyy == 'email') {
            if (!this.data) {
               this.error = "Email is Required"
            }
            else if (this.data.match(!/^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*$/)) {
               this.error = 'Email Id is invalid'
            }
            else {
               this.error = ''
               this.save()
            }
         }
         else if (this.keyy == 'phone_number') {
            if (!this.data) {
               this.error = "Phone Number is Required"
            }
            else if (!this.data.match(/\d/)) {
               this.error = 'Phone Number is invalid'
            }
            else if (!this.data.length != 0) {
               this.error = 'Phone Number must be of 10 digits'
            }
            else {
               this.error = ''
               this.save()
            }
         }
         else if (this.keyy == 'city') {
            if (!this.data) {
               this.error = "City is required is Required"
            }
            else {
               this.error = ''
               this.save()
            }
         }
         else if (this.keyy == 'state') {
            if (!this.data) {
               this.error = "State is required is Required"
            }
            else {
               this.error = ''
               this.save()
            }
         }
         else if (this.keyy == 'country_code') {
            if (!this.data) {
               this.error = "Country code is required is Required"
            }
            else {
               this.error = ''
               this.country = this.data
               this.save()
            }
         }


      },
      save() {
         for (let i = 1; i < this.arrayData.length; i++) {

            for (let j = 1; j < this.arrayData.length; j++) {
               if (this.arrayData[i][j] == this.temp && this.arrayData[i][0] == this.index) {
                  this.arrayData[i][j] = this.data
                  
                  this.popUp = false
                  this.isLoading1 = true
                  setTimeout(() => {
                     this.isLoading1 = false
                  }, 1000)

                  this.data = null;
                  this.popUp = false
                  this.popUpName = false
                  this.popUpEmail = false
                  this.popUpPhone = false
                  this.popUpCity = false
                  this.popUpState = false
                  this.popUpCountry = false
                  break;
               }
            }
         }
      },
      downloadFile() {
         const data = this.keyPair_Data;
         var blob = new Blob([data], {
            type: 'application/csv'
         });
         var fileOfBlob = new File([blob], 'myFile.csv');
         let formData = new FormData();
         formData.append('upload', fileOfBlob, 'file.csv')
         fetch('http://localhost:5000/upload', {
            method: 'post',
            body: formData
         })
      },
      paginate(number) {
         this.current = number
         console.log(this.current);
         this.indexOfLastPage = this.current * this.dataPerPage;
         this.indexOfFirstPage = this.indexOfLastPage - this.dataPerPage;
         this.currentData = this.arrayData.slice(this.indexOfFirstPage, this.indexOfLastPage);
      }
   },
   beforeUpdate() {
      this.local = []
      let headers = this.arrayData[0];
      let final_data = [];

      for (let i = 1; i < this.arrayData.length; i++) {
         let data = this.arrayData[i];
         let object = {};
         Object.keys(headers).map(j => {
            object[this.snake_case_string(Object.values(headers)[j])] = data[j]
         })
         final_data.push(object);
      }
      this.keyPair_Data = final_data;

      this.pageNumber = []
      this.indexOfLastPage = this.current * this.dataPerPage;
      this.indexOfFirstPage = this.indexOfLastPage - this.dataPerPage;
      this.currentData = this.arrayData.slice(this.indexOfFirstPage + 1, this.indexOfLastPage + 1);
      for (let i = 1; i <= Math.ceil((this.arrayData.length - 1) / this.dataPerPage); i++) {
         this.pageNumber.push(i)
      }

      this.keyPair_Data.forEach((value) => {
         Object.keys(value).map(i => {
            if (i == 'country_code') {
               this.local.push(value[i])
            }
         })
      })

      let obj = {}
      const myCountryCodesObject = CountryCodes.customList('countryCode', '{countryCallingCode}')
      obj = myCountryCodesObject;

      this.allStateList = []
      Object.values(this.local).map((v) => {
         var key = Object.keys(obj)[Object.values(obj).indexOf(v.toString())]
         this.allStateList.push(State.getStatesOfCountry(key?.toString()))
      })

      this.validate()

      this.allCityList = []
      this.key = []
      Object.values(this.local).map((v) => {
         this.key.push(Object.keys(obj)[Object.values(obj).indexOf(v.toString())])
      })

      this.stateCodeArray.forEach((s, i) => {
         this.allCityList.push(City.getCitiesOfState(this.key[i]?.toString(), s.toString()));
      })

      this.cityObject = []
      for (let j = 0; j < this.allCityList.length; j++) {
         this.cityArray = [];
         for (let k = 0; k < this.allCityList[j].length; k++) {
            this.cityArray.push(this.allCityList[j][k].name)
         }
         this.cityObject.push(this.cityArray)
      }

      this.keyPair_Data.forEach((value) => {
         Object.keys(value).map(i => {
            if (i == 'city') {
               if (!Object.values(this.cityObject[value.no - 1]).includes(value[i])) {
                  this.checkData.push(value[i])
                  this.desc.push(value[i] + " is not a city of provided country code or state")
                  this.checkInd.push(value.no)
               }
            }
         })
      })

      this.issueCount = 0
      for (let i = 1; i < this.arrayData.length; i++) {
         for (let j = 0; j < this.arrayData[i].length; j++) {
            for (let k = 0; k < this.checkData.length; k++) {
               if (this.arrayData[i][j] == this.checkData[k] && this.arrayData[i][0] == this.checkInd[k]) {
                  this.errorCounter++
                  break;
               }
            }
         }
         if (this.errorCounter != 0) {
            this.issueCount++
         }
         this.errorCounter = 0
      }
      this.okCount = this.keyPair_Data.length - this.issueCount

      this.countryDropArray = []
      const countryCode = CountryCodes.customList('countryCode', '{countryCallingCode}')
      Object.values(countryCode).map((v) => {
         this.countryDropArray.push({ label: v, value: v })
      })
   },
}
</script>

<style>
.main {
   display: flex;
   flex-direction: column;
   align-items: center;
   justify-content: center;
   padding: 1rem;
}

.h-text {
   font-size: 30px;
   font-weight: bold;
}

.excel {
   content: 'Select some files';
   color: #fff;
   border-radius: 10px;
   display: flex;
   width: 15rem;
   padding: 1rem;
   outline: none;
   cursor: pointer;
   font-weight: 700;
   font-size: 15px;
   background-color: #61876E;
   box-shadow: 0 3px 10px rgb(0 0 0 / 0.2);
}

.excel:hover {
   opacity: 0.7;
}

.excel::-webkit-file-upload-button {
   display: none;
}

.file {
   width: 80%;
}

th {
   background-color: #3C6255;
   color: #EAE7B1;
   font-size: 20px;
}

td {
   border-bottom: 1px solid black;
   width: 89px;
   text-align: center;
   height: 5rem;
   background-color: #fff;
}

table {
   width: 100%;
   border-collapse: collapse;
   border-radius: 3rem;
   border: none;
   box-shadow: 0 3px 10px rgb(0 0 0 / 0.2);
}

.error {
   font-size: 10px;
   color: red;
   cursor: pointer;

}

.blur {
   position: fixed;
   top: 0;
   right: 0;
   left: 0;
   bottom: 0;
   backdrop-filter: blur(5px);
   display: flex;
   justify-content: center;
   align-items: center;
}

.popDiv {
   width: 20%;
   height: 20rem;
   background-color: #fff;
   box-shadow: 0 3px 10px rgb(0 0 0 / 0.2);
   border-radius: 30px;
   display: flex;
   flex-direction: column;
   align-items: center;
   justify-content: center;
   padding: 2rem;
}

.p {
   font-size: 20px;
   border: 1px solid grey;
   width: 80%;
   padding: 0.5rem;
   border-radius: 10px;
}

.editText {
   font-size: 20px;
   border: 1px solid grey;
   width: 80%;
   -webkit-margin-after: 0px;
   margin-block-end: 0px;
   margin-block-start: 1em;
   margin-inline-start: 0px;
   margin-inline-end: 0px;
   padding: 0.5rem;
   border-radius: 10px;
}

.save {
   color: #fff;
   border-radius: 10px;
   display: flex;
   width: 8rem;
   text-align: center;
   padding: 0.5rem;
   border: none;
   outline: none;
   cursor: pointer;
   font-weight: 700;
   font-size: 15px;
   background-color: #61876E;
   box-shadow: 0 3px 10px rgb(0 0 0 / 0.2);
   display: flex;
   justify-content: center;
}

.save:hover {
   opacity: 0.7;
}

.export {
   color: #fff;
   border-radius: 10px;
   display: flex;
   width: 10rem;
   text-align: center;
   padding: 1rem;
   border: none;
   outline: none;
   cursor: pointer;
   font-weight: 700;
   font-size: 15px;
   background-color: #61876E;
   box-shadow: 0 3px 10px rgb(0 0 0 / 0.2);
   display: flex;
   justify-content: center;
   margin-top: 2rem;
}

.export:hover {
   opacity: 0.7;
}

.buttDiv {
   display: flex;
   justify-content: end;
}

.options {
   display: flex;
   justify-content: center;
   align-items: center;
   margin-top: 2rem;
   padding: 2rem;
}

.numbers {
   padding: 20px;
   outline: none;
   cursor: pointer;
   background-color: #fff;
   box-shadow: 0 3px 10px rgb(0 0 0 / 0.2);
   color: #61876E;
   border-radius: 10px;
   margin-left: 20px;
   text-align: center;
   border: none;
}

.numbers:hover {
   background-color: #61876E;
   color: #fff;
}
</style>