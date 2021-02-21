<template>
  <form-wizard ref="formwizard" @onComplete="onComplete">
    <tab-content title="Tarih" :selected="true">
      <b-form-group label="Check-in">
        <b-form-datepicker v-model="form.dateStart" :min="config.min" @input="onStartChange" placeholder="Giriş Tarihi Seç"></b-form-datepicker>
        <div v-if="hasError('dateStart')" class="invalid-feedback d-block">
          <div class="error" v-if="!$v.form.dateStart.required">Lütfen tarih seçiniz.</div>
        </div>
      </b-form-group>
      <b-form-group label="Check-out">
        <b-form-datepicker v-model="form.dateEnd" :min="config.endMin" placeholder="Çıkış Tarihi Seç"></b-form-datepicker>
        <div v-if="hasError('dateEnd')" class="invalid-feedback d-block">
          <div class="error" v-if="!$v.form.dateEnd.required">Lütfen tarih seçiniz.</div>
        </div>
      </b-form-group>
    </tab-content>
    <tab-content title="Oda"> 
      <div class="second-step">
        <ul class="d-flex flex-wrap list-unstyled alert alert-success">
            <li class="pr-5">Check-in: {{ form.dateStart }}</li>
            <li>Check-out: {{ form.dateEnd }}</li>
        </ul>
        <b-form-group label="Oda Tipi" >
          <b-form-radio-group v-slot="{ ariaType }" v-model="form.type">
              <b-form-radio v-model="form.type" :aria-describedby="ariaType" name="type" value="Standart">Standart</b-form-radio>
              <b-form-radio v-model="form.type" :aria-describedby="ariaType" name="type" value="Deluxe">Deluxe</b-form-radio>
              <b-form-radio v-model="form.type" :aria-describedby="ariaType" name="type" value="Suit">Suit</b-form-radio>
          </b-form-radio-group>
          <div v-if="hasError('type')" class="invalid-feedback d-block">
              <div class="error" v-if="!$v.form.type.required">Lütfen otel odası seçin.</div>
          </div>
        </b-form-group>
        <b-form-group label="Manzara Seçimi" v-if="form.type">
          <b-form-radio-group v-slot="{ ariaView }" v-model="form.view">
              <b-form-radio v-model="form.view" :aria-describedby="ariaView" name="view" value="Deniz">Deniz</b-form-radio>
              <b-form-radio v-model="form.view" :aria-describedby="ariaView" name="view" value="Kara">Kara</b-form-radio>
          </b-form-radio-group>
          <div v-if="hasError('view')" class="invalid-feedback d-block">
              <div class="error" v-if="!$v.form.view.required">Lütfen oda manzarası seçin.</div>
          </div>
        </b-form-group>
      </div>
    </tab-content>
    <tab-content title="Ödeme">
      <div class="card-form">
        <div class="card-form__inner">
            <div class="card-container mb-5">
                <div class="credit-card">
                    <div class="credit-card__inner" :class="{ 'show-back': showBack }">
                        <front
                            :cardNumber="cardNumber"
                            :expireMonth="form.expireMonth"
                            :expireYear="form.expireYear"
                            :name="form.name"
                        />
                        <back :cvv="cvv" />
                    </div>
                </div>
            </div>
            <b-form-group label="Kart Numarası" label-for="card-number">
                <input
                    v-imask="cardMasks"
                    @accept="onAcceptCardType"
                    :value="cardNumber"
                    autofocus
                    id="card-number"
                    class="form-control"
                    autocomplete="off"
                />
                <div v-if="hasError('cardNumber')" class="invalid-feedback d-block">
                    <div class="error" v-if="!$v.form.cardNumber.required">Lütfen geçerli kart numarası girin.</div>
                    <div class="error" v-if="!$v.form.cardNumber.minLength">Kart numarası 16 hane olmalıdır.</div>
                </div>
            </b-form-group>
             <b-form-group label="Kart Üzerindeki İsim" label-for="card-name">
                <input
                    id="card-name"
                    class="form-control"
                    v-model="form.name"
                    autocomplete="off"
                />
                <div v-if="hasError('name')" class="invalid-feedback d-block">
                    <div class="error" v-if="!$v.form.name.required">Lütfen isim girin.</div>
                </div>
            </b-form-group>
            <div class="form-row form-group">
                <div class="col-sm-8 mb-4">
                    <label for="card-month" class="card-input__label">Tarih</label>
                    <div class="d-flex">
                        <select class="form-control mr-2" id="card-month" v-model="form.expireMonth">
                          <option value disabled selected>Ay</option>
                          <option v-for="n in 12" :key="n" :value="10 > n ? '0' + n : n">
                              {{ 10 > n ? "0" + n : n}}
                          </option>
                        </select>
                        <select class="form-control" id="card-year" v-model="form.expireYear">
                          <option value disabled selected>Yıl</option>
                          <option v-for="(n, $index) in 12" :value="$index + currentYear" :key="n">
                              {{ $index + currentYear }}
                          </option>
                        </select>
                    </div>
                </div>
                <div class="col-sm-4">
                    <label for="card-cvv" class="card-input__label">CVV</label>
                    <input
                      type="text"
                      class="form-control"
                      id="card-cvv"
                      :value="cvv"
                      autocomplete="off"
                      v-imask="cvvMask"
                      @accept="onAcceptCvv"
                      @focus="showBack = true"
                      @blur="showBack = false"
                    />
                    <div v-if="hasError('cvv')" class="invalid-feedback d-block">
                        <div class="error" v-if="!$v.form.cvv.required">Lütfen güvenlik numarasını girin.</div>
                    </div>
                </div>
            </div>
        </div>
      </div>
    </tab-content>
  </form-wizard>
