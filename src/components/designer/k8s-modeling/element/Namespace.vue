<template>
    <div>
        <group-element
                selectable
                :movable="!value.editing"
                :resizable="!value.editing"
                :deletable="editMode"
                :id.sync="value.elementView.id"
                :x.sync="value.elementView.x"
                :y.sync="value.elementView.y"
                :label.sync="name"
                :width.sync="value.elementView.width"
                :height.sync="value.elementView.height"
                v-on:selectShape="selectedActivity"
                v-on:deSelectShape="deSelectedActivity"
                v-on:dblclick="showProperty"
                v-on:addToGroup="onAddToGroup"
                v-on:removeShape="onRemoveShape(value)"
                :_style="{stroke:'black',
                'vertical-align': 'top',
                                'font-weight': 'bold',
                                'font-size': '16',
                        }">
            <sub-elements>
                <!--title-->
                <image-element
                        :image="imgSrc"
                        :sub-top="5"
                        :sub-left="5"
                        :sub-width="25"
                        :sub-height="25"
                >

                </image-element>
            </sub-elements>
        </group-element>

        <property-panel
                v-if="openPanel"
                v-model="value"
                :img="imgSrc">
        </property-panel>

    </div>
</template>

<script>
    import PropertyPanel from './NamespacePropertyPanel'
    import Element from '../Kube-Element'
    import GroupElement from "../../../opengraph/shape/GroupElement";
    import ImageElement from "../../../opengraph/shape/ImageElement";

    var changeCase = require('change-case');
    var pluralize = require('pluralize');


    export default {
        components: {ImageElement, GroupElement,PropertyPanel},
        mixins: [Element],
        name: 'namespace',
        props: {},
        computed: {
            defaultStyle() {
                return {}
            },
            className() {
                return 'Namespace'
            },
            imgSrc() {
                return `${ window.location.protocol + "//" + window.location.host}/static/image/symbol/kubernetes/ns.svg`
            },
            createNew(elementId, x, y, width, height) {
                return {
                    _type: this.className(),
                    name: '',
                    elementView: {
                        '_type': this.className(),
                        'id': elementId,
                        'x': x,
                        'y': y,
                        'width': width,
                        'height': height,
                        'style': JSON.stringify({}),
                        'angle': 0,
                    },
                    object: {
                        "apiVersion": "v1",
                        "kind": "Namespace",
                        "metadata": {
                            "name": ""
                        },
                        "spec": {
                            "finalizers": [
                                "kubernetes"
                            ]
                        }
                    },
                    innerElement: []
                }
            },
            name() {
                try {
                    return this.value.object.metadata.name;
                } catch(e) {
                    return "Untitled";
                }
            },
        },
        data: function () {
            return {
            };
        },
        created: function () {
        },
        watch: {
            name: _.debounce(function(appName) {
                var me= this
                this.value.name = appName
                var obj = {
                    state: 'changeName',
                    element: appName
                }
                this.value.innerElement.forEach(function(element)  {
                    me.$EventBus.$emit(element, obj)
                })
            }, 300),
        },
        mounted: function () {
            var me = this
            me.$EventBus.$on(`${me.value.elementView.id}`, function(message) {
                console.log(message)
                me.value.innerElement.splice(me.value.innerElement.indexOf(message.element), 1);
            })
        },
        methods: {
            deleteNameSpace() {
                var me = this
                return new Promise(function (resolve) {
                    me.value.aggregates.forEach(function (aggregate) {
                        console.log(aggregate.elementView.id)
                        var obj = {
                            state: "deleteBounded",
                            element: me.value
                        }
                        me.$EventBus.$emit(aggregate.elementView.id, obj)
                    })
                    me.deleteInNameSpace(me.value.name)
                    me.$EventBus.$off(`${me.value.elementView.id}`);
                    me.$emit('update:value', null);
                    // me.$EventBus.$emit('storage')
                    resolve()
                })
            },
            deleteInNameSpace(boundedName) {
                var me = this
                var designer = this.getComponent('kube-modeling-designer')

                designer.value.definition.forEach(function (item, idx) {
                    if (item != null) {
                        if (item.boundedContext && item.boundedContext.name == boundedName) {
                            // console.log("boundedItem Id : ", item.elementView.id)
                            me.$EventBus.$off(`${item.elementView.id}`);
                            designer.value.definition[idx] = null;
                        }
                    }
                })
                // me.$EventBus.$emit('storage')
            }
        }
    }
</script>


<style scoped lang="scss" rel="stylesheet/scss">

</style>
