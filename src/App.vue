<template>
  <div id="app">
    {{msg}}
  <hr>

    <div>  
    From: <input type="text" v-model="searchFrom">
    <br>
    To: <input type="text" v-model="searchTo">
    </div>
    <br>
    from: {{searchFrom}} to: {{searchTo}}

    <br>

    <ul>
      <li v-for="result in eventArray">
        {{result.eventTitle}}: {{result.abstract}}<hr />
      </li>
    </ul>
    <hr>
    <img src="./assets/Algolia_logo_bg-white.svg" alt="Algolia" />
  </div>
</template>

<script>
// USING .then()
// index.search({
//   query: '',
//   filters: '(unixStartDate:0 TO 1288965600)',
//   attributesToRetrieve: ['eventTitle', 'abstract']
// }).then(res => {
//   console.log(res);
//   for (var h in res.hits) {
//     console.log('Hit(' + res.hits[h].objectID + '): ' + res.hits[h].eventTitle.toString() + res.hits[h].abstract );
//   }
// });

export default {
  name: 'app',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App Chrissy B',
      searchFrom: 0,
      searchTo: 1288965600,
      eventArray: []
    }
  },
  mounted: function() {

      this.$nextTick(function(){
        this.searchAlg; // run the computed property as soon as we're booted
      })

  },
  computed: {
      searchAlg: function() {

      // set object to self rather than binding it in        
      var self = this;

      var algoliasearch = require('algoliasearch');
      var client = algoliasearch("HT7VYJG3KU", "d37bbf3291b226676c9f3f1937e865d3");
      var index = client.initIndex('dev_EVENTS');
      // with params
      index.search('', {

        filters: '(unixStartDate:' + this.searchFrom + ' TO ' + this.searchTo + ')',
        attributesToRetrieve: ['eventTitle', 'abstract'],
        hitsPerPage: 50
        }, function searchDone(err, content) {
            if (err) {
              console.error(err);
              return;
            }
          
        // clear array
        self.eventArray = [];  
        // and now push the new array          
        for (var h in content.hits) {
          // push new results to array
          self.eventArray.push(content.hits[h]);
          // console.log(content.hits[h]);
          console.log('Hit(' + content.hits[h].objectID + '): ' + content.hits[h].eventTitle.toString() + content.hits[h].abstract );

        }
        // check output
        console.log(self.eventArray);
      });
    }
  },
  watch: {
    searchFrom: function (val) {
      this.searchAlg;
    },
    searchTo: function(val) {
      this.searchAlg;
    },
    eventArray: function(val) {
      this.searchAlg;
    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
