<template>
  <div>
    <or-button @click="$refs.regexesBuilder.open()" color="primary">
      Regexes Builder
    </or-button>
    <or-modal id='or-modal' ref="regexesBuilder" title="Regexes Builder">
      <div class="header_info">
        <div>
            <h3>Target(s)</h3>
        </div>
      </div>
      <div class="list--empty" v-if="!fr_targets.length">
        List is empty
      </div>
      <or-list  :canDeleteItems="false" :readonly="readonly"  addButtonLabel="Add new target" v-model="fr_targets">
          <template scope="item">
              <div else class="flex_box">
                  <div class="flex_box__single" v-if="!item.item.regExp">
                      <or-textbox floatingLabel :invalid="validate(item)" placeholder="Target" disableCodeMode v-model="item.item.target"
                      :steps="steps" :step-id="stepId" :readonly="readonly" >
                      </or-textbox>
                  </div>
                  <div class="flex_box__single" v-else>
                      <or-textbox floatingLabel class="or-text-expression--reg" placeholder="RegExp" disableCodeMode
                      v-model="item.item.target" :invalid="validate(item)" :steps="steps" :step-id="stepId" :readonly="readonly">
                      </or-textbox>
                      <or-select multipleDelimiter="" multiple placeholder="Flags" v-model="item.item.flags" :invalid="validate(item)"
                      :steps="steps" :step-id="stepId" :disabled="readonly" :options="flags">
                          <template slot-scope="props" slot="option">
                              <div>{{ props.option.title }}
                                  <br>
                                  <span class="help">{{ props.option.help }}</span>
                              </div>
                              <or-icon class="icon_selected" v-if="props.selected" icon="done"></or-icon>
                          </template>
                      </or-select>
                  </div>
                  <or-icon-button  disableRipple icon="more_vert" class="flat btn" has-dropdown :disabled="readonly" type="secondary" :ref="'dropdownMenu1'+item.index">
                    <or-menu disableRipple contain-focus has-icons @select="selectMenu"  slot="dropdown"  :options="menuOptions(item.item,item.index)" @close="closeMenu(item)"></or-menu>
                  </or-icon-button>
              </div>
          </template>
          
          <template slot="footer">
            <or-button :disabled="readonly" disableRipple icon="add" type="secondary" class="list-add-target" color="primary" has-dropdown ref="dropdownButton1">
              <or-menu @select="addTarget" class="list-add-menu" contain-focus slot="dropdown" :options="menuTarget" @close="closeAddTargetDropdown"></or-menu>
              Add Target
            </or-button>
          </template>
      </or-list>
    </or-modal>
  </div>


</template>

