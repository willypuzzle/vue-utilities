# My Utilities for Vue.js 

I use these in some of my applications.

To import:

```javascript

import WpVueUtilities from 'wp-vue-utilities';

/*In order to use AjaxForm*/
Vue.component('ajax-form', WpVueUtilities.AjaxForm);

```

To Use AjaxForm, an Example:

```
    <template>
      <ajax-form
            :inputs="inputs"
            :buttons="buttons"
            :config="config"
      >
    
      </ajax-form>
    </template>
    
    <script>
        export default {
            data(){
                return {
                    inputs: [
                        {
                            type: 'text',
                            name: 'address',
                            label: {
                                name: 'Address'
                            },
                            required: true
                        }
                    ],
                    buttons: {
                        cancel: {
                            label: 'Cancel',
    
                        },
                        confirm: {
                            label: 'Confirm'
                        }
                    },
                    config: {
                        inputErrorClass: 'input-error',
                        responseOkRoute: 'list-sites',
                        postUrl: 'http://localhost/sites/create/'
                    }
                };
            }
        }
    </script>
```

There are more others options, check the code!