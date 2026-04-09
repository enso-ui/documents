<template>
    <card collapsible
        :collapsed="collapsed">
        <card-header>
            <template #title>
                <span class="icon is-small mr-1">
                    <fa :icon="icon"/>
                </span>
                {{ displayTitle }}
            </template>
            <template #controls>
                <card-refresh @refresh="fetch"/>
                <card-badge :label="count"/>
                <card-collapse/>
            </template>
        </card-header>
        <card-content class="p-0">
            <documents v-bind="$attrs"
                :id="id"
                :type="type"
                :query="query"
                @update="count = $refs.documents.count; $refs.card.resize()"
                ref="documents"/>
        </card-content>
    </card>
</template>

<script>
import { FontAwesomeIcon as Fa } from '@fortawesome/vue-fontawesome';
import { faCopy } from '@fortawesome/free-solid-svg-icons';
import {
    Card, CardHeader, CardRefresh, CardCollapse, CardBadge, CardContent,
} from '@enso-ui/card/bulma';
import Documents from './Documents.vue';
import { useStore } from '../utils/pinia';

export default {
    name: 'DocumentsCard',

    components: {
        Fa,
        Card,
        CardHeader,
        CardRefresh,
        CardCollapse,
        CardBadge,
        CardContent,
        Documents,
    },

    inject: ['i18n', 'route'],

    props: {
        icon: {
            type: [String, Array, Object],
            default: () => faCopy,
        },
        collapsed: {
            type: Boolean,
            default: false,
        },
        id: {
            type: [String, Number],
            required: true,
        },
        type: {
            type: String,
            required: true,
        },
        title: {
            type: String,
            default: '',
        },
    },

    data: () => ({
        query: '',
        count: 0,
    }),

    computed: {
        isMobile() {
            return useStore('layout').isMobile;
        },
        isEmpty() {
            return this.count === 0;
        },
        uploadLink() {
            return this.route('core.documents.store');
        },
        displayTitle() {
            return !this.isMobile
                ? this.title || this.i18n('Documents')
                : null;
        },
    },

    watch: {
        count() {
            this.$refs.card.resize();
        },
    },
};
</script>
