<style>

</style>
<template>
    <div>
        <div>
            <div v-for="(input, index) in inputs">
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
        <div>
            <button v-if="buttons.cancel" :id="'ajax-form-cancel-button-' + domId" @click="cancel()" :class="buttons.cancel.className">{{ buttons.cancel.label }}</button>
            <button v-if="buttons.confirm" :id="'ajax-form-confim-button-' + domId" @click="confirm()" :class="buttons.confirm.className">{{ buttons.confirm.label }}</button>
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
                domId: randomstring.generate(7)
            };
        },
        props: ['inputs', 'buttons', 'config'],
        computed: {

        },
        methods: {
            cancel(){
                if(buttons.cancel && buttons.cancel.action){
                    buttons.cancel.action(this);
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
                var obj = {};
                for(let index in data){
                    let el = data[index];
                    obj[el.name] = el.value;
                }
            },
            confirm(){
                if(buttons.confirm && buttons.confirm.action){
                    buttons.confirm.action(this);
                }
                let $dom = $(this.$el);
                let $inputs = $dom.find('.ajax-form-input-field')
                let data = [];
                $inputs.each(function () {
                    let $input = $(this);
                    $input.removeClass('ajax-form-input-field-error');
                    if(inputs.inputErrorClass){
                        $input.removeClass(config.inputErrorClass);
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
                        if(config.inputErrorClass){
                            $input.addClass(config.inputErrorClass);
                        }
                        ctrl = false;
                    }
                }

                if(!ctrl){
                    return;
                }

                let $errorField = $dom.find('.ajax-form-error-field');

                axios.post(
                    this.config.postUrl,
                    this._buildPostObject(data)
                ).then((response) => {
                    if(response.code === 200){
                        if(config.responseOkAction){
                            config.responseOkAction(this);
                        }else if(this.$router && config.responseOkRoute){
                            this.$router.go(config.responseOkRoute);
                        }
                    }else{
                        $errorField.html(config.responseErrorMessage ? config.genericErrorMessage(response) : response.statusText);
                    }
                }).catch((error) => {
                    $errorField.html(config.genericErrorMessage ? config.genericErrorMessage(error) : error);
                })

            }
        }
    }
</script>