</template>

<script>
import FormWizard from './Wizard/FormWizard.vue';
import TabContent from './Wizard/TabContent.vue';
import ValidationHelper from './Wizard/ValidationHelper.vue';
import { required, minLength } from 'vuelidate/lib/validators';
import Front from "./Card/Front.vue";
import Back from "./Card/Back.vue";
import { IMaskDirective } from "vue-imask";

export default {
  name: 'Reservation',
  components: {
    FormWizard, TabContent, Front, Back
  },
  mixins: [ValidationHelper],
  data(){
    return {
      form: {
        dateStart: null,
        dateEnd: null,
        type: null,
        view: null,
        expireMonth: '',
        expireYear: '',
        name: '',
        cardNumber: '',
        cvv: '',
      },
      cardNumber: '',
      cvv: '',
      config: {
        min: new Date(),
        endMin: new Date(),
      },
      cardMasks : {
          mask: "0000 0000 0000 0000",
      },
      cvvMask : {
          mask: "000"
      },
      showBack: false,
      currentYear: new Date().getFullYear(),
      validationRules:[
        {
          dateStart: {required},
          dateEnd: {required} 
        },
        {
          type: {required},
          view: {required}
        },
        { 
          cardNumber: {required, minLength: minLength(19)}, 
          expireMonth: {required}, 
          expireYear: {required}, 
          name: {required},
          cvv: {required},
        },
      ]
    }
  },
  methods: {
      onComplete(){
          alert("Form Gönderildi!");
          console.log(this.form);
          localStorage.setItem("form", JSON.stringify(this.form));
          this.$refs.formwizard.changeStatus();
      },
      onAcceptCardType(e) {
          const maskRef = e.detail;
          this.cardNumber = maskRef.value;
          this.form.cardNumber = maskRef.value;
      },
      onAcceptCvv(e) {
          const maskRef = e.detail;
          this.cvv = maskRef.value;
          this.form.cvv = maskRef.value;
      },
      onStartChange(evt) {
        this.config.endMin =  new Date(evt);
      }
  },
  directives: {
      imask: IMaskDirective
  },
  mounted() {
    if(localStorage.getItem('form')){
      this.form = JSON.parse(localStorage.getItem('form'));
      this.config.endMin = JSON.parse(localStorage.getItem('form')).dateStart;
      this.cardNumber = JSON.parse(localStorage.getItem('form')).cardNumber;
      this.cvv = JSON.parse(localStorage.getItem('form')).cvv;
    }
  }
}
</script>