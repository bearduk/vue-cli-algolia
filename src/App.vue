<template>
  <div id="app">
    {{msg}}
  <hr>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/pikaday/1.6.1/css/pikaday.min.css">
<!--     <div>  
    From: <input type="text" v-model="searchFrom">
    <br>
    To: <input type="text" v-model="searchTo">
    </div> -->
<!--     <div>  
    From: <input type="text" v-model="momentFrom">
    <br>
    To: <input type="text" v-model="momentTo">
    </div> -->

    <div>
      <!-- <input type="text" v-model.trim="searchQuery" class="searchQ"> --> <!-- trim to reduce queries-->
      <p>on key up (eats up quota)</p>
      <input type="text" v-model.trim="searchQuery" class="searchQ" v-on:keyup="searchMethod">
      <p>on enter or submit (reduces quota hits)</p>
      <input type="text" v-model.trim="searchQuery" class="searchQ" v-on:keyup.enter="searchMethod">      
      <button @click="searchMethod">submit your search</button>
      <p><b>search term:</b> {{ searchQuery }}</p>
    </div>
    <div>
      From (date picker): <input type="text" id="datepickerFrom">
      
      To (date picker): <input type="text" id="datepickerTo">
    </div>

    
    <br>
    <p><b>unix from:</b> {{searchFrom}} <b>to:</b> {{searchTo}}</p>
<!--     <p>converts to:</p>
    <p><b>from:</b> {{momentFrom}} <b>to:</b> {{momentTo}}</p>
    <br> -->

<div>
  <input type="checkbox" id="artsculture" value="Arts and Culture" v-model="checkedCategories">
<label for="artsculture">Arts and Culture</label>
<input type="checkbox" id="academicconferences" value="Academic Conferences" v-model="checkedCategories">
<label for="academicconferences">Academic Conferences</label>
<input type="checkbox" id="business" value="Business" v-model="checkedCategories">
<label for="business">Business</label>
<input type="checkbox" id="graduation" value="Graduation Ceremonies" v-model="checkedCategories">
<label for="graduation">Graduation Ceremonies</label>
<input type="checkbox" id="opendays" value="Open Days" v-model="checkedCategories">
<label for="opendays">Open Days</label>
<input type="checkbox" id="publiclectures" value="Public Lectures" v-model="checkedCategories">
<label for="publiclectures">Public Lectures</label>
<input type="checkbox" id="sport" value="Sport" v-model="checkedCategories">
<label for="sport">Sport</label>
<br>
<span>Checked categories: {{ checkedCategories }}</span>
<br>
<span>checkedCategoriesString (computed string): {{ checkedCategoriesString }}</span>
</div>



