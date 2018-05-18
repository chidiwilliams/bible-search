<template>
  <section class="body">
    <form action="#" class="search-form" v-on:submit.prevent>
      <input type="text" id="search-box" v-model="query" placeholder="Please enter text" :disabled="!completed">
    </form>

    <div class="results">
      <div v-if="loading" class="loading">Loading...</div>
      
      <div v-for="result in results" class="result" v-bind:key="result.id">
        <span class="result-verse-number">{{ formatVerse(result.id) }}</span>
        <span class="result-verse">{{ result.verse }}</span>
      </div>
    </div>
  </section>
</template>

<script>
import BibleBooks from './js/BibleBooks.js'
import Bible from './js/Bible.js'

export default {
  name: 'search',
  data () {
    return {
      elastic: null,
      completed: false,
      query: '',
      noLoad: false,
      results: [],
      loading: false
    }
  },
  created() {    
    this.loading = true

    this.debouncedQueryData = _.debounce(this.queryData, 400);

    this.elastic = elasticlunr(function () {
      this.addField('verse');
    });

    elasticlunr.clearStopWords();

    this.saveBible();
  },
  mounted() {
    document.getElementById('search-box').focus();
  },
  methods: {
    saveBible(callback) {
      Bible.data.forEach(element => {
        const id = element.field[0]
        const verse = element.field[4]

        this.elastic.addDoc({ id, verse })
      });

      this.loading = false;
      this.completed = true;
    },
    formatVerse(verseId) {
      var verseIdString = verseId.toString();

      var bookNo = +verseIdString.substring(0, verseIdString.length - 6);
      var chapterNo = +verseIdString.substring(verseIdString.length - 6, verseIdString.length - 3);
      var verseNo = +verseIdString.substring(verseIdString.length - 3);

      var book = BibleBooks.data[bookNo - 1];

      return book + ' ' + chapterNo + ':' + verseNo;
    },
    
    queryData() {
      var results = this.elastic.search(this.query, {
        expand: true
      });
      results = results.slice(0, 10);

      this.results = [];
      results.forEach(result => {
        this.results.push(this.elastic.documentStore.getDoc(result.ref))
      });
    }
  },
  watch: {
    query() {
      this.debouncedQueryData();
    }
  }
}
</script>
