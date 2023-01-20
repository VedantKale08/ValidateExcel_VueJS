<template>
   <div class="header">
      FindError
   </div>
   <UploadExcel/>
</template>

<script>
import UploadExcel from './Components/UploadExcel.vue';


export default {
    name: "App",
    components: { UploadExcel }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@500&display=swap');

body {
   margin: 0;
   font-family: 'Poppins', sans-serif;
   background-color: #eee;
}
.header{
   display: flex;
   background-color: #3C6255;
   padding: 1rem;
   font-size: 30px;
   color: #EAE7B1;
   font-weight: bold;
}
</style>