<h1 v-show="eventArrayLength == 1">{{eventArrayLength}} result:</h1>
<h1 v-show="eventArrayLength > 1 | eventArrayLength == 0">{{eventArrayLength}} results:</h1>
    <ul>

      <li v-for="result in eventArray">
        <!-- <h3 v-html="result._highlightResult.eventTitle.value"></h3> {{result.abstract}}<hr /> -->
        <h3><a v-bind:href="result.urlPath" v-html="result._highlightResult.eventTitle.value"></a></h3> {{result.abstract}}<hr />
        <!-- <h3><a href="{{result.urlPath}}" v-html="result._highlightResult.eventTitle.value"></a></h3> <p>{{result.abstract}}</p><hr /> -->
      </li>
    </ul>
    <hr>
    <!-- <img src="./assets/Algolia_logo_bg-white.svg" alt="Algolia" /> -->
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
  name: 'app', // not needed
  data () {
    return {
      msg: 'Welcome to Algolia Events Vue.js, Chris B.',
      searchFrom: 0,
      searchTo: 0,
      // momentFrom: 0, // no longer needed
      // momentTo: 0, // no longer needed
      pickerFrField: 0,
      pickerToField: 0,
      eventArray: [],
      eventArrayLength: 0,
      searchQuery: '*',
      checkedCategories: [],
      checkedCategoriesString: '',
      page: 0
    }
  },
  mounted: function() {



      this.$nextTick(function(){
        this.searchAlg; // run the computed property as soon as we're booted

        var self = this;        

        // init the algolia search index
        var algoliasearch = require('algoliasearch');
        // -> corporate alg
        this.client = algoliasearch("HT7VYJG3KU", "d37bbf3291b226676c9f3f1937e865d3");
        // -> test alg
        // this.client = algoliasearch("QW5WOXYL7O", "0f825e2369bc691ec1bb85815e452ff5");
        this.index = this.client.initIndex('keele_main_EVENTS');


        var moment = require('moment');
        var Pikaday = require('pikaday');

        // initialise Pikaday from and to
        var pickerF = new Pikaday({ 
          field: document.getElementById('datepickerFrom'),
          format: 'D-M-YYYY',
          // format: 'Do MMM YY', // moment format for DOM. time and unix gets passed separately so ok to change this?
          onSelect: function(){
            console.log(this._d);
            var newFromUnix = new Date(this._d).getTime() / 1000;
            console.log(newFromUnix);
            self.searchFrom = newFromUnix;
            console.log('searchFrom is: ' + self.searchFrom);
          }
        });
        var pickerT = new Pikaday({ 
          field: document.getElementById('datepickerTo'),
            format: 'D-M-YYYY',
            // format: 'Do MMM YY', // moment format            
            onSelect: function(){
              console.log(this._d);
              var newFromUnix = new Date(this._d).getTime() / 1000;
              console.log(newFromUnix);
              self.searchTo = newFromUnix;
              console.log('searchFrom is: ' + self.searchTo);
          }
        });

        // populate dates on initial load from today, for one (momentjs) month
        // default start today
        var today = new Date();
        // today in unix
        var unixToday = Math.floor(today.getTime() / 1000);
        // alter picker from to today. this will trigger a search
        pickerF.setDate(today);
        // console.log('today is: ' + today + ' which is unix ' + unixToday);

        // month ahead
        var monthAhead = moment().add(1, 'months').format('MM/DD/YYYY');
        // set picker date, this will trigger a search
        pickerT.setDate(monthAhead);

        this.checkedCategoriesStringify();

        // just for testing, remove this when it's complete as we don't need a default search loaded
        this.searchMethod();

      }) // tixk

  },
  methods: {
    searchMethod: function () {
      console.log('search method run');

      // set object to self rather than binding it in 
      var self = this;
      var page = self.page;
      // with params // to do - add searchQuery here
      this.index.search(this.searchQuery, {

      filters: '(unixStartDate:' + this.searchFrom + ' TO ' + this.searchTo + ') AND (' + this.checkedCategoriesString + ' )',

      // TO DO:
      // Add in additional filters for categories https://www.algolia.com/doc/guides/searching/filtering/#filtering
      // Note _tags is reserved so if I add that to the JSON, I can then effectively do:
      // filters: " (unixstart TO unixend stuff) AND ( lecture OR music OR openday )  "
      // OR FOR FACET FILTERS SEE https://www.algolia.com/doc/guides/searching/filtering/#facet-filters
      // Try faceting using an aray that's created with vue bound check boxes

      attributesToRetrieve: ['eventTitle', 'abstract', 'urlPath', '_tags'],
      hitsPerPage: 50,
      page: page // pass in from var above which references data 'page'. Use this to work on creating pagination buttons
      }, function searchDone(err, content) {
            if (err) {
              console.error(err);
              return;
            }

      // view full return
      console.log(content);            

      // clear array
      self.eventArray = [];  
      // and now push the new array          
      for (var h in content.hits) {
          // push new results to array
          self.eventArray.push(content.hits[h]);
          console.log(content.hits[h]);
          // console.log(content.hits[h]);
          // console.log('Hit(' + content.hits[h].objectID + '): ' + content.hits[h].eventTitle.toString() + content.hits[h].abstract );

      }

      // output the array length to make it easy to track
      var eventArrayLength = self.eventArray.length;
      self.eventArrayLength = eventArrayLength; 

      // check output
      // console.log(self.eventArray);
      }); // index search complete


    },
    checkedCategoriesStringify: function(){
      
      let localcheckedCategories = this.checkedCategories;

      // first cater for nothing being ticked. Set the filter string to something that returns all results such as have a start data that isn't 0
      if(  localcheckedCategories.length <= 0 ) {
        // console.log("LESS THAN OR EQUAL TO ZERO: " + localcheckedCategories.length );
        this.checkedCategoriesString = "unixStartDate != 0";
      } else {
      // first wrap array in quotes (required for filter syntax)
      let quoteCategories = localcheckedCategories.map(function(item) {
        return '"' + item + '"';
      })
      // now join the categories using OR
      let filterBuilt = quoteCategories.join(" OR ");
      this.checkedCategoriesString = filterBuilt;
      // console.log("MORE THAN ZERO: " + localcheckedCategories.length );
      return filterBuilt;
      }
    }
  },
  computed: {

//       searchAlg: function() {

//       // set object to self rather than binding it in        
// //      var self = this;
//       // var searchingFromComputed = this.searchFrom;
//       // var searchingToComputed = this.searchTo;
//       // var searchQueryComputed = this.searchQuery;

//       // this init needs moving outside of computed
//  //     var algoliasearch = require('algoliasearch');
//  //     var moment = require('moment');

//       // -> corporate alg
//       // var client = algoliasearch("HT7VYJG3KU", "d37bbf3291b226676c9f3f1937e865d3");
//       // -> test alg
// //      this.client = algoliasearch("QW5WOXYL7O", "0f825e2369bc691ec1bb85815e452ff5");
// //      this.index = this.client.initIndex('dev_EVENTS');

//       // var mdateFrom = moment.unix(this.searchFrom).format("LLLL");
//       // console.log("mdateFrom", mdateFrom);
//       // this.momentFrom = mdateFrom;
//       // var mdateTo = moment.unix(this.searchTo).format("LLLL");
//       // console.log("mdateTo", mdateTo);
//       // this.momentTo = mdateTo;

    

//     }
  },
  watch: {
    searchFrom: function () {
      this.searchMethod();
    },
    searchTo: function() {
      this.searchMethod();
    },
    checkedCategories: function(){
      this.checkedCategoriesStringify();
      this.searchMethod();
    }
    // searchQuery now handled by v-on in the view
    // searchQuery: function() {
    //   this.searchAlg;
    // }
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
// Above CSS from vue-cli
/* CB added from now on: */
em {
  background-color: yellow;
}
input {
  padding:10px;
  text-align: center;
}
input.searchQ {
    width: 200px;    
}
</style>
