<template>
  <div class="container is-widescreen">
    <section class="section" v-if="error">
      <div class="container is-widescreen">
        <div class="notification is-danger">
          <!-- <%= error.code + ': ' + error.sqlMessage %> -->
          <!---->
          {{ error }}
        </div>
      </div>
    </section>
    <section class="hero">
      <div class="hero-body">
        <p class="title">Create new Blog</p>
      </div>
    </section>
    <section class="px-6">
      <input
        class="mb-5"
        multiple
        type="file"
        accept="image/png, image/jpeg, image/webp"
        @change="selectImages"
      />

      <div v-if="images" class="columns is-multiline">
        <div v-for="(image, index) in images" :key="image.id" class="column is-one-quarter">
          <div class="card">
            <div class="card-image">
              <figure class="image is-4by3">
                <img :src="showSelectImage(image)" alt="Placeholder image" />
              </figure>
            </div>
            <footer class="card-footer">
              <a @click="deleteSelectImage(index)" class="card-footer-item has-text-danger">Delete</a>
            </footer>
          </div>
        </div>
      </div>

      <div class="field mt-5">
        <label class="label">Title</label>
        <div class="control">
          <input v-model="$v.titleBlog.$model" :class="{'is-danger': $v.titleBlog.$error}" class="input" type="text" placeholder="Text input" />
        </div>
        <template v-if="$v.titleBlog.$error">
            <p class="help is-danger" v-if="!$v.titleBlog.required">This field is required</p>
            <p class="help is-danger" v-if="!$v.titleBlog.minLength">min 10</p>
            <p class="help is-danger" v-if="!$v.titleBlog.maxLength">max 25</p>
            <p class="help is-danger" v-if="!$v.titleBlog.allString">All string</p>
        </template>
      </div>

      <div class="field">
        <label class="label">Content</label>
        <div class="control">
          <textarea v-model="$v.contentBlog.$model" :class="{'is-danger': $v.contentBlog.$error}" class="textarea" placeholder="Textarea"></textarea>
        </div>
        <template v-if="$v.contentBlog.$error">
            <p class="help is-danger" v-if="!$v.contentBlog.required">This field is required</p>
            <p class="help is-danger" v-if="!$v.contentBlog.minLength">min 50</p>
        </template>
      </div>
      
      <div class="field">
        <label class="label">Reference</label>
        <div class="control">
          <input class="input" type="url" v-model="$v.reference.$model" :class="{'is-danger': $v.reference.$error}" placeholder="e.g. https://www.google.com">
        </div>
        <template v-if="$v.reference.$error">
            <p class="help is-danger" v-if="!$v.reference.url">only url</p>
        </template>
      </div>

      <div class="control mb-3">
        <label class="radio">
          <input v-model="$v.statusBlog.$model" type="radio" name="answer" value="status_private" />
          Private
        </label>
        <label class="radio">
          <input v-model="$v.statusBlog.$model" type="radio" name="answer" value="status_public" />
          Public
        </label>
      </div>

      <div class="field">
        <div class="control">
          <label class="checkbox">
            <input v-model="pinnedBlog" type="checkbox" />
            Pinned
          </label>
        </div>
      </div>

      <hr>

      <div class="columns">
        <div class="column">
          <div class="field">
            <label class="label">วันที่โพสต์</label>
            <div class="control">
              <input v-model="start_date" :class="{'is-danger': checkDate || dateMore}" class="input" type="date">
            </div>
            <template>
              <p class="help is-danger" v-if="checkDate">enter all</p>
              <p class="help is-danger" v-if="dateMore">start more than end</p>
            </template>
          </div>
        </div>
        <div class="column">
          <div class="field">
            <label class="label">วันสิ้นสุดโพสต์</label>
            <div class="control">
              <input v-model="end_date" :class="{'is-danger': checkDate || dateMore}" class="input" type="date">
            </div>
            <template>
              <p class="help is-danger" v-if="checkDate">enter all</p>
              <p class="help is-danger" v-if="dateMore">start more than end</p>
            </template>
          </div>
        </div>
      </div>

      <div class="field is-grouped">
        <div class="control">
          <button @click="submitBlog" class="button is-link">Submit</button>
        </div>
        <div class="control">
          <button @click="$router.go(-1)" class="button is-link is-light">Cancel</button>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
// function allString (value) {
//   var numbers = /^[0-9]+$/;
//   for(let i =0; i < value.length; i++){
//     if(value[i].match(numbers)){
//       return false
//     }
//   }
//   return true
// }
function status(value){
  if(value === 'status_public' || value === 'status_private'){
    return true
  }
  return false
}

import axios from "axios";
import moment from 'moment';
import { required, email, minLength, maxLength, sameAs, integer, alpha, url, and } from 'vuelidate/lib/validators'
export default {
  data() {
    return {
      blog: {},
      error: null,
      images: [], // array of image
      titleBlog: "",
      contentBlog: "",
      pinnedBlog: false,
      statusBlog: "status_public",
      reference: "",
      start_date: "",
      end_date: "",
      checkDate: false,
      dateMore: false
    };
  },
  watch: {
    start_date(newDate, old) {
      if ( (this.start_date === '' && this.end_date !== '') || (this.start_date !== '' && this.end_date === '') ) {
        this.checkDate = true
      }else{
        this.checkDate = false
      }
      var date1 = moment(this.start_date).format("YYYY-MM-DD")
      var date2 = moment(this.end_date).format("YYYY-MM-DD")
      if ( date1 > date2 && this.end_date !== '') {
        this.dateMore = true
      }else{
        this.dateMore = false
      }
    },
    end_date(newDate, old) {
      if ( (this.start_date === '' && this.end_date !== '') || (this.start_date !== '' && this.end_date === '') ) {
        this.checkDate = true
      }else{
        this.checkDate = false
      }
      var date1 = moment(this.start_date).format("YYYY-MM-DD")
      var date2 = moment(this.end_date).format("YYYY-MM-DD")
      if ( date1 > date2 && this.start_date !== '') {
        this.dateMore = true
      }else{
        this.dateMore = false
      }
    }
  },
  validations: {
        titleBlog: {
            required: required,
            allString: alpha,
            minLength: minLength(10),
            maxLength: maxLength(25)
        },
        contentBlog: {
          required: required,
          minLength: minLength(50),
        },
        statusBlog: {
          status: status
        },
        reference: {
          url: url
        },
    },
  methods: {
    selectImages(event) {
      if(event.target.files[0].size <= 1024 * 1024){
        this.images = event.target.files;
      }
    },
    showSelectImage(image) {
      // for preview only
      return URL.createObjectURL(image);
    },
    deleteSelectImage(index) {
      console.log(this.images);
      this.images = Array.from(this.images);
      this.images.splice(index, 1);
    },
    submitBlog() {
      this.$v.$touch();
      if (!this.$v.$invalid, !this.dateMore, !this.checkDate){
        let formData = new FormData();
        formData.append("title", this.titleBlog);
        formData.append("content", this.contentBlog);
        formData.append("pinned", this.pinnedBlog ? 1 : 0);
        formData.append("reference", this.reference);
        formData.append("start_date", this.start_date);
        formData.append("end_date", this.end_date);
        formData.append("status", this.statusBlog);
        this.images.forEach((image) => {
          formData.append("myImage", image);
        });
  
        // Note ***************
        // ตอนเรายิง Postmant จะใช้ fromData
        // ตอนยิงหลาย ๆ รูปพร้อมกันใน Postman จะเป็นแบบนี้
  
        // title   | "This is a title of blog"
        // comment | "comment in blog"
        // ...
        // myImage | [select file 1]
        // myImage | [select file 2]
        // myImage | [select file 3]
  
        // จะสังเกตุว่าใช้ myImage เป็น key เดียวกัน เลยต้องเอามา loop forEach
        // พอไปฝั่ง backend มันจะจัด file ให้เป็น Array เพื่อเอาไปใช้งานต่อได้
  
        axios
          .post("http://localhost:3000/blogs", formData)
          .then((res) => this.$router.push({name: 'home'}))
          .catch((e) => console.log(e.response.data));
      }
    },
  },
};
</script>

<style>
</style>