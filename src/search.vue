<template>
  <section class="body">
    <div class="import-db">
      <form action="#" class="upload-form" v-on:submit.prevent="submitForm">
        <label for="file">Import database</label>
        <input type="file" name="file" id="fileupload" accept=".csv" @change="submitForm">
      </form>
    </div>

    <form action="#" class="search-form" v-on:submit.prevent>
      <input type="text" id="search-box" v-model="query" placeholder="Please enter text" :disabled="!completed">
    </form>

    <div class="results">
      <div v-if="loading">Loading...</div>
      
      <div v-for="result in results" class="result" v-bind:key="result.doc.verse">
        <span class="result-verse-number">{{ getCleanVerse(result.doc.id) }}</span>
        <span class="result-verse">{{ result.doc.verse }}</span>
      </div>
    </div>


  </section>
</template>

<script>

export default {
  name: 'search',
  data () {
    return {
      elastic: null,
      completed: false,
      query: '',
      noLoad: false,
      results: [],
      books: [
        'Genesis', 'Exodus', 'Leviticus', 'Numbers', 'Deuteronomy', 'Joshua', 'Judges', 'Ruth', '1 Samuel', '2 Samuel', '1 Kings', '2 Kings', '1 Chronicles', '2 Chronicles', 'Ezra', 'Nehemiah', 'Esther', 'Job', 'Psalms', 'Proverbs', 'Ecclesiastes', 'Song of Solomon', 'Isaiah', 'Jeremiah', 'Lamentations', 'Ezekiel', 'Daniel', 'Hosea', 'Joel', 'Amos', 'Obadiah', 'Jonah', 'Micah', 'Nahum', 'Habakkuk', 'Zephaniah', 'Haggai', 'Zechariah', 'Malachi', 'Matthew', 'Mark', 'Luke', 'John', 'Acts', 'Romans', '1 Corinthians', '2 Corinthians', 'Galatians', 'Ephesians', 'Philippians', 'Colossians', '1 Thessalonians', '2 Thessalonians', '1 Timothy', '2 Timothy', 'Titus', 'Philemon', 'Hebrews', 'James', '1 Peter', '2 Peter', '1 John', '2 John', '3 John', 'Jude', 'Revelation',
      ],
      loading: false,
    }
  },
  created() {
    this.debouncedQueryData = _.debounce(this.queryData, 400);

    // this.elastic = elasticlunr(function () {
    //   this.addField('verse');
    // });

    // elasticlunr.clearStopWords();

    var idx = elasticlunr(function () {
        this.addField('title')
        this.addField('body')
    });

    var doc1 = {
        "id": 1,
        "title": "Oracle released its latest database Oracle 12g",
        "body": "Yestaday Oracle has released its new database Oracle 12g, this would make more money for this company and lead to a nice profit report of annual year."
    }

    var doc2 = {
        "id": 2,
        "title": "Oracle released its profit report of 2015",
        "body": "As expected, Oracle released its profit report of 2015, during the good sales of database and hardware, Oracle's profit of 2015 reached 12.5 Billion."
    }

    idx.addDoc(doc1);
    idx.addDoc(doc2);

    console.log(idx.search("Oracle database profit"));
  },
  methods: {
    submitForm() {
      // Check for correctness of file upload

      this.loading = true;

      // Get file
      if (window.File && window.FileReader && window.FileList && window.Blob) {
        let reader = new FileReader();
        reader.readAsText(document.getElementById('fileupload').files[0]);

        reader.onload = this.processData;
        reader.onerror = this.errorHandler;
      }
    },
    processData(event) {
      var csv = event.target.result;

      var allTextLines = csv.split(/\r\n|\n/);

      for (let i = 1; i < allTextLines.length - 1; i += 1) {
        var data = allTextLines[i];

        // get verse id
        // get position of comma after id
        var matchId = /,/.exec(data);
        var id = data.substring(0, matchId.index);

        // get position of comma before verse
        var matchVerse = /[0-9],[A-Z\"\(\[]/.exec(data);
        var verse = data.substring(matchVerse.index + 2);

        var obj = {
          "id": +id,
          "verse": verse
        }

        this.elastic.addDoc(obj)
      }

      this.loading = false;
      this.completed = true;

      document.getElementById('search-box').focus();
    },
    // errorHandler(event) {},
    getCleanVerse(verseId) {
      var bookNo = +verseId.toString().substring(0, verseId.length - 6);
      var chapterNo = +verseId.toString().substring(verseId.length - 6, verseId.length - 3);
      var verseNo = +verseId.toString().substring(verseId.length - 3);

      var book = this.books[bookNo - 1];

      return book + ' ' + chapterNo + ':' + verseNo;
    },
    queryData() {
      console.log(this.elastic.search(this.query));
      // this.results = this.elastic.search(this.query).slice(0, 5);
    }
  },
  watch: {
    query() {
      this.queryData();
      // this.debouncedQueryData();
    }
  }
}
</script>
