<template>
    <div class="documents-wrapper">
        <slot :id="id"
            name="controls"
            :type="type"
            :uploadLink="uploadLink"
            :fetch="fetch"
            :internal-query="internalQuery"
            v-if="!disableControls">
            <div class="field is-grouped">
                <p class="control">
                    <enso-uploader is-small
                        :compact="compact"
                        :params="params"
                        :url="uploadLink"
                        :label="i18n('Upload')"
                        :file-size-limit="fileSizeLimit"
                        multiple
                        v-if="!disableUpload && uploadLink"
                        @upload-successful="fetch();"/>
                </p>
                <p class="control has-icons-left has-icons-right is-expanded">
                    <input v-model="internalQuery"
                        class="input is-small"
                        type="text"
                        :placeholder="i18n('Filter')">
                    <span class="icon is-small is-left">
                        <fa :icon="faSearch"/>
                    </span>
                    <span v-if="internalQuery"
                        class="icon is-small is-right clear-button is-clickable"
                        @click="internalQuery = ''">
                        <a class="delete is-small"/>
                    </span>
                </p>
                <p class="control">
                    <a class="button is-small"
                        @click="fetch()">
                        <span v-if="!compact">
                            {{ i18n('Reload') }}
                        </span>
                        <span class="icon">
                            <fa :icon="faArrowsRotate"/>
                        </span>
                    </a>
                </p>
            </div>
        </slot>
        <item v-for="(document, index) in documents"
            :key="document.id"
            :file="document.file"
            @delete="destroy(index)"/>
    </div>
</template>

<script>
import { FontAwesomeIcon as Fa } from '@fortawesome/vue-fontawesome';
import { faArrowsRotate, faSearch } from '@fortawesome/free-solid-svg-icons';
import { EnsoUploader } from '@enso-ui/uploader/bulma';
import Item from '@enso-ui/files/src/bulma/pages/files/components/Item.vue';
import debounce from 'lodash/debounce';

export default {
    name: 'Documents',

    components: {
        Fa, Item, EnsoUploader,
    },

    inject: ['errorHandler', 'i18n', 'http', 'route', 'canAccess'],

    props: {
        id: {
            type: [String, Number],
            required: true,
        },
        type: {
            type: String,
            required: true,
        },
        query: {
            type: String,
            default: '',
        },
        compact: {
            type: Boolean,
            default: false,
        },
        disableControls: {
            type: Boolean,
            default: false,
        },
        disableUpload: {
            type: Boolean,
            default: false,
        },
        fileSizeLimit: {
            default: 20 * 1024 * 1024,
            type: Number,
        },
    },

    emits: ['update'],

    data: () => ({
        documents: [],
        faArrowsRotate,
        faSearch,
        loading: false,
        internalQuery: '',
    }),

    computed: {
        params() {
            return {
                documentable_type: this.type,
                documentable_id: this.id,
                query: this.internalQuery,
            };
        },
        count() {
            return this.documents.length;
        },
        uploadLink() {
            return this.canAccess('core.documents.store')
                ? this.route('core.documents.store')
                : null;
        },
    },

    watch: {
        query() {
            this.internalQuery = this.query;
        },
        internalQuery() {
            this.fetch();
        },
    },

    created() {
        this.fetch();
        this.fetch = debounce(this.fetch, 300);
    },

    methods: {
        fetch() {
            this.loading = true;

            this.http.get(this.route('core.documents.index'), {
                params: this.params,
            }).then(({ data }) => {
                this.documents = data;
                this.loading = false;
                this.$emit('update');
            }).catch(this.errorHandler);
        },
        destroy(index) {
            this.loading = true;

            this.http.delete(this.route(
                'core.documents.destroy',
                this.documents[index].id, false,
            )).then(() => {
                this.loading = false;
                this.documents.splice(index, 1);
                this.$emit('update');
            }).catch(this.errorHandler);
        },
    },
};
</script>
