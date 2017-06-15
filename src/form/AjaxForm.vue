<style>

</style>
<template>
    <div>
        <div :class="config.inputsDivClassName">
            <div v-for="(input, index) in inputs" :class="input.div && input.div.className ? input.div.className : ''">
                <label  v-if="input.label"
                        :class="input.label.className ? input.label.className : ''"
                        :for="'ajax-form-input-field-' + domId + '-' + index"
                >
                    {{ input.label.name }}
                </label>
                <input
                        :class="'ajax-form-input-field ' + (input.className ? input.className : '')"
                        :type="input.type"
                        :name="input.name"
                        :id="'ajax-form-input-field-' + domId + '-' + index"
                        :required="input.required"
                />
            </div>
        </div>
        <div :class="'ajax-form-error-field ' + (config.errorFieldClassName ? config.errorFieldClassName : '')">

        </div>
        <div :class="config.buttonsDivClassName">
            <a href="#" v-if="buttons.cancel" :id="'ajax-form-cancel-button-' + domId" @click.prevent="cancel()" :class="buttons.cancel.className">{{ buttons.cancel.label }}</a>
            <a href="#" v-if="buttons.confirm" :id="'ajax-form-confim-button-' + domId" @click.prevent="confirm()" :class="buttons.confirm.className">{{ buttons.confirm.label }}</a>
        </div>
    </div>
</template>

<script>
    let randomstring = require("randomstring");

    let axios = require('axios');
    axios.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';

    export default {
        created() {

        },
        data(){
            return {
                domId: randomstring.generate(7),
                fired: false
            };
        },
        props: ['inputs', 'buttons', 'config'],
        computed: {

        },
        methods: {
            cancel(){
                if(this.buttons.cancel && this.buttons.cancel.action){
                    this.buttons.cancel.action(this);
                }else{
                    if(this.$router){
                        this.$router.go(-1);
                    }
                }
            },
            _validateInput(inputData){
                if(inputData.required){
                    switch (inputData.type){
                        case 'text':
                            if(inputData.value.trim() !== ''){
                                return true;
                            }else{
                                return false;
                            }
                            break;
                    }
                }else{
                    return true;
                }
            },
            _buildPostObject(data){
                let obj = {};
                for(let index in data){
                    let el = data[index];
                    obj[el.name] = el.value;
                }
                return obj;
            },
            confirm(){
                let thisComponent = this;

                if(thisComponent.fired){
                    return;
                }
                thisComponent.fired = true;

                if(thisComponent.buttons.confirm && thisComponent.buttons.confirm.action){
                    thisComponent.buttons.confirm.action(thisComponent);
                }
                let $dom = $(this.$el);
                let $inputs = $dom.find('.ajax-form-input-field')
                let data = [];
                $inputs.each(function () {
                    let $input = $(this);
                    $input.removeClass('ajax-form-input-field-error');
                    if(thisComponent.config.inputErrorClass){
                        $input.removeClass(thisComponent.config.inputErrorClass);
                    }
                    let id = $input.attr('id');
                    let required = $input.prop('required');
                    let type = $input.attr('type')
                    let name = $input.attr('name');
                    let value = $input.val();

                    data.push({
                        id: id,
                        required: required,
                        type: type,
                        name: name,
                        value: value
                    });
                });

                let ctrl = true;
                for(let index in data){
                    let inputData = data[index];
                    if(!this._validateInput(inputData)){
                        let $input = $('#' + inputData.id);
                        $input.addClass('ajax-form-input-field-error');
                        if(thisComponent.config.inputErrorClass){
                            $input.addClass(thisComponent.config.inputErrorClass);
                        }
                        ctrl = false;
                    }
                }

                if(!ctrl){
                    thisComponent.fired = false;
                    return;
                }

                let $errorField = $dom.find('.ajax-form-error-field');

                axios.post(
                    thisComponent.config.postUrl,
                    thisComponent._buildPostObject(data)
                ).then((response) => {
                    if(thisComponent.config.responseOkAction){
                        thisComponent.config.responseOkAction(response, thisComponent);
                    }else if(thisComponent.$router && thisComponent.config.responseOkRoute){
                        thisComponent.$router.push(thisComponent.config.responseOkRoute);
                    }
                }).catch((error) => {
                    $errorField.html(thisComponent.config.genericErrorMessage ? thisComponent.config.genericErrorMessage(error, thisComponent) : error);
                }).then(() => {
                    thisComponent.fired = false;
                })
            }
        }
    }
</script>