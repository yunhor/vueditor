
<style module>
  .wrap {
    width: 500px;
    position: relative;
    top: 50%;
    background: #fff;
    margin: 0 auto;
    transform: translateY(-60%);
  }
</style>

<template>
  <div class="ve-dialog" v-show="showPopup" @click.self="hideDialog">
    <div :class="$style.wrap">
      <div class="ve-dialog-header">{{lang.title}}<a href="javascript:;" class="ve-close" @click="hideDialog">&times;</a></div>
      <div class="ve-dialog-body">
        <form ref="form">
          <input type="file" name="image" ref="file">
        </form>
      </div>
      <div class="ve-dialog-footer">
        <div class="ve-btn-box">
          <button class="ve-btn" @click="hideDialog">{{lang.cancel}}</button>
          <button class="ve-btn" @click="certainHandler">{{lang.ok}}</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

  import { getLang } from '../config/lang.js'
  import { getConfig } from '../config/'

  export default {
    data () {
      return {
        lang: getLang('picture')
      }
    },
    computed: {
      showPopup: function () {
        return this.$store.state.toolbar.picture.showPopup;
      }
    },
    watch: {
      'showPopup': function (val) {
        if(val){
          document.body.classList.add('ve-fixed');
        }else{
          document.body.classList.remove('ve-fixed');
        }
      }
    },
    methods: {
      hideDialog () {
        this.$store.dispatch('updatePopupDisplay');
      },
      certainHandler (event) {
        let url = '';
        let obj = this.$refs.file;
        let form = this.$refs.form;
        let uploadUrl = getConfig('uploadUrl');
        if (navigator.userAgent.indexOf('MSIE') >= 1) { // IE
          url = obj.value;
        } else {
          if (obj.files.length != 0 && obj.files.item(0).type.indexOf('image') != -1) {
            url = window.URL.createObjectURL(obj.files.item(0));
          }
        }
        if (url) {
          if(this.$parent.upload){
            this.$parent.upload(obj, function (href) {
              this.$store.dispatch('execCommand', {name: 'insertHTML', value: `<img src="${href}">`});
              this.hideDialog();
            });
          }else if(uploadUrl){
            let formData = new FormData(form);
            let xhr = new XMLHttpRequest();
            xhr.open('POST', uploadUrl);
            xhr.send(formData);
            xhr.onload = function () {
              this.$store.dispatch('execCommand', {name: 'insertHTML', value: `<img src="${xhr.responseText}">`});
              this.hideDialog();
            }.bind(this);
            xhr.onerror = function (err) {
              alert(err);
            }
          }else{
            this.$store.dispatch('execCommand', {name: 'insertHTML', value: `<img src="${url}">`});
            this.hideDialog();
          }
        } else {
          alert(this.lang.invalidFile);
        }
      }
    }
  }
</script>