<script>
  import * as _ from 'lodash';

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
      }
    },

    data () {
      return {
        fr_targets: [],
        prev_targets:[],
        menuTarget: [
          {
            label:"Contains",
            value:"default"
          },
          {
            label:"Start with",
            value:"start"
          },
          {
            label:"End with",
            value:"end"
          },
        {
            label:"Equal",
            value:"equal"
          },
          {
            type:"divider"
          },
          {
            label:"Custom RegExp",
            value:"custom"
          } 
        ],
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
    },

    methods : {
      addTarget(value){
        let obj;
        switch(value.value){
          case('start'):
            obj = {
              dirty:false,
              target:"^your-text",
              replace:"``",
              regExp:true,
              flags:["g","m"],
              caseSentensive:true
            };  
            break;
          case('end'):
            obj = {
              dirty:false,
              target:"your-text$",
              replace:"``",
              regExp:true,
              flags:["g","m"],
              caseSentensive:true
            };  
            break;
          case('custom'):
            obj = {
              dirty:false,
              target:"",
              replace:"``",
              regExp:true,
              flags:["g"],
              caseSentensive:true
            };  
            break;
          case('equal'):  
            obj = {
              dirty:false,
              target:"^your-text$",
              replace:"``",
              regExp:true,
              flags:["g","m"],
              caseSentensive:true
            };
            break
            default:
            obj = {
              dirty:false,
              target:"",
              replace:"``",
              regExp:false,
              flags:["g"],
              caseSentensive:true
            };
        }
        this.fr_targets.push(obj);
        this.prev_targets.push(obj);
      },
      closeAddTargetDropdown(){
        if(this.$refs.dropdownButton1){
          this.$refs.dropdownButton1.closeDropdown();
        }
      },
      validate(item){
        return !item.item.target && !(this.isNew && !this.fr_targets[item.index].dirty);
      },
      menuOptions(item,index){
        return [
          {
            label:item.regExp?"Use Simple Text":"Use RegExp",
            icon:item.regExp?"text_fields":"icon-link",
            iconProps:{
              useSvg:!item.regExp
            },
            index,
            value:"regExp"
          },
          {
            label:item.caseSentensive?"Case Insensetive":"Case Sensetive",
            icon:item.caseSentensive?"format_strikethrough":"title",
            index,
            value:"case"
          },
          {
            type:"divider"
          },
          {
            label:"Remove",
            icon:"delete_forever",
            value:"delete",
            index,
            disabled:this.fr_targets.length<2
          }
        ]
      },
      selectMenu(option){
        switch(option.value){
          case("regExp"):
            this.fr_targets[option.index].regExp=!this.fr_targets[option.index].regExp;
            break;
          case("case"):
            this.fr_targets[option.index].caseSentensive=!this.fr_targets[option.index].caseSentensive;
            if(this.fr_targets[option.index].flags.includes("i")){
              this.fr_targets[option.index].flags.splice(this.fr_targets[option.index].flags.indexOf("i"),1)
            }else{
              this.fr_targets[option.index].flags.push("i")
            }
            break;
          case("delete"):
            console.log(option.index);
            this.fr_targets.splice(option.index,1)
            break;
        }
      },
      closeMenu(item){
        for(let i = 0; i < item.items.length; i++){
          if(this.$refs['dropdownMenu1'+i]){
            this.$refs['dropdownMenu1'+i].closeDropdown();
          }
          if(this.$refs['dropdownMenu2'+i]){
            this.$refs['dropdownMenu2'+i].closeDropdown();
          }
        }
      },
    },

    watch : {
        
    },

    mounted () {
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
      booInput    : template.booInput,
      hasBooInput : template.hasBooInput
    }
  };

  export const toJSON = ({schema, inputData, context}) => {
    return ({
      booInput    : schema.booInput,
      hasBooInput : schema.hasBooInput
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
  .list--empty{
    display:flex;
    justify-content:center;
    align-items:center;
    height:63px;
    width:100%;
    color:#91969d;
  }
  .btn {
    width: 24px;
  }
  .flex_box {
    display: flex;
    flex-wrap:wrap;
    width: calc(100% - 24px);
    .ui-select {
      width: 103px;
      margin:0;
      margin-left:-3px;
      .ui-select__display {
        height: 36px;
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
    }

    .or-text-expression, .ui-textbox {
      
      width: 100%;
      position: relative;
      margin:0;
      flex-direction: row-reverse;

      .header {
        position: absolute;
        right: 0;
        z-index: 2;
      }
      .input-wrapper {
        width: 100%;
      }
      &--reg {
        width: calc(100% - 100px);
        position: relative;
        margin: 0;
        flex-direction: row-reverse;
        .header {
          position: absolute;
          right: 0;
          z-index: 2;
        }
        .input-wrapper {
          width: 100%;
        }
      }
      .or-editable-wrapper{
        height:36px;
      }
    }
    &__item {
      display: flex;
      flex-grow:1;
      height: 42px;
      width:200px;
      padding: 3px;
      .or-editable-wrapper.single-line {
        padding-right: 30px;
      }
    }
    &__single{
      display:flex;
      width: calc(100% - 24px);
      padding:3px;
      .or-editable-wrapper.single-line {
        padding-right: 30px;
      }
    }
  }

</style>
