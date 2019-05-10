<template>
  <div>

    <div class="input-group mb-3" id="loadButton">
      <input list="ttlDocs" type="text" class="form-control" placeholder="Brick TTL" aria-label="Brick TTL" aria-describedby="loadButton" id="ttlBar">
      <datalist id="ttlDocs">
        <option value="https://raw.githubusercontent.com/BuildSysUniformMetadata/Brick/master/dist/Brick.ttl">BRICK 1.0</option>
        <option value="https://raw.githubusercontent.com/BrickSchema/brick-owl-dl/master/Brick.ttl">BRICK 2.0</option>
      </datalist>
      <div class="input-group-append">
        <button class="btn btn-outline-primary" type="button" v-on:click="fetchRdf">Load Docs</button>
      </div>
    </div>
    <input autocomplete="off" hidden=true type="text" class="form-control shadow p-3 mb-8 bg-white rounded" placeholder="Search" aria-label="Search" aria-describedby="basic-addon2" id="searchBar" v-on:keyup="filterList"/>
    <modal height="auto" id="modal" name="details" @before-open="beforeOpen">
        <b-card style="height:inherit;">
          <div class="label">
            <a  target="_blank" rel="noopener noreferrer" :href="current.id"><h3 class="mt-0">{{ current.label }}</h3></a>
          </div>


          <div class="detail" v-if="current.name">
            <div class="detailKey">
              Name:
            </div>
            <div class="detailValue">
              {{ minify(current.name) }}
            </div>
          </div>
          <div class="detail" v-if="current.type">
            <div class="detailKey">
              Type:
            </div>
            <div class="detailValue">
              <a target="_blank" rel="noopener noreferrer" :href="current.type">
                {{ minify(current.type).split('_').join(' ') }}
              </a>
            </div>
          </div>
          <div class="detail" v-if="current.subClassOf">
            <div class="detailKey">
              Parent Class:
            </div>
            <div class="detailValue" v-on:click="show(current.subClassOf)">
              <a target="_blank" rel="noopener noreferrer" :href="current.subClassOf">
                {{ minify(current.subClassOf).split('_').join(' ') }}
              </a>
            </div>
          </div>
          <div class="detail" v-if="current.definition.length">
            <div class="detailKey">
              Definition:
            </div>
            <div class="detailValue">
              {{ current.definition }}
            </div>
          </div>


        </b-card>
    </modal>
    <ul id="results" hidden="true">
      <li v-for="doc in brickDocs">
        <!--<b-card bg-variant="dark" text-variant="white"><h4>{{ doc.label }}</h4>-->
          <!--<b-card-text>-->
            <!--{{ doc.definition }}-->
          <!--</b-card-text>-->
          <!--<b-button :href="doc.id" variant="primary">Learn more</b-button>-->
        <!--</b-card>-->

        <!--<b-card bg-variant="info" text-variant="white" :header="doc.label" class="text-center">-->
        <!--<b-card-text>{{ doc.definition }}</b-card-text>-->
        <!--</b-card>-->


        <div>
          <b-card class="shadow p-3 mb-4 bg-white rounded">
            <div class="label">
              <!--<a  target="_blank" rel="noopener noreferrer" :href="doc.id">-->
                <h3 class="mt-0"  v-on:click="show(doc)">{{ doc.label }}</h3>
              <!--</a>-->
            </div>


            <div class="detail" v-if="doc.definition.length">
              <div class="detailKey">
                Definition:
              </div>
              <div class="detailValue">
                {{ doc.definition }}
              </div>
            </div>


          </b-card>
        </div>

      </li>
    </ul>
  </div>

</template>

