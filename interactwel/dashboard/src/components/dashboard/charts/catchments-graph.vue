<template>
    <div id="graph" class="card">
        <div class="card-header">
            <span v-on:click="dismiss" class="close"><font-awesome-icon icon="times-circle"/> Close</span>
            <div class="row">
                <div class="col-8"><strong>Sub-basins</strong></div>
                <div class="col-4"><div class="form-group">
                    <select class="form-control form-control-sm" v-model="selected" @change="changeBasin">
                        <option v-for="option in options" v-bind:value="option.value">{{ option.text }}</option>
                    </select>
                </div></div>
            </div>
        </div>
        <div class="card-body no-padding">
            <b-tabs card>
                <b-tab title="Irrigation" active>
                    <div class="card-body">
                        <chart-barv :chart-data="datacollection" :options="optionsirr" :width="5"
                                    :height="3"></chart-barv>
                    </div>
                </b-tab>
                <b-tab title="Water Rights">
                    <div class="card-body">
                        <!--<n-fertilizer-graph></n-fertilizer-graph>-->
                        <img class="img-fluid" src="../../../assets/graph-placeholder2.png"/>
                    </div>
                </b-tab>
                <b-tab title="Crop yield">
                    <div class="card-body">
                    <img class="img-fluid" src="../../../assets/graph-placeholder2.png"/>
                    </div>
                </b-tab>
                <b-tab title="Groundwater Recharge">
                    <div class="card-body">
                        <img class="img-fluid" src="../../../assets/graph-placeholder2.png"/>
                    </div>
                </b-tab>
                <b-tab title="N fertilizer">
                    <div class="card-body">
                        <img class="img-fluid" src="../../../assets/graph-placeholder2.png"/>
                        <!--<crops-yield-graph></crops-yield-graph>-->
                    </div>
                </b-tab>
            </b-tabs>
        </div>
    </div>
</template>


<script>
    import JSONData from "../../../assets/result_action_plans.json";
    import IrrigationGraph from './irrigation-graph.vue'
    import CropAreaGraph from './crop-area-graph.vue'
    import CropYieldGraph from './crop-yield-graph.vue'
    import NFertilizerGraph from './n-fertilizer-graph.vue'
    import PFertilizerGraph from './p-fertilizer-graph.vue'
    import EventBus from './../../../event-bus';
    import axios from 'axios';
    import ChartBarV from "../../../chart";

    export default {
        name: 'CatchmentsGraph',

        components: {
            'irridationGraph': IrrigationGraph,
            'cropAreaGraph': CropAreaGraph,
            'cropYieldGraph': CropYieldGraph,
            'nFertilizerGraph': NFertilizerGraph,
            'pFertilizerGraph': PFertilizerGraph,
            'chart-barv': ChartBarV,
        },

        data() {
            return {
                selected: '1',
                selectedBasinID: '1',
                planName: 'Adaptation Plan 1',
                JSONData: null,

                options: [
                    {text: 'Sub-basin: 1', value: '1'},
                    {text: 'Sub-basin: 2', value: '2'},
                    {text: 'Sub-basin: 3', value: '3'},
                    {text: 'Sub-basin: 4', value: '4'},
                    {text: 'Sub-basin: 5', value: '5'}
                ],

                datacollection: null,
                optionsirr: {
                    responsive: true,
                    title: {
                        display: true,
                        text: 'Irrigation - Yearly totals per water source'
                    },
                    tooltips: {
                        mode: 'point',
                        intersect: false,
                    },
                    hover: {
                        mode: 'nearest',
                        intersect: true
                    },
                    scales: {
                        xAxes: [{
                            display: true,
                            stacked: false,
                            scaleLabel: {
                                display: true,
                                labelString: 'Years'
                            }
                        }],
                        yAxes: [{
                            display: true,
                            stacked: false,
                            scaleLabel: {
                                display: true,
                                labelString: 'acre-ft'
                            }
                        }]
                    }
                }
            };
        },
        computed: {},

        mounted() {
            let $this = this;
            EventBus.$on('CLICK_ITEM_SIDEBAR', function (planName) {
                $this.planName = planName;
                $this.buildDataCollection($this.JSONData, $this.planName, $this.selectedBasinID)
            });

        },

        created() {
            axios.get("/static/BASIN_Irrigation_basins_data.json").then(response => {
                this.JSONData = response.data;
                this.buildDataCollection(this.JSONData, this.planName, this.selectedBasinID);
            })
        },

        methods: {
            dismiss() {
                EventBus.$emit('CLOSE');
            },

            buildDataCollection(data, adaptationPlan, basinID) {
                this.datacollection = {};
                this.datacollection.labels = [];
                for (let legend in data.Legend) {
                    this.datacollection.labels.push(data.Legend[legend]);
                }
                this.datacollection.datasets = [];
                for (let dataIndex in data.Adaptation_plans[adaptationPlan][basinID]["Data"]) {
                    let dataPoint = data.Adaptation_plans[adaptationPlan][basinID]["Data"][dataIndex];
                    let dataset = {};
                    dataset.label = dataPoint.Name;
                    dataset.backgroundColor = this.getRandomColor();
                    dataset.data = [];
                    for (let dataValue in dataPoint.Data) {
                        dataset.data.push(dataPoint.Data[dataValue]);
                    }
                    this.datacollection.datasets.push(dataset);
                }
            },

            getRandomColor() {
                let letters = '0123456789ABCDEF';
                let color = '#';
                for (let i = 0; i < 6; i++) {
                    color += letters[Math.floor(Math.random() * 16)];
                }
                return color;
            },


            changeBasin() {
                this.selectedBasinID = this.selected;
                this.buildDataCollection(this.JSONData, this.planName, this.selectedBasinID)
            },
        },
        //props: ["jsonData"]

    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>

    .chart-container {
        position: absolute;
        top: 30px;
        left: 30px;
        z-index: 1000;
        background-color: #4cae4c;
        height: auto;
        width: 750px;
        max-width: 750px !important;
    }

    .nav-pills .nav-link.active, .nav-pills .show > .nav-link {
        color: #fff;
        background-color: #4cae4c !important;
    }

    .chart-inner {
        position: absolute !important;
        width: 100% !important;
        bottom: 0 !important;
    }

    .filter-options-container {
        text-align: left;
    }

    .filter-options {
        padding: 10px;
        color: #FFFFFF;
        background-color: #EFEFEF;
        margin: 5px !important;
        border-radius: 5px;
    }

    #sidebar .active {
        color: #FFF !important;
        background-color: #4cae4c;
        margin: 5px !important;
        border-radius: 5px;
    }

    .active a {
        color: rgba(255, 255, 255, 0.8);
    }

    .active a:hover {
        color: #fff
    }

    .filter-options input {
        margin-left: 10px !important;
    }

    #graph {
        width: 750px;
        max-width: 750px !important;
    }

    #graph .form-group{
        margin-bottom:0;
    }
</style>
