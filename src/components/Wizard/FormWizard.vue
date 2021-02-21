<template>
  <div class="vue-step-wizard">
    <div class="step-header">
      <ul class="step-pills">
          <li @click.prevent.stop="selectTab(index)" class="step-item" :class="{ 'active': tab.isActive, 'validated': tab.isValidated }" v-for="(tab, index) in tabs" v-bind:key="`tab-${index}`">
              <a class="step-link" href="#">
                <span class="tab-status">
                  <span :class="{'d-none': tab.isValidated }">{{ index+1 }}</span>
                  <b-icon :class="{'d-block': tab.isValidated }" stacked icon="check"></b-icon>
                </span> 
                <span>{{ tab.title }}</span>
              </a>
          </li>
      </ul>
    </div>
    <div class="step-body">
      <form>
          <slot></slot>
      </form>
    </div>
    <div class="step-footer">
      <div class="btn-group" role="group">
        <template v-if="!submitSuccess">
          <button @click="previousTab" v-if="currentTab > 0" class="step-button step-button-previous">GERİ</button>
          <button @click="nextTab" v-if="currentTab < totalTabs - 1" class="step-button step-button-next">DEVAM ET</button>
          <button @click="onSubmit" v-if="currentTab === totalTabs - 1" class="step-button step-button-submit">GÖNDER</button>
        </template>
        <template v-else>
          <button @click="reset" class="step-button step-button-reset">Reset</button>
        </template>
      </div>
    </div>
  </div>
</template>
<script>
import { store } from "./store.js";
export default {
  name: 'form-wizard',
  data(){
      return{
          tabs: [],
          currentTab : 0,
          totalTabs : 0,
          storeState: store.state,
          submitSuccess : false,
          progress: 0,
          isValidationSupport: false
      }
  },
  mounted(){
          this.tabs = this.$children;
          this.totalTabs = this.tabs.length;
          this.currentTab = this.tabs.findIndex((tab) => tab.isActive === true);
          //Select first tab if none is marked selected
          if(this.currentTab === -1 && this.totalTabs > 0){  
              this.tabs[0].isActive = true;
              this.currentTab = 0;
          }
          
          //Setup Initial Progress
          this.progress = ((this.currentTab + 1) / this.totalTabs * 100);
  },
  updated(){
      this.isValidationSupport = (Object.keys(this.storeState.v).length !== 0 && this.storeState.v.constructor === Object) ? true : false;
  },
  methods:{
      previousTab(){
          this._switchTab(this.currentTab - 1);
          this.$emit('onPreviousStep'); 
      },
      nextTab(){
          if(this._validateCurrentTab() === false)
              return;
          this._switchTab(this.currentTab + 1);    
          this.$emit('onNextStep');          
            
      },
      reset(){
          this.tabs.forEach(tab => {
            tab.isActive = false;
            tab.isValidated = false;
          });
          this._switchTab(0);
          this.submitSuccess = false;
          this.storeState.v.$reset();
          this.$emit('onReset');
      },
      changeStatus(){
          this.submitSuccess = true;
      },
      selectTab(index){
          if(index < this.currentTab){
            this._switchTab(index);
          }
          if(this._validateCurrentTab() === false){
              return;
          }
          if(this.tabs[index - 1].isValidated === false){
              return;
          }
          this._switchTab(index);
          
      },
      onSubmit(){
          if(this._validateCurrentTab() === false)
              return;
          this.$emit('onComplete');
      },
      _switchTab(index){
          //Disable all tabs
          this.tabs.forEach(tab => {
            tab.isActive = false;
          });
          this.currentTab = index;
          this.tabs[index].isActive = true;
          this.progress = ((this.currentTab + 1) / this.totalTabs * 100);
      },
      _validateCurrentTab(){
          if(!this.isValidationSupport)  //Check if user wants to validate 
              return true;
          this.storeState.v.$touch();
          if (this.storeState.v.$invalid) {
              this.tabs[this.currentTab].isValidated = false;
              return false;
          }
          this.tabs[this.currentTab].isValidated = true;
          return true;
      }
  },
  watch:{
      currentTab(){
        store.setCurrentTab(this.currentTab);
      }
  }
}
</script>
