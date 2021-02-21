<template>
  <div class="card-front">
    <img class="card-front__image" src="../../assets/img/card.jpg" />
    <img class="card-front__symbol"  src="../../assets/img/mastercard.png" />
    <img class="card-front__chip" src="../../assets/img/chip.png" />
    <p class="card-front__number">{{ cardNumber }}</p>
    <div class="card-front__info">
      <p>Tarih</p>
      <p class="card-front__expires value">
        {{ expireMonth || "MM" }} / {{ (expireYear && sliceYear) || "YY" }}
      </p>
    </div>
    <div class="card-front__info left">
      <p class="value">{{ name || "İsim Soyisim" }}</p>
    </div>
  </div>
</template>

<script>
export default {
  props: ['cardNumber','expireMonth', 'expireYear', 'name' ],
  computed: {
    sliceYear() {
      return this.expireYear.toString().slice(2);
    }
  }
};
</script>