<template>
   <div>
      <table>
         <tbody>
            <tr v-for="(i) in currentData.length" :key="i">
               <td v-for="(j, index) in currentData[i-1]" :key="index">{{ j }}</td>
            </tr>
         </tbody>
      </table>
      <div style="display: flex; color: blue;">
         <div style="width: 30px;border: 1px solid black; text-align: center;cursor: pointer;" v-for="i in pageNumber"
            :key="i">
            <a href="!#" @click="paginate(i)">{{ i }}</a>
         </div>
      </div>
   </div>


</template>
<script>

export default {
   name: 'PopUpComp',
   data() {
      return {
         current: 1,
         dataPerPage: 6,
         indexOfLastPage: 0,
         indexOfFirstPage: 0,
         currentData: [],
         pageNumber: [],

         data: [
            [1, 2, 3, 4, 5],
            [1, 2, 3, 4, 5],
            [1, 2, 3, 4, 5],
            [1, 2, 3, 4, 5],
            [1, 2, 3, 4, 5],
            [1, 2, 3, 4, 5],
            [1, 3, 3, 44, 5],
            [1, 2, 444, 4, 5],
            [1, 2, 3, 4, 5],
            [1, 2, 32, 4, 5],
            [111, 2, 3, 4, 5],
            [111, 2, 3, 4, 5],
            [111, 2, 3, 4, 5],
         ],
      }
   },
   methods: {
      paginate(number) {
         this.current = number
         this.indexOfLastPage = this.current * this.dataPerPage;
         this.indexOfFirstPage = this.indexOfLastPage - this.dataPerPage;
         this.currentData = this.data.slice(this.indexOfFirstPage, this.indexOfLastPage);
      }
   },

   mounted() {
      this.indexOfLastPage = this.current * this.dataPerPage;
      this.indexOfFirstPage = this.indexOfLastPage - this.dataPerPage;
      this.currentData = this.data.slice(this.indexOfFirstPage, this.indexOfLastPage);

      for (let i = 1; i <= Math.ceil(this.data.length / this.dataPerPage); i++) {
         this.pageNumber.push(i)
      }
   }
}
</script>