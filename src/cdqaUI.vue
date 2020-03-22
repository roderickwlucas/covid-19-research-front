<template>
  <div class="cdqaUI container-fluid">
    <link
      rel="stylesheet"
      href="https://use.fontawesome.com/releases/v5.8.2/css/all.css"
      integrity="sha384-oS3vJWv+0UjzBfQzYUhtDYW+Pj2yciDJxpsK1OYPAYjqT085Qq/1cq5FLXAZQ7Ay"
      crossorigin="anonymous"
    >

    <div class="row covid-research">
      <h1>COVID-19 Q&amp;A</h1><br>
      <h2>Use the knowledge of 44.000 COVID-virus research papers</h2>
      <p>This application has been built in a couple of hours in a response to this <a href="https://www.kaggle.com/allen-institute-for-ai/CORD-19-research-challenge">Kaggle Challenge</a>. The models are out of the box and are not optimized for the medical field. Many, many improvements can be made, but the significant potential of AI to tackle these challenges are demonstrated. The tool is a combination of work of others and from professional projects. For questions, do not hesitate to <a href="mailto:roderickwlucas@gmail.com">email me</a>.</p>
    </div>

    <div class="row examples">
      <h3>Example questions</h3>
      <br>

      <b-button class="btn btn-light example" @click="imputeExample('incubation', 'How long is the incubation period?', 10);">
        Incubation period
      </b-button>
    
      <b-button class="btn btn-light example" @click="imputeExample('tubat', 'For how long should patients be intubated?', 10);">
        Intubation
      </b-button>

      <!-- <b-button class="btn btn-light example" @click="imputeExample('intensive_care', 'For how long should patients stay at the intensive care?', 10);">
        Intensive care
      </b-button> -->

        <b-button class="btn btn-light example" @click="imputeExample('', 'What is the most common symptom?', 10);">
        Most common symptom
      </b-button>

        <b-button class="btn btn-light example" @click="imputeExample('chloroquine', 'What is the effect of chloroquine medication?', 10);">
        Chloroquine medication
      </b-button>
    </div>



    <div class="row search">
      <b-form @submit="onSubmit">
        <b-input-group>
          <b-form-input v-model="query" placeholder="Articles should contain any of these words..."></b-form-input>
          <b-form-input v-model="question" placeholder="Formulate a question..."></b-form-input>
          <b-form-input v-model="n_results" placeholder="Number of results"></b-form-input>

          <b-input-group-append>
            <b-button class="gradient-fill background hover" @click="onSubmit">
              <b-spinner v-if="status == 'loading'" small :variant="'white'" label="Small Spinner"></b-spinner>
              <i v-else class="fa fa-search"></i>
            </b-button>
          </b-input-group-append>
        </b-input-group>
      </b-form>
      <br>
    </div>

      <div v-if="status == 'done' && query != '' && question != ''" class="row">
            <div class="cards-row">
            <div class="card" v-for="prediction in predicitons" :key="prediction.id">
              <img src='https://digitalsynopsis.com/wp-content/uploads/2017/02/beautiful-color-gradients-backgrounds-091-eternal-constance.png' class="card-image" />
              <div class="card-title">
                {{ prediction['answer'] }}
              </div>
              <div class="card-desc">
                <p class="mb-0"><i class="fas fa-user-edit mr-2"></i>{{ prediction['authors'] }}</p>
                <p class="mb-0"><i class="fas fa-book-medical mr-2"></i>{{ prediction['journal'] }}</p>
                <p class="mb-0"><i class="fas fa-calendar mr-2"></i>{{ prediction['publish_time'] }}</p>
                <p class="mb-0"><i class="fas fa-external-link-alt mr-2"></i>{{ prediction['doi'] }}</p>

                <div class="journal-title">{{ prediction['title'] }}</div>
                <p><span style="font-weight:bold;">Abstract:</span> <span v-html="prediction['abstract']">></span> </p>
              </div>
            </div>

          </div>
        </div>
    </div>
</template>

<script>
import axios from "axios";

