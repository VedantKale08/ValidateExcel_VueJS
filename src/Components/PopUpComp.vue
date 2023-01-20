<template>
   
      <div class="popDiv" @click.stop="doSomething">
         Previous Data
         <p class="p">{{ temp }}</p>
         Add Correct Data
         <input type="text" class="editText" placeholder="Enter Name with (Dr.)" v-on:change="enter($event)"
            v-show="popUpName" required>
         <input type="email" class="editText" placeholder="Enter Email" v-on:change="enter($event)" v-show="popUpEmail"
            required>
         <input type="number" class="editText" placeholder="Enter Phone Number" v-on:change="enter($event)"
            v-show="popUpPhone" required maxlength="10">

         <select class="p" v-show="popUpCountry" v-model="datas">
            <option v-for="(i, index) in countryDropArray" :key="index" :value='i.value'>{{ i.label }}</option>
         </select>

         <select class="p" v-show="popUpState" v-model="datas">
            <option v-for="(i, ind) in stateObject[index - 1]" :key="ind" :value='i'>{{ i }}</option>
         </select>

         <select class="p" v-show="popUpCity" v-model="datas">
            <option v-for="(i, ind) in cityObject[index - 1]" :key="ind" :value='i'>{{ i }}</option>
         </select>
         <div style="text-align: left;color: red;margin-bottom: 1em;width: 80%; font-size: 10px;">{{ error }}</div>
         <button class="save" v-on:click="saveData" type="submit">
            Save
         </button>
      </div>

</template>
<script>

export default {
   name: 'PopUpComp',
   props: {
         False:Function, 
         popup:Boolean,
         temp:String,
         enter:Function,
         popUpName:Boolean,
         popUpEmail:Boolean,
         popUpPhone:Boolean,
         popUpState:Boolean,
         popUpCity:Boolean,
         popUpCountry:Boolean,
         data:String,
         countryDropArray:Array,
         cityObject:Array,
         stateObject:Array,
         index:Number,
         saveData:Function,
         error:String
   },
   data(){
      return{
         datas:this.data
      }
   }
}
</script>