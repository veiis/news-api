<template>
  <div id="app">
    <div class="container">
      <!-- Start of Container -->
      <div class="search-area">
        <div class="search-items">
          <p>What do you looking for?</p>
          <input type="text" v-model="search_term" v-on:input="search(search_term)"/>
          <p>{{ data_counter }} results</p>
          <span>Filtering</span>
          <select name="scf_select"  v-model="scf_select_value" @change="set_url_by_type(scf_select_value)">
            <option value="story">Filter by Story</option>
            <option value="comment">Filter by Comment</option>
            <option value="front_page">Filter by Front Page</option>
          </select>
          <span>Filtering</span>
          <select name="lr_select" v-model="lr_select_value" @change="set_url_by_lat(lr_select_value)">
            <option value="last">Latest</option>
            <option value="revelant">Revelant</option>
          </select>
          <span>Filtering</span>
          <select name="hits_select" v-model="hits_select_value" @change="set_news_hits(hits_select_value)">
            <option value="10">10</option>
            <option value="20">20</option>
            <option value="30">30</option>
          </select>
        </div>
      </div>
      <div class="news-area">
        <div class="news-item" v-for="item in news" :key="item.objectID">
          <h3>
            <a :href="item.url">{{ item.title || '[ TITLE IS EMPTY ]'}}</a>
          </h3>
          <p>
            {{ item.points || '[ NO ]'}} points
            <span>|</span>
            <a :href="'https://news.ycombinator.com/user?id='+item.author">{{ item.author }}</a>
            <span>|</span>
            At {{ item.created_at }}
            <span>|</span>
            {{ item.num_comments || '0'}} comments
          </p>
        </div>
      </div>
      <div class="pagination">
        <span v-for="i in pages" :key="i" v-bind="page" @click="get_next_page(i)">{{ i }}</span>
      </div>
      <!-- End of Container -->
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "app",
  data() {
    return {
      news: [],
      url: "http://hn.algolia.com/api/v1/search?",
      scf_select_value: '',
      lr_select_value: '',
      hits_select_value: '',
      search_term: '',
      data_counter: '',
      pages: 0,
      page: 0
    };
  },
  methods: {

    get_currect_time(timestamp) {
      let date = new Date(timestamp * 1000);
      let year = date.getFullYear();
      // let date_now = new Date();
      // let ymd = date_now.getFullYear();
      return year;
    },
    set_url_by_type(type) {
      if (type === 'story') {
        this.url = `http://hn.algolia.com/api/v1/search?tags=story`;
      }
      if (type === 'comment') {
        this.url = `http://hn.algolia.com/api/v1/search?tags=comment`;
      }
      if (type === 'front_page') {
        this.url = `http://hn.algolia.com/api/v1/search?tags=front_page`;
      }
      this.get_data();
    },
    set_url_by_lat(type) {
      if (type === 'last') {
        this.url = this.url.replace('search', 'search_by_date') + 'tags=story';
      }
      if (type === 'revelant') {
        this.url = `http://hn.algolia.com/api/v1/search?`;
      }
      this.get_data();
    },
    set_news_hits(hits) {
      this.url = this.url + `&hitsPerPage=${hits}`
      this.get_data();
    },
    search(search_term) {
      this.url = this.url + `&query=${search_term}`
      this.get_data();
    },
    get_next_page(page) {
      this.url += `&page=${page}`;
      this.get_data();  
    },
    get_data: function() {
      this.news = [],
      axios.get(this.url).then(response => {
        response.data.hits.forEach(item => {
          item.created_at = this.get_currect_time(item.created_at_i);
          this.news.push(item);
        });
        this.data_counter = (response.data.nbHits).toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
        this.pages = response.data.nbPages;
      });
    }
  },
  mounted() {
    this.get_currect_time();
    this.get_data();
  }
};
</script>

<style>
@import "./assets/style.css";
</style>