<script>
  import Vue from 'vue'
  import Fuse from 'fuse.js'
  import N3 from 'n3'
  import axios from 'axios'
  import VModal from 'vue-js-modal'
  import VueTextareaAutosize from 'vue-textarea-autosize'

  Vue.use(VueTextareaAutosize)
  Vue.use(VModal)
  export default {
    name: "Search.vue",
    data() {
      return {
        fuse: null,
        quadStore: new N3.Store(),
        docs: [],
        current: {
          "id": "https://brickschema.org/schema/1.0.3/Brick#VAV_Zone_Temperature_Setpoint",
          "name": "VAV_Zone_Temperature_Setpoint",
          "type": "http://www.w3.org/2002/07/owl#Class",
          "label": "VAV Zone Temperature Setpoint",
          "subClassOf": "https://brickschema.org/schema/1.0.3/Brick#Zone_Temperature_Setpoint",
          "definition": "A temperature setpoint in charge of a zone.",
          "alias": " VAV ZTS"
        },
        filteredDocs: []
      }
    },
    methods: {
      beforeOpen (event) {
        this.current = event.params.id;
      },
      show (id) {
        this.$modal.show('details', {id});
      },
      hide () {
        this.$modal.hide('details');
      },
      filterList() {
        var query = document.getElementById('searchBar').value;
        this.filteredDocs = (query.length>0) ? this.fuse.search(query): this.fuse.list;
      },
      fetchRdf() {
        this.$Progress.start();
        const parser = new N3.Parser();
        axios.get("https://cors-anywhere.herokuapp.com/" + document.getElementById('ttlBar').value).then(res => parser.parse(res.data, this.store))

      },
      store(err, quad) {
        if (quad !== null) {
          this.quadStore.addQuad({...quad})
        } else {
          this.formDefinitions();
        }
      },
      formDefinitions() {
        document.getElementById('loadButton').hidden = true;
        document.getElementById('searchBar').hidden = false;
        document.getElementById('results').hidden = false;
        this.quadStore.getQuads(null, 'http://www.w3.org/2004/02/skos/core#definition').forEach(quad => {
          var doc = {id: quad.subject.id, name: this.minify(quad.subject.id)};
          this.quadStore.getQuads(quad.subject.id).forEach(details => {
            doc[this.minify(details.predicate.id)] = details.object.id;
          });
          if (doc.label === undefined) {
            doc.label = this.minify(doc.id).split('_').join(' ');
          }
          doc.label = this.removeEncoding(doc.label);
          doc.alias = this.getAlias(doc.label);
          doc.definition = this.removeEncoding(doc.definition);
          this.docs.push(doc);
        });
        this.fuse = new Fuse(this.docs, {
          threshold: 0.4,
          keys: [{name: 'alias', weight: '0.3'}, {name: 'label', weight: '0.5'}, {name: 'definition', weight: '0.2'}],
          tokenize: true
        });
        this.filteredDocs = this.docs;
        this.$Progress.finish()

      },
      minify(IRI) {
        return IRI.split('#').pop();
      },
      removeEncoding(string) {
        var tokens = string.split('"');
        string = (tokens.length === 3) ? tokens[1] : string;
        return string.split('@').shift();
      },
      getAlias(label) {

        var alias = '';
        label.split(' ').forEach(word => {
          if (word.toUpperCase() === word)
            alias += ` ${word} `;
          else
            alias += word[0];
        });
        return alias;
      }
    },
    computed: {
      brickDocs: {
        get() {
          return this.filteredDocs;
        },
        set(newValue) {
          this.filteredDocs = newValue;
        }
      }
    }
  }


</script>

<style scoped>

  li {
    text-align: left;
  }

  ul {
    list-style: none;
  }

  #searchBar{
    margin: 40px;
    width: 96%;
  }

  .detailKey{
    float: left;
    margin-right: 10px;
    font-weight: bold;
    font-size: larger;
  }
  .detailValue{
    float: snap;
    font-size: larger;
  }
  .label{
    padding-bottom: 4px;
    color:  #4792ef;
    /*background: -webkit-linear-gradient(#4792ef, #39a3a3);*/
    /*-webkit-background-clip: text;*/
    /*-webkit-text-fill-color: transparent;*/

  }
  .label:hover{
    text-decoration: underline;
    cursor: pointer;
  }
  .detail{
    margin: 8px;
  }

  #loadButton{
    width: 60em;
  }

  #modal{
    padding: 0px;
  }

  #results{
    height: -webkit-fill-available;
  }
</style>
