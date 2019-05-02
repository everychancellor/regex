<template>
	<div class="example-input-component-wrapper">
		<div class="wrapper" @click="$refs.configModal.open()">
            <editor :template="input.data"
                    :schema="defaultStep.data"
                    :step="defaultStep"
                    :steps="[defaultStep]"
                    :readonly="true">
            </editor>
        </div>

        <or-modal ref="configModal" 
                  title="Configure component"
                  class="input-message-settings"
				  :remove-close-button="true"
                  :contain-focus="false">

			<or-switch v-model="input.data.hasBooInput">Use single exit</or-switch>

            <div slot="footer">
                <or-button color="primary"
                           type="secondary"
                           @click.prevent="$refs.configModal.close()">
                    Close
                </or-button>
            </div>
        </or-modal>
	</div>
</template>

<script>
    import _ from 'lodash';
    import base from '@default/src/inputs/_design_base.vue';
    import editor from '../editor/editor.vue';

    export default {
        extends    : base,

        components : {
            editor
        },

        defaultValue () {
            return _.cloneDeep(data);
        }
    };

    export const label = 'Regexes xtrnl component';

	export const data = {
		hasBooInput : false,
		booInput	: "``"	
	};

    export const meta = {
        name    : 'regexes-external-component',
        type    : 'onereach-studio-form-input',
        version : '1.0.0'
    };
</script>

<style lang="scss" scoped>
	.example-input-component-wrapper {
		width: 100%;
	}
</style>
