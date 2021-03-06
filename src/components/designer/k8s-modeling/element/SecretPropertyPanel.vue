<template>
    <v-layout wrap>
        <v-navigation-drawer absolute permanent right v-bind:style="{width: 800}">
            <!--  상단 이미지 및 선택 타이틀 이름-->
            <v-list class="pa-1">
                <v-list-item>
                    <v-list-item-avatar>
                        <img :src="img">
                    </v-list-item-avatar>
                    <v-list-item-title class="headline">
                        {{ value._type }}
                    </v-list-item-title>
                    <v-tooltip left>
                        <template v-slot:activator="{ on }">
                            <v-btn icon v-on="on" @click="desDocOpen()">
                                <v-icon color="grey lighten-1">mdi-information</v-icon>
                            </v-btn>
                        </template>
                        <span>{{ descriptionText }}</span>
                    </v-tooltip>
                </v-list-item>
            </v-list>

            <v-list class="pt-0" dense flat>
                <v-layout wrap>
                    <v-flex shrink style="width: 230px;">
                        <v-card flat>
                            <v-card-text>
                                <v-text-field
                                    label="Name"
                                    v-model="value.object.metadata.name"
                                ></v-text-field>
                                <v-text-field
                                    label="Type"
                                    v-model="value.object.type"
                                ></v-text-field>
                                <v-label>Data</v-label>
                                <v-row>
                                    <v-col cols="5" class="py-0">
                                        <v-text-field
                                                label="Key"
                                                v-model="dataKey"
                                        ></v-text-field>
                                    </v-col>
                                    <v-col class="py-0">
                                        <v-text-field
                                                label="Value"
                                                v-model="dataValue"
                                                hint="base64 encoding"
                                                v-on:keyup.enter="addData(dataKey, encodingDataValue)"
                                        ></v-text-field>
                                    </v-col>
                                </v-row>
                                <v-row justify="end">
                                    <v-btn 
                                            class="mx-5"
                                            color="primary"
                                            rounded dark
                                            @click="addData(dataKey, encodingDataValue)"
                                    >Add Data</v-btn>
                                </v-row>
                            </v-card-text>
                        </v-card>
                    </v-flex>
                    <v-flex>
                        <yaml-editor
                            v-model="value.object">
                        </yaml-editor>
                    </v-flex>
                </v-layout>
            </v-list>

        </v-navigation-drawer>
    </v-layout>

</template>


<script>
    import yaml from "js-yaml";
    var Base64 = require('js-base64').Base64;

    import YamlEditor from "../KubeYamlEditor";
    import NumberField from "./NumberField";

    export default {
        name: 'property-panel',
        props: {
            value: Object,
            img: String,
        },
        components: {
            "yaml-editor": YamlEditor,
            "number-field": NumberField,
        },
        computed: {
            descriptionText() {
                return 'Secret'
            },
            encodingDataValue() {
                return Base64.encode(this.dataValue)
            }
        },
        data: function () {
            return {
                dataObj: JSON.parse(JSON.stringify(this.value.object.data)),
                dataKey: "",
                dataValue: "",
            }
        },
        watch: {
        },
        methods: {
            addData(key, value) {
                var me = this
                if(key != "" && value != "") {
                    me.dataObj[key] = value
                    me.value.object.data = me.dataObj
                }
                me.dataKey = ""
                me.dataValue = ""
            },
            desDocOpen() {
                window.open('https://kubernetes.io/docs/concepts/configuration/secret/')
            },
        }
    }
</script>


<style lang="scss" rel="stylesheet/scss">
    .v-icon.outlined {
        border: 1px solid currentColor;
        border-radius: 0%;
    }

    .md-sidenav .md-sidenav-content {
        width: 400px;
    }

    .md-sidenav.md-right .md-sidenav-content {
        width: 600px;
    }

    .flip-list-move {
        transition: transform 0.5s;
    }

    .no-move {
        transition: transform 0s;
    }

    .ghost {
        opacity: 0.5;
        background: #c8ebfb;
    }

    .list-group {
        min-height: 20px;
    }

    .list-group-item {
        cursor: move;
    }

    .list-group-item i {
        cursor: pointer;
    }
</style>
