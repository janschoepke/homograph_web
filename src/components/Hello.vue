<template>
  <div class="main">
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark fixed-top" id="mainNav">
      <div class="container">
        <span class="navbar-brand js-scroll-trigger"><span class="highlight">h</span>omograph</span>
        <div class="collapse navbar-collapse" id="navbarResponsive">
          <ul class="navbar-nav ml-auto">
            <li class="nav-item">
              <a class="nav-link" href="https://github.com/janschoepke/homograph" target="_blank">GitHub</a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="https://en.wikipedia.org/wiki/IDN_homograph_attack" target="_blank">about homograph attacks</a>
            </li>
          </ul>
        </div>
      </div>
    </nav>

    <header class="bg-primary text-white">
      <div class="container text-center">
        <input type="text" name="companyUrl" v-model="companyUrl" class="companyUrl" autofocus>
        <p class="lead" v-if="!companyUrl">enter company name to analyze.</p>
        <p class="lead" v-if="companyUrl && potentialLeaksFound === 1">{{potentialLeaksFound}} potential risk found for <strong>{{companyUrl}}</strong>.</p>
        <p class="lead" v-if="companyUrl && potentialLeaksFound !== 1">{{potentialLeaksFound}} potential risks found for <strong>{{companyUrl}}</strong>.</p>
      </div>
    </header>

    <section v-if="companyUrl">
      <div class="container">
        <div class="row">
          <div class="col-lg-8 mx-auto">
            <h2>string analysis</h2>
            <div class="chars">
              <div class="char">
                <div>Char: </div>
                <div>Unicode char: </div>
              </div>
              <div v-for="char in companyUrl" class="char">
                <div>{{char}}</div>
                <div><code>{{char.charCodeAt(0)}}</code></div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <section id="services" class="bg-light" v-if="companyUrl  && potentialLeaksFound > 0">
      <div class="container">
        <div class="row">
          <div class="col-lg-8 mx-auto">
            <h2>possible risks ({{potentialLeaksFound}})</h2>
          </div>
        </div>
        <div class="row">
          <div class="col-md-5">

            <p>
              The identified possible risks are listed beside. To check the availability of the respective domains, select the relevant domain extensions and start the query by clicking on the button.
            </p>

            <b-form-checkbox-group v-model="selectedDomains"
                                   buttons
                                   button-variant="primary"
                                   size="lg"
                                   name="domains"
                                   :options="enabledDomains">
            </b-form-checkbox-group>

            <p class="margin">
              <button class="btn btn-primary" @click="whois()" :disabled="!selectedDomains.length">check domain availability</button>
            </p>

            <b-alert variant="danger"
                     :show="apiError">
              <strong>Oops!</strong> We currently have problems contacting our API. Please try again later.
            </b-alert>

            <b-alert variant="danger"
                     :show="limitError">
              <strong>Oops!</strong> Our API only accepts requests for < 50 domain requests, you requested {{potentialLeaks.length * selectedDomains.length}}. Please shorten your input or the number of domain extensions.
            </b-alert>


            <div v-if="selectedDomains.length" class="text-left background-white">
              Legend:
              <div class="separator"><span class="domain-unknown">.{{selectedDomains[0]}}</span> <span>This domain has not been checked yet.</span></div>
              <div class="separator"><span class="domain-unknown text-success">.{{selectedDomains[0]}}</span> <span>This domain is free to purchase.</span></div>
              <div><span class="domain-unknown text-danger">.{{selectedDomains[0]}}</span> <span>This domain has already been purchased.</span></div>
            </div>

          </div>
          <div class="col-md-7">
            <b-list-group>
              <b-list-group-item v-for="potentialLeak in potentialLeaks" :key="potentialLeak.id">
                <span v-for="char in potentialLeak.domain">
                  <span v-if="char.homographic"  v-b-popover.hover.auto="'Char: ' + char.letter + ' (' + char.letter.charCodeAt(0)+')\n Homographic: ' + char.homographicLetter + ' (' + char.homographicLetter.charCodeAt(0) + ')'" title="homographic character" class="homographic text-danger">{{char.homographicLetter}}</span>
                  <span v-if="!char.homographic">{{char.letter}}</span>
                </span>
                <span class="domain-com domain-unknown" v-if="selectedDomains.indexOf('com') > -1" v-bind:class="{'text-success': potentialLeak.com === 2, 'text-danger': potentialLeak.com === 1 }">.com</span>
                <span class="domain-net domain-unknown" v-if="selectedDomains.indexOf('net') > -1" v-bind:class="{'text-success': potentialLeak.net === 2, 'text-danger': potentialLeak.net === 1 }">.net</span>
                <span class="domain-eu domain-unknown" v-if="selectedDomains.indexOf('eu') > -1" v-bind:class="{'text-success': potentialLeak.eu === 2, 'text-danger': potentialLeak.eu === 1 }">.eu</span>
                <span class="domain-info domain-unknown" v-if="selectedDomains.indexOf('info') > -1" v-bind:class="{'text-success': potentialLeak.info === 2, 'text-danger': potentialLeak.info === 1 }">.info</span>
                <span class="domain-com domain-unknown" v-if="selectedDomains.indexOf('de') > -1" v-bind:class="{'text-success': potentialLeak.de === 2, 'text-danger': potentialLeak.de === 1 }">.de</span>
              </b-list-group-item>
            </b-list-group>
          </div>
        </div>
      </div>
    </section>

    <section id="contact">
      <div class="container">
        <div class="row">
          <div class="col-lg-8 mx-auto">
            <h2>about homograph</h2>
            <p class="lead">This application allows the detection and prevention of homographic attacks in company names and their domains. Except for the optional domain query, no data is stored or passed on to third parties.</p>
          </div>
        </div>
      </div>
    </section>

    <footer class="py-5 bg-dark">
      <div class="container">
        <p class="m-0 text-center text-white">homograph by <a href="https://github.com/janschoepke" target="_blank">janschoepke</a> (2017)</p>
      </div>
      <!-- /.container -->
    </footer>

  </div>