export default {
  name: "CdqaUI",
  title: 'COVID-19 Q&A',
  components: {
  },
  props: {
    api_endpoint: {
      type: String,
      default: "http://104.44.138.169:5000/api"
    },
    queries_examples: {
      type: Array,
      default: function () {
        return ['For how long should patients be intubated?', 'What is the effect of malaria medication on patients?']
      }
    },
  },
  data() {
    return {
      query: "",
      question: "",
      n_results: 10,
      show: false,
      status: "started",
      answer: "",
      title: "",
      paragraph: "",
    };
  },
  methods: {

    imputeExample: function (exampleQuery, exampleQuestion, exampleNResults=10) {
      this.query = exampleQuery;
      this.question = exampleQuestion;
      this.n_results = exampleNResults;
    },

    make_span_abstract: function (abstract, answer) {
      this.start_span = '<span class="answer">'
      this.stop_span = "</span>"
      
      this.len_answer =  answer.length
      this.start_pos_span =  abstract.lastIndexOf(answer)
      this.end_pos_span = this.start_pos_span + this.start_span.length + this.answer.length + this.stop_span.length

      return abstract.substring(0, this.start_pos_span) + this.start_span + answer + this.stop_span + abstract.substring(this.end_pos_span, )
    },


    onSubmit(evt) {
      evt.preventDefault();
      this.status = "loading";

      var api_endpoint = this.api_endpoint

      let self = this;
      axios
        .get(api_endpoint, { params: { query: self.query, question: self.question, n_results: self.n_results } })
        .then(function(response) {
          self.n_results = response.data.n_results;
          self.query = response.data.query;
          self.question = response.data.question;
          self.predicitons = response.data.predicitons;

          var data = response.data;

          console.log(data)
          self.status = "done";
        })
        .catch(function() {
          alert("Something went wrong. Change your search and try again...");
        });
    }
  }
};
</script>
<style>

body { 
    font-family: 'Roboto';
  }

.covid-research {
  text-align: center;
}

.covid-research > h1 {
  font-size: 600%;
  width: 100%;
  text-align: center;
}
.covid-research > h2 {
  font-size: 200%;
  width: 100%;
  text-align: center;

}
.covid-research > p {
    margin-left: auto;
  margin-right: auto;
  width: 75%;
  text-align: center;

}

.examples {

  padding-left: 50px;
  margin-right: 50px;
  text-align: center;
}

.btn  {
  margin-right:10px;
}

.search {
  display: block;
  margin-left: auto;
  margin-right: auto;
  text-align: center;
  width: 100%;
}

input {
  height: 55px;
}

.journal-title {
  font-weight: bold;
  color: #666;
  text-align: center;
  padding-top: 30px;
}

.answer {
  color:blueviolet;
}

.row {padding-top: 10px; padding-bottom: 3px;}

.card{
  display:inline;
  width: 50%;
};

.cards-row {
  width: 100%;
	background-color: #fff;
	margin: 0px auto;
}

.card {
	display: inline-block;
  float: left;
	width: 30%;
	min-width: 400px;
	/* height: 330px; */
	border: 1px solid transparentize(grey,0.7);
	box-sizing: border-box;
	box-shadow: 2px 2px 25px 0px transparentize(black,0.3);
	border-radius: 3px;
	margin: 1em 1.5%;
	animation: scl 0.5s ease-in-out;
	transform-origin: left center;
	background-color: #fff;
  box-shadow: 10px 10px 5px #aaa;
}
.card-title {
		margin-top: -1.5em;
		padding-bottom: 0.5em;
		padding-left: 0.5em;
		color: #fff;
		font-size: 2em;
}
.card-image {
		width: 100%;
		height: 162px;
}

.card-desc {
		padding: 0 1em 1em 1em;
		border-bottom: 1px solid transparentize(grey,0.7);
		height: 400px;
		overflow: hidden;
		text-align: justify;
}

button {
  margin:5px;
  background-color: #999;
}

.btn {
  padding-right: 15px;
}

form {
  width: 90%;
  margin-left: auto;
  margin-right: auto;

}

.form-control:focus {
  border-color: #777 !important;
  box-shadow: 0 0 5px #777 !important;
}

</style>

