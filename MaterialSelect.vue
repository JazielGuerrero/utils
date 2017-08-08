<template>
  <label>
    {{label}}
    <select :id="id" :multiple="multiple"></select>
  </label>
</template>

<script>
  export default {
    name: "material-select",
    data(){
      return{
        select: undefined,
        dataset: [],
        change: false //True every time user changes the selected value
      }
    },
    props: {
      /**
       * Specified an ID if there's more
       * than one select in the same Vue
       */
      id: {
        type: String,
        default: "default"
      },
      /**
       * Label of the select input
       */
      label: {
        type: String,
        require: true
      },
      /**
       * Array object containing the
       * options of the select
       */
      options: {
        type: Array,
        require: true
      },
      /**
       * Array object containing the
       * value of the options of the select
       */
      optionsValue: {
        type: Array,
      },
      /**
       * Specified if the select is
       * multiple
       */
      multiple: {
        type: Boolean,
        default: false
      },
      /**
       * Object model use on the parent
       * to bind to this child
       */
      model: {
        require: true
      }
    },
    watch: {
      /**
       * If the options was change update the select options
       * If the length of the dataset !== from options, options changes
       * otherwise it search option by option and check if the placeholder
       * or value of the option change
       */
      "options": function () {
        if(this.select){
          let change = this.dataset.length !== this.options.length;

          if(!change){
            for(let index = 0; index < this.dataset.length; index++){
              if(this.dataset[index].placeholder !== this.options[index] || this.dataset[index].value !== this.optionsValue[index]){
                change = true;
                break;
              }
            }
          }

          if(change){
            this.parseOptions();
            this.select.material_select();
          }
        }
      },
      /**
       * If the selected value was change programmatically
       * from the parent, update the selected value
       */
      "model": function () {
        if(this.select && !this.change && this.select.val() !== this.model){
          this.select.val(this.model);
          this.select.material_select();
        }

        this.change = false;
      }
    },
    methods: {
      parseOptions: function () {
        this.dataset.length = 0;

        //Parse the Options
        for(let index = 0 ; index < this.options.length; index++){
          this.dataset.push({
            placeholder: this.options[index],
            value: this.options[index]
          })
        }

        //Parse the Options Value if any
        if(this.optionsValue){
          if(this.dataset.length !== this.optionsValue.length){
            console.error("Dataset length is different from optionsValue[props] length");
            return;
          }

          for(let index = 0; index < this.dataset.length; index++){
            this.dataset[index].value = this.optionsValue[index];
          }
        }

        //Add the options
        this.select.empty();
        for(let index = 0; index < this.dataset.length; index++){
          this.select.append("<option value='" + this.dataset[index].value + "'>" + this.dataset[index].placeholder + "</option>");
        }
      }
    },
    mounted(){
      const component = this;

      $(document).ready(function() {
        //Set initials options
        component.select = $("#" + component.id);
        component.parseOptions();

        //Sets a selected option
        component.select.val(this.model);

        //On select change emit the new value
        component.select.on("change", function () {
          if(component.model !== $(this).val()){
            component.change = true;
            component.$emit("change", $(this).val());
          }
        });

        //Initialize Material Select
        component.select.material_select();
      });
    }
  }
</script>