</template>

<script>

export default {
  name: 'hello',
  data: function () {
    return {
      companyUrl: '',
      homographicChars: {
        'c': 'ċ',
        'a': 'ą',
        'd': 'đ',
        'e': 'ė',
        'g': 'ġ',
        'h': 'ħ',
        'i': 'ı',
        'k': 'ĸ',
        'l': 'ĺ',
        'n': 'ŋ',
        's': 'ş',
        't': 'ŧ',
        'u': 'ų',
        'z': 'ż'
      },
      potentialLeaksFound: 0,
      potentialLeaks: [],
      enabledDomains: [
        {text: '.com', value: 'com'},
        {text: '.net', value: 'net'},
        {text: '.eu', value: 'eu'},
        {text: '.info', value: 'info'},
        {text: '.de', value: 'de'}
      ],
      selectedDomains: [],
      apiError: false,
      limitError: false
    }
  },
  methods: {
    k_combinations (set, k) {
      var i, j, combs, head, tailcombs
      if (k > set.length || k <= 0) {
        return []
      }
      if (k === set.length) {
        return [set]
      }
      if (k === 1) {
        combs = []
        for (i = 0; i < set.length; i++) {
          combs.push([set[i]])
        }
        return combs
      }
      combs = []
      for (i = 0; i < set.length - k + 1; i++) {
        head = set.slice(i, i + 1)
        tailcombs = this.k_combinations(set.slice(i + 1), k - 1)
        for (j = 0; j < tailcombs.length; j++) {
          combs.push(head.concat(tailcombs[j]))
        }
      }
      return combs
    },
    combinations (set) {
      var k
      var i
      var combs
      var kCombs
      combs = []
      for (k = 1; k <= set.length; k++) {
        kCombs = this.k_combinations(set, k)
        for (i = 0; i < kCombs.length; i++) {
          combs.push(kCombs[i])
        }
      }
      return combs
    },
    whois () {
      this.$http.post(
        'http://janschoepke.de/api/homograph/index.php',
        {
          potentialLeaks: this.potentialLeaks,
          selectedDomains: this.selectedDomains
        }).then(
        function success (response) {
          this.apiError = false
          if (response.body !== 'ERR1') {
            this.potentialLeaks = response.body
            this.limitError = false
          } else {
            this.limitError = true
          }
        },
        function fail (response) {
          this.apiError = true
        }
      )
    }
  },
  watch: {
    'companyUrl': function () {
      this.potentialLeaksFound = 0
      this.potentialLeaks = []
      var leakChars = []
      for (var i = 0; i < this.companyUrl.length; i++) {
        if (this.homographicChars.hasOwnProperty(this.companyUrl[i])) {
          leakChars.push({'letter': this.companyUrl[i], 'charAt': i, 'homographicLetter': this.homographicChars[this.companyUrl[i]]})
        }
      }
      var potentialLeakArray = this.combinations(leakChars)
      var companyUrlChars = this.companyUrl.split('')
      for (var l = 0; l < companyUrlChars.length; l++) {
        companyUrlChars[l] = {letter: companyUrlChars[l], homographic: false}
      }
      this.potentialLeaksFound = potentialLeakArray.length
      for (var j = 0; j < potentialLeakArray.length; j++) {
        var tempLeakString = JSON.parse(JSON.stringify(companyUrlChars))
        for (var k = 0; k < potentialLeakArray[j].length; k++) {
          tempLeakString[potentialLeakArray[j][k].charAt] = {letter: potentialLeakArray[j][k].letter, homographic: true, homographicLetter: potentialLeakArray[j][k].homographicLetter}
        }
        var domainString = ''
        for (var m = 0; m < tempLeakString.length; m++) {
          if (typeof tempLeakString[m].homographicLetter !== 'undefined') {
            domainString += tempLeakString[m].homographicLetter
          } else {
            domainString += tempLeakString[m].letter
          }
        }
        this.potentialLeaks.push({domain: tempLeakString, com: 0, net: 0, eu: 0, info: 0, de: 0, domainString: domainString})
      }
    }
  }
}
</script>

<style scoped>
  .navbar-brand {
    cursor: default;
    background: none;
    background-image: none;
    color: #ccc;
  }
  .navbar-brand .highlight {
    color: #59badd;
  }
  @media screen and (max-width: 991px) {
    .navbar-brand {
      text-align: center;
      width: 100%;
    }
  }

h1, h2 {
  font-weight: normal;
  margin-bottom: 15px;
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
  color: #59badd;
}

  .homographic {
    margin-left: 4px;
  }

  .margin {
    margin: 20px 0;
  }

  .domain-unknown {
    padding: 0.2rem 0.4rem;
    font-size: 90%;
    background-color: #f8f9fa;
    border-radius: 0.25rem;
  }

  .background-white {
    background-color: #ffffff;
    padding: 20px;
  }

  .separator {
    margin-bottom: 5px;
  }

  .list-group {
    max-height: 400px;
    overflow: scroll;
  }

header {
  padding: 154px 0 100px;
}

@media (min-width: 992px) {
  header {
    padding: 156px 0 100px;
  }
}

section {
  padding: 160px 0;
}

  .companyUrl {
    background-color: transparent;
    border: 0px;
    border-bottom: 2px solid #fff;
    font-size:24px;
    outline: none;
    margin-bottom: 20px;
    color: #fff;
  }

  .char {
    display: inline-block;
    margin-left: 5px;
  }
</style>
