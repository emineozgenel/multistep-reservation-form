<template>
  <div class="card-back">
    <img class="card-back__image" src="../../assets/img/card.jpg" />
    <div class="card-back__stripe"></div>
    <div class="card-back__cvv">
      {{ cvv }}
    </div>
  </div>
</template>

<script>
export default {
  props: {
    cvv: String
  }
};
</script>

