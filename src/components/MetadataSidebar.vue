<template>
<div class="table-responsive metadata">
    <table class="table">
        <tbody>
            <tr>
                <td class="group" colspan="2">
                <h4>Metadata</h4>
                </td>
            </tr>
            <tr v-if="stacVersion">
                <td class="title">STAC Version</td>
                <td>{{ stacVersion }}</td>
            </tr>
            <tr v-if="keywords">
                <td class="title">Keywords</td>
                <td>{{ keywords }}</td>
            </tr>
            <tr v-if="collectionLink">
            <td class="title">Collection</td>
                <td>
                    <router-link :to="linkToCollection">
                    {{ collection.title || "Untitled" }}
                    </router-link>
                </td>
            </tr>
            <tr v-if="license">
                <td class="title">License</td>
                <!-- eslint-disable-next-line vue/no-v-html -->
                <td v-html="license" />
            </tr>
            <tr v-if="temporalExtentReadable.length > 0">
                <td class="title">Temporal Extent</td>
                <td>{{ temporalExtentReadable }}</td>
            </tr>
            <template v-for="(props, ext) in propertyList">
                <tr v-if="ext" :key="ext">
                <td class="group" colspan="2">
                    <h4>{{ ext }}</h4>
                </td>
                </tr>
                <tr v-for="prop in props" :key="prop.key">
                <td class="title">
                    <!-- eslint-disable-next-line vue/no-v-html -->
                    <span :title="prop.key" v-html="prop.label" />
                </td>
                <!-- eslint-disable-next-line vue/no-v-html -->
                <td v-html="prop.value" />
                </tr>
            </template>
            <template v-if="Array.isArray(providers) && providers.length > 0">
                <tr>
                <td colspan="2" class="group">
                    <h4>
                    <template v-if="providers.length === 1">
                        Provider
                    </template>
                    <template v-if="providers.length !== 1">
                        Providers
                    </template>
                    </h4>
                </td>
                </tr>
                <template v-for="(provider, index) in providers">
                <tr :key="provider.url + index">
                    <td colspan="2" class="provider">
                    <a :href="provider.url">{{ provider.name }}</a>
                    <em v-if="provider.roles"
                    >({{(Array.isArray(provider.roles) ? provider.roles : []).join(", ") }})</em
                    >
                    <!-- eslint-disable-next-line vue/no-v-html vue/max-attributes-per-line -->
                    <div
                        v-if="provider.description"
                        class="description"
                        v-html="provider.description"
                    />
                    </td>
                </tr>
                </template>
            </template>
            <template v-if="hasSummary">
                <tr>
                <td colspan="2" class="group summary">
                    <h4>Item Summary</h4>
                </td>
                </tr>
                <template v-for="(props, ext) in summariesList">
                <tr v-if="ext" :key="ext">
                    <td class="group" colspan="2">
                    <h4>{{ ext }}</h4>
                    </td>
                </tr>
                <tr v-for="prop in props" :key="prop.key">
                    <td class="title summary-title" >
                        <!-- eslint-disable-next-line vue/no-v-html -->
                        <span :title="prop.key" v-html="prop.label" />
                    </td>
                    <!-- eslint-disable-next-line vue/no-v-html -->
                    <td v-html="prop.value" />
                </tr>
                </template>
            </template>
        </tbody>
    </table>
</div>
</template>

<script>
import isEmpty from "lodash.isempty";

import { getPropertyDefinitions } from "../properties.js";

const propertyDefinitions = getPropertyDefinitions(),
  propertyMap = propertyDefinitions.properties,
  groupMap = propertyDefinitions.groups;

const constructPropList = (props, summaries = false, skip = () => false) => {
    return Object.entries(props || [])
                .filter(([, v]) => Number.isFinite(v) || !isEmpty(v) || (summaries && Array.isArray(v) && v.length == 0)) // last part: Skip empty summaries
                .filter(([k]) => !skip(k))
                .sort(([a], [b]) => a - b)
                .map(([key, value]) => ({
                    key,
                    label: propertyDefinitions.formatPropertyLabel(key),
                    value: summaries ? propertyDefinitions.formatSummaryValues(key, value) : propertyDefinitions.formatPropertyValue(key, value)
                }))
                .reduce((acc, prop) => {
                    let ext = "";
                    if (prop.key.includes(":")) {
                        const prefix = prop.key.split(":")[0];
                        ext = groupMap[prefix] || prefix;
                    }

                    acc[ext] = acc[ext] || [];
                    acc[ext].push(prop);

                    return acc;
                }, {});
};

export default {
    name: "MetadataSidebar",
    props: [
        "properties",
        "summaries",
        "stacVersion",
        "keywords",
        "collection", // Item-specific
        "collectionLink", // Item-specific
        "license",
        "temporalExtent", // Collection-specific
        "providers",
        "slugify",
    ],
    computed: {
        linkToCollection() {
            if (this.collectionLink.href != null) {
                return `/collection/${this.slugify(this.collectionLink.href)}`;
            }

            return null;
        },
        hasSummary() {
            return this.summaries && typeof this.summaries === 'object' && Object.keys(this.summaries).length > 0;
        },
        summariesList() {
            const skip = key => propertyMap[key] && propertyMap[key].skip;
            return constructPropList(this.summaries, true, skip);
        },
        propertyList() {
            const skip = key => propertyMap[key] && propertyMap[key].skip;
            return constructPropList(this.properties, false, skip);
        },
        temporalExtentReadable() {
            if (!Array.isArray(this.temporalExtent)) {
                return '';
            }
            return this.temporalExtent
                .map(interval => {
                    return [
                        interval[0] ? new Date(interval[0]).toLocaleString() : "beginning of time",
                        interval[1] ? new Date(interval[1]).toLocaleString() : "now"
                    ].join(" - ")
                }).join(', ');
        }
    }
};

</script>

<style scoped lang="css">
.summary-title {
    font-weight: bold;
    width: 40%;
}
</style>
<style>
.table td.group.summary {
  background-color: #555;
}

.table td.group.summary h4 {
  font-weight: bold;
  color: #ddd;
}
.metadata-object .metadata-object {
    margin-left: 1em;
}
</style>
