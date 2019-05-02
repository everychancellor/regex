<template>
  <div>
    <or-button 
      class="regexesBuilterBtn"
      @click="$refs.regexesBuilder.open()"
      color="primary"
      :disabled="readonly"
      :loading="isLoading"
    >Regexes Builder
    </or-button>
    <or-modal id='or-modal' ref="regexesBuilder" title="Regexes Builder" size="large">
      <or-tabs class="flex-box" fullwidth>
          <or-tab title="Basic" class="tab">
            <div class="flex">
              <or-menu :options="regexCategories" @select="categorySelect"></or-menu>
              <or-menu :options="selectedCategoryRegexes" @select="selectItem" id="item-selector"></or-menu>
               <or-select multipleDelimiter="" multiple placeholder="Flags" v-model="localSelectedFlags" :options="flags">
                  <template slot-scope="props" slot="option">
                      <div>{{ props.option.title }}
                          <br>
                          <span class="help">{{ props.option.help }}</span>
                      </div>
                      <or-icon class="icon_selected" v-if="props.selected" icon="done"></or-icon>
                  </template>
              </or-select>
            </div>
          </or-tab>

          <or-tab title="Intermediate" class="tab">
            Intermediate
          </or-tab>

          <or-tab title="Advanced" class="tab">
             <div class="flex">
              <or-select-expression
                has-search
                id="regex-list"
                extendable-options
                placeholder="Select Regex"
                :options.sync="regexList"
                :steps="steps" 
                :step-id="stepId" 
                v-model="localSelectedRegex"
                :disabled="isLoading"
              ></or-select-expression>
              <or-select multipleDelimiter="" multiple placeholder="Flags" v-model="localSelectedFlags" :options="flags">
                  <template slot-scope="props" slot="option">
                      <div>{{ props.option.title }}
                          <br>
                          <span class="help">{{ props.option.help }}</span>
                      </div>
                      <or-icon class="icon_selected" v-if="props.selected" icon="done"></or-icon>
                  </template>
              </or-select>
              <div class="manageIconContainer">
                <or-icon-button
                  @click="refreshRegexList"
                  icon="refresh"
                  color="primary"
                  class="manageBtn"
                  size="small"
                  :disabled="isLoading">
                </or-icon-button>
                <or-icon-button 
                  @click="$refs.addRegexModal.open()"
                  icon="add"
                  color="orange"
                  class="manageBtn"
                  size="small"
                  :disabled="isLoading">
                </or-icon-button>
                </div>
                <or-icon-button 
                  @click="deleteRegex"
                  icon="delete"
                  color="red"
                  class="manageBtn"
                  size="small"
                  :disabled="isLoading">
                </or-icon-button>
              </div>
          </or-tab>
        </or-tabs>
        <div class="header_info">
          <div>
            <h4>Example text</h4>
          </div>
          <or-icon-button 
            :disabled="readonly" 
            @click="exampleEdit=!exampleEdit" 
            disableRipple type="secondary" 
            :color="(exampleEdit)?'black':'default'" 
            class="flat" 
            icon="edit"
          ></or-icon-button>
        </div>
        <div class="example_text">
          <or-textbox 
            multiLine rows="6"
            v-if="exampleEdit"
            v-model="localTextExample"
            maxlength="250"
            enforceMaxlength="true"
          ></or-textbox>
          <p v-else v-html="highlight"></p>
        </div>
        <div 
          class="list--empty" 
          v-if="!exampleResult.length">
          No matches
        </div>
        <div id="example-res">
          <div 
            v-for="(match,i) in exampleResult">
            <table>
              <thead>
                <tr>
                  <th colspan="3"><span class="match_header">Match {{++i}}</span></th>
                </tr>
              </thead>
              <tbody>
                <tr>
                  <td><span style="background: rgb(116, 196, 255)">Full match</span></td>
                  <td>`{{match.match}}` <or-icon icon="trending_flat"></or-icon> `{{match.replace}}`</td>
                  <td><small>{{match.range}}</small></td>
                </tr>
                <tr v-for="(group,i) in match.groups">
                  <td><span style="background:rgb(198, 233, 157)">Group {{++i}}</span></td>
                  <td colspan="2">`{{group}}`</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
    </or-modal>
    <or-modal ref="addRegexModal" title="Create Regex">
      <or-textbox class="input-exp" label="Friendly Name" v-model="regexName" :disabled="isLoading"></or-textbox>
      <or-textbox class="input-exp" label="Regular Expression" v-model="newRegex" :disabled="isLoading"></or-textbox>
      <or-button
        @click="addRegex"
        size="small"
        color="primary"
        :loading="isLoading"
      >Save in User Library</or-button>
    </or-modal>
  </div>


</template>

