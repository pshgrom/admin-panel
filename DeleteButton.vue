<template>
  <button class="deleteButton" dense size="24px"></button>
</template>

<script>
  export default {
    name: "DeleteButton",
  }
</script>

<style lang='scss'>
   .deleteButton {
      display: block;
      background-color: #fff;
      border: none;
      width: 24px;
      height: 24px;
      cursor: pointer;
      background-image: url(../../public/icons/delete-24px.svg);
      transition: background-image 0.2s ease-in-out;

      &:hover {
         background-image: url(../../public/icons/delete_filled-24px.svg);
      }
   }
</style>