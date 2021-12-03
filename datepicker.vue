<template>
    <datepicker
        ref="datepicker"
        v-model="model"
        v-bind="props_full"
        :required="required"
        :config="_config"
        :class="dclass"
        :placeholder="placeholder"
        @on-close="() => { emitClose(); $emit('on-close') }"
        @on-open="$emit('focusin')"
        @blur="$emit('focusout')"
        @on-day-create="(dObj, dStr, fp, dayElem) => $emit('on-day-create', dObj, dStr, fp, dayElem)"
        :events=" ['onChange','onDayCreate']"
        readonly="readonly"
    />
</template>

<script>
    import Datepicker from 'vue-flatpickr-component';

    import 'flatpickr/dist/flatpickr.css';

    import $ from 'jquery';

    import ConfirmDatePlugin from 'flatpickr/dist/plugins/confirmDate/confirmDate';

    import 'flatpickr/dist/plugins/confirmDate/confirmDate.css';

    import { Spanish } from 'flatpickr/dist/l10n/es.js';

    export default {
        components: {
            Datepicker,
        },
        props: {
            dclass: {
                default: () => 'form-control',
                type: [String, Object]
            },
            value: {
                default: () => '',
                type: [String, Array]
            },
            config: {
                default: () => new Object(),
                type: Object
            },
            placeholder: {
                default: () => '',
                type: String
            },
            lang_value: {
                default: () => 'en',
                type: String,
                required: false
            },
            lang_texts: {
                default: () => new Object(),
                type: Object,
                required: false
            }
        },
        data(){
            return({
                state: {
                    close: false
                }
            });
        },
        computed:{
            props_full(){
                let data = {};

                Object.assign(data, this.$props, this.$attrs.props);

                return data;
            },
            texts(){
                if(Object.keys(this.lang_texts).length){
                    return this.lang_texts
                }
            },
            _config(){
                return Object.assign(JSON.parse(JSON.stringify(this.config), (k ,v) => {
                    if(k!=="parameters"){
                        return v;
                    }
                }),
                {
                    ...(value => {
                        switch (value) {
                            case 'es':
                                return {
                                    locale: Spanish
                                };
                            default:
                                return {};
                        }
                    })(this.lang_value),
                    plugins: [new ConfirmDatePlugin({
                        showAlways: false,
                        confirmIcon: "",
                        ...(value => {
                            switch (value) {
                                case 'es':
                                    return {
                                        confirmText: this.texts.ok
                                    };
                                default:
                                    return {};
                            }
                        })(this.lang_value),
                    })],
                });
            },
            model: {
                get(){
                    let value = this.value;

                    if(typeof this.config.mode !== 'undefined'){
                        if(this.config.mode === 'range'){
                            if(value instanceof Array){
                                return value.join(' a ');
                            }
                        }
                    }

                    if(this.config.mode === 'single'){
                        if(value instanceof Array){
                            value = value[0];
                        }
                    }

                    return value;
                },
                set(val){
                    if(typeof this.config.mode !== 'undefined'){
                        if(this.config.mode === 'range'){
                            if(this.datePicker.fp.selectedDates instanceof Array){
                                this.$emit('input', this.datePicker.fp.selectedDates.map(item => item.toISOString()));

                                return;
                            }
                        }
                    }

                    this.$emit('input', val);
                }
            },
            parameters(){
                return this.config.parameters;
            },
            required(){
                if(typeof this.parameters !== "undefined"){
                    if(typeof this.parameters.required !== "undefined"){
                        return this.parameters.required;
                    }
                }

                return true;
            },
            datePicker(){
                return this.$refs.datepicker;
            }
        },
        watch:{
            'value': function(){
                this.$emit('on-change', this.value);

                this.state.emit = true;
            }
        },
        created(){
            window.addEventListener('scroll', this.handleScroll);

            if(typeof this.config.mode !== 'undefined'){
                if(this.config.mode === 'range'){
                    this.config.defaultDate = this.value;
                }
            }
        },
        methods:{
            emitClose(){
                if(this.state.emit){
                    this.$emit('on-close');

                    this.state.close = false;
                }
            },
            handleScroll(){
                if(this.$route.name == 'step2'){
                    $('.flatpickr-calendar.open').addClass('fixed-class');
                }
            }
        },
        destroyed(){
            window.removeEventListener('scroll', this.handleScroll);
        }
    }
</script>

<style scoped>
    .button-datepicker{
        min-width: 46px;
        display: flex;
        flex: 1;
        justify-content: center;
        align-items: center;
        border: 1px solid #ced4da;
        background-color: #fff;
    }

    .parent-icon{
        cursor:pointer
    }

    .parent-form-group{
        min-width: 308px;
    }
</style>

<style>
    .flatpickr-input.form-control:focus{
        color: #495057;
        background-color: #fff;
        border-color: #80bdff;
        box-shadow: none;
        outline: 0;
    }

    .fixed-class{
        top: 90px!important;
        position: fixed !important;

    } 

    .flatpickr-input.form-control{
        border-radius: .25rem!important;
        border-top-right-radius: 0px!important;
        border-bottom-right-radius: 0px!important;
    }
    .lightTheme
    {
        background: #569ff7;
        margin-top: 10px;
        color: white;
    } 

    @media (max-width: 540px) {
         .flatpickr-calendar:not(.noCalendar).open, .flatpickr-calendar:not(.noCalendar).inline{
         /* position: fixed !important; */

         }
        .flatpickr-calendar:not(.noCalendar).open{
          /*   top: 0 !important;
            left: 0 !important;
            right: 0 !important;
            bottom: 0 !important;
            padding: 10px !important;
            min-height: 100% !important; */
        }
        .flatpickr-rContainer, .flatpickr-calendar:not(.noCalendar).open, .dayContainer, .flatpickr-days{
           /*  width: 100% !important; */
        }
        .rangeMode .flatpickr-day{
            margin-top: 15px !important;
        }
        .dayContainer{
            min-width: 0px !important ;
            max-width: 100% !important ;
        }
        .flatpickr-months .flatpickr-prev-month, .flatpickr-months .flatpickr-next-month{
            top: 6px !important;
        }
    }
</style>