<script>
  import * as _ from 'lodash';
  import uuid from 'uuid';

  import {validators} from '_validators';
  
  const {required, each, jsExpressionNonEmptyString, generateValidators} = validators;

  export default {
    components : {
    },

    props :   {
      template : {
        type : Object,
        default () {
            return {}
        }
      },
      schema : {
        type : Object,
        default () {
            return {}
        }
      },
      stepId : '',
      steps  : {
        type : Array,
        default () {
            return []
        }
      },
      readonly : {
        type : Boolean,
        default : false
      },
      isLoading: {
        type: Boolean,
        default: false
      }
    },

    data () {
      return {
        regexCategories: [
          {
            id: "Contacts",
            label: "Contacts"
          }, 
          {
            id: "Web",
            label: "Web",
          }
        ],
        selectedCategoryRegexes: [],
        regexLib: [
          {
            name: "Email",
            regex: "(?:[a-z0-9!#$%&\'*+\/=?^_`{|}~-]+(?:\\.[a-z0-9!#$%&\'*+\/=?^_`{|}~-]+)*|\"(?:[\\x01-\\x08\\x0b\\x0c\\x0e-\\x1f\\x21\\x23-\\x5b\\x5d-\\x7f]|\\\\[\\x01-\\x09\\x0b\\x0c\\x0e-\\x7f])*\")@(?:(?:[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\\.)+[a-z0-9](?:[a-z0-9-]*[a-z0-9])?|\\[(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?|[a-z0-9-]*[a-z0-9]:(?:[\\x01-\\x08\\x0b\\x0c\\x0e-\\x1f\\x21-\\x5a\\x53-\\x7f]|\\\\[\\x01-\\x09\\x0b\\x0c\\x0e-\\x7f])+)\\])",
            category: "Contacts"
          },
          {
            name: "Mobile phone number",
            regex: "[+]*[(]{0,1}[0-9]{1,4}[)]{0,1}[-\s\./0-9]*",
            category: "Contacts"
          },
          {
            name: "IP Address",
            regex: "(?:(?:2(?:[0-4][0-9]|5[0-5])|[0-1]?[0-9]?[0-9])\\.){3}(?:(?:2([0-4][0-9]|5[0-5])|[0-1]?[0-9]?[0-9]))",
            category: "Web"
          },
          {
            name: "URL",
            regex: "(?:(?:https?|ftp|file):\/\/|www\.|ftp\.)(?:\([-A-Z0-9+&@#\/%=~_|$?!:,.]*\)|[-A-Z0-9+&@#\/%=~_|$?!:,.])*(?:\([-A-Z0-9+&@#\/%=~_|$?!:,.]*\)|[A-Z0-9+&@#\/%=~_|$])",
            category: "Web"
          },
          {
            name: "YouTube Video URL",
            regex: "(https?\:\/\/)?((www\.)?youtube\.com|youtu\.?be)\/.+",
            category: "Web"
          }
        ],
        storeTable: '_regexes-user-library',
        regexList: '',
        regexName: '',
        newRegex: '',
        exampleResult: '',
        exampleEdit: false,
        exampleReplace: '$&',
        flags:[
          {         
            value:"g",
            label:"g",
            help:"Don't return after first match",
            title:"global"
          },{          
            value:"m",
            label:"m",
            help:"^ and $ match start/end of line",
            title:"multi line"
          },{          
            value:"i",
            label:"i",
            help:"Case insensitive match",
            title:"insensitive"
          },{          
            value:"y",
            label:"y",
            help:"Proceed matching from where previous match ended only",
            title:"sticky"
          },{          
            value:"u",
            label:"u",
            help:"Match with full unicode",
            title:"unicode"
          },{
            label:"s",
            value:"s",
            help:"Dot matches newline",
            title:"single line"
          }
        ]
      }
    },

    computed : {
      localSelectedRegex : {
        get () {
          return this.schema['selectedRegex'] || this.template.selectedRegex;
        },
        set (newValue) {
          this.$set(this.schema, 'selectedRegex', newValue);
        }
      },
      localTextExample : {
        get () {
          if ( (this.schema['textExample'] == undefined) && (this.template.textExample == undefined) ) return "Lorem ipsum dolor sit amet consectetur, adipisicing elit. Placeat exercitationem assumenda incidunt hic nulla alias labore perferendis, vitae non voluptas nobis. Atque voluptas nisi numquam soluta unde sit, nesciunt quibusdam?";
          return this.schema['textExample'] || this.template.textExample;
        },
        set (newValue) {
          this.$set(this.schema, 'textExample', newValue);
        }
      },
      localSelectedFlags : {
        get () {
          return this.schema['selectedFlags'] || this.template.selectedFlags || [];
        },
        set (newValue) {
          this.$set(this.schema, 'selectedFlags', newValue);
        }
      },
      highlight() {
        let target = this.parsedSelectedRegex;
        let result = [];
        let regExp = new RegExp(target, this.localSelectedFlags.join(''));
        if(!target) {
          this.exampleResult=[];
          return this.localTextExample;
        }
        let html = this.localTextExample.replace(regExp, (match,...args) => {
          let replace = match.replace(regExp, this.exampleReplace);
          result.push({
            groups:args.slice(0,-2),
            range:`${args.slice(-2,-1)[0]} - ${match.length+args.slice(-2,-1)[0]}`,
            match,
            replace
          });
          return `<span style="border: .5px dashed #f6f6f6;
    background: #64b2da;">${replace}</span>`;
        });
        this.exampleResult = result;
        return html;
      },
      parsedSelectedRegex() {
        return this.localSelectedRegex ? this.localSelectedRegex.split('::')[2] : ''; 
      }
    },

    methods : {
      async categorySelect(item) {
        this.selectedCategoryRegexes = this.regexLib.filter(val => {
          return val.category === item.id;
        }).map(val => {
          return {
            id: val.regex,
            label: val.name
          }
        })
      },
      async selectItem(item) {
        this.localSelectedRegex = '::::' + item.id;
      },
      async addRegex() {
        this.isLoading = true;
        try {
          let bodyData = {
            name: this.regexName,
            regex: this.newRegex,
            date_created: new Date().toISOString(),
            user_id: this.$flow.userId
          };
          await this.requestToStore('post', bodyData);
        } catch (error) {
          console.error(error);
        } finally {
          this.isLoading = false;
        }
        this.closeModal('addRegexModal');
        this.regexName = '';
        this.newRegex = '';
        this.refreshRegexList();
      },
      async refreshRegexList() {
        this.isLoading = true;
        this.regexList = [];
        try {
          let res = await this.requestToStore('get');
          let tempRegexList = [];
          _.forEach(res.body.value.records, (item) => {
            let itemList = _.split(item.key, '::');
            tempRegexList.push({ value: item.key, label: `${itemList[1]} - /${itemList[2]}/` });
          });

          this.regexList = tempRegexList;
          if (this.localSelectedRegex && !_.find(tempRegexList, {value: this.localSelectedRegex})) {
            this.localSelectedRegex = '';
          }
        } catch (error) {
          console.error(error);
        } finally {
          this.isLoading = false;
        }

      },
      async deleteRegex() {
        if (this.localSelectedRegex) {
          try {
            this.isLoading = true;
            await this.requestToStore('delete');
            this.refreshRegexList();
          } catch (error) {
            console.error(error);
          } finally {
            this.isLoading = false;
          }
        }
      },
      async requestToStore(httpType = 'get', data) {
        if (httpType === 'get') {
          let url = this.$flow.api.settings.sdkApiUrl + `/storage/${this.storeTable}`;
          return await this.$http.get(url,
            {
              headers: { authorization: this.$flow.api.settings.userToken }
            }
          );
        } else if (httpType === 'post') {
          let tempId = uuid.v4();
          let tmpSelectedRegex = `${tempId}::${this.regexName}::${this.newRegex}`;
          this.localSelectedRegex = tmpSelectedRegex;
          let url = this.$flow.api.settings.sdkApiUrl + `/storage/${this.storeTable}/${tmpSelectedRegex}`;
          return await this.$http.post(url, {value: data},
            {
              headers: { authorization: this.$flow.api.settings.userToken }
            }
          );
        } else if (httpType === 'delete') {
          let url = this.$flow.api.settings.sdkApiUrl + `/storage/${this.storeTable}/${this.localSelectedRegex}`;
          return await this.$http.delete(url,
            {
              headers: { authorization: this.$flow.api.settings.userToken }
            }
          );
        }
      },
      closeModal(ref) {
        this.$refs[ref].close();
      },
    },

    watch : {
        
    },

    mounted () {
      this.refreshRegexList();
      console.log('this', this);
    },

    validations () {
      return {
        schema : validator(this.template)
      };
    }

  };

  export const data = template => {
    return {
      textExample: template.testExample,
      selectedFlags: template.selectedFlags,
      selectedRegex    : template.selectedRegex
    }
  };

  export const toJSON = ({schema, inputData, context}) => {
    return ({
      textExample      : schema.testExample,
      selectedFlags    : schema.selectedFlags,
      selectedRegex    : schema.selectedRegex
    })
  }

  export const validator = (template) => {
    return {};  
  };

  export const meta = {
      name    : 'regexes-external-component',
      type    : 'onereach-studio-form-editor',
      version : '1.0.0'
  };

</script>

<style scoped lang="scss" rel="stylesheet/scss">
  .header_info {
    display: flex;
    align-items: center;
    justify-content: space-between;
  }
  #item-selector {
    flex-grow: 1
  }
  #example-res {
    min-height: 200px
  }
  .flex {
    display: flex;
  }
  #regex-list {
    flex-grow: 1;
  }
  .manageBtn {
    margin-left: 5px;
  }
  .regexesBuilterBtn {
    margin: 5px;
  }
  .list-item {
    padding: 10px 0 !important;
  }
  .list--empty{
    display:flex;
    justify-content:center;
    align-items:center;
    height:63px;
    width:100%;
    color:#91969d;
  }
  .highlightText {
    border: .5px dashed #f6f6f6;
    background: #64b2da;
  }
  .example_text{
    background-color: #F6F6F6;
    padding: 8px 10px;
    margin: 15px 0;
    textarea{
      line-height: 19px;
    }
  }
  .tab {
    height: 300px;
  }
  .icon_selected {
    color: rgb(126, 211, 33);
    font-size: 1.25em;
    font-weight: 900;
  }
  .help {
    line-height: 14px;
    font-size: 80%;
  }

</style>