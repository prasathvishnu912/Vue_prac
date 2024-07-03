<template>
    <div>
        <div id="main" style="width: 100vw; height: 100vh;"></div>
    </div>
</template>


<script>
import jsonData from '../../../src/data.json';

export default {
    name: 'EChartsComponent',
    data() {
        return {
            myChart: null,
            option: null
        };
    },
    mounted() {
        this.initChart();
        window.addEventListener('resize', this.resizeChart);
    },
    methods: {
        initChart() {
            this.myChart = echarts.init(document.getElementById('main'), null, {
                renderer: 'svg'
            });
     
            this.myChart.showLoading();

            let nodes = {};
            let links = [];

            jsonData.forEach((obj) => {
                if (!nodes[obj.sourceComponent]) {
                    nodes[obj.sourceComponent] = { name: obj.sourceComponent, symbolSize: 150 };
                }

                obj.destinationComponent.forEach((dest) => {
                    if (!nodes[dest]) {
                        nodes[dest] = { name: dest, symbolSize: 150 };
                    }
                    console.log(obj.sourceComponent, dest)
                    links.push({
                        source: obj.sourceComponent,
                        target: dest,
                        sourceAttribute: obj.sourceAttribute,
                        destinationAttribute: obj.destinationAttribute,
                        url: obj.url
                    });
                });
            });

            let nodesArray = Object.keys(nodes).map((key) => nodes[key]);

            this.option = {
                tooltip: {
                    triggerOn: 'mousemove',
                    enterable: true,
                    
                    formatter: function (params) {
                        if (params.dataType === 'edge') {
                            let data = params.data;
                            return `
                                <b>${data.source} ➜ ${data.target}</b><br/>
                                <b>Source Attribute:</b> ${data.sourceAttribute}<br/>
                                <b>Destination Attribute:</b> ${data.destinationAttribute}<br/>
                                <b>URL:</b> ${data.url.join('<br/>')}
                            `;
                        }
                        return params.name;
                    }
                },
                series: [
                    {
                        type: 'graph',
                        layout: 'circular',
                        data: nodesArray,
                        links: links,
                        emphasis: {
                            lineStyle: {
                                width: 10,
                                opacity: 0.7,
                                //arrow: 'to' 
                            },
                            label: {
                                show: true,
                                position: 'inside',
                                formatter: '{b}'
                            }
                        },
                        roam: true,
                        label: {
                            show: true,
                            position: 'inside',
                            formatter: '{b}'
                        },
                        lineStyle: {
                            width: 3,
                            curveness: 0.3,
                            color: '#6d809b',
                            arrow: 'to'
                        },
                        edgeSymbol: ['none', 'arrow'],
                        edgeSymbolSize: [4, 10],
                        edgeEffect: {
                            show: true,
                            period: 6,
                            trailLength: 0.7,
                            color: 'rgba(255, 255, 255, 0.6)',
                            symbolSize: 3
                        },
                        animation: true,
                        animationDurationUpdate: 1500,
                        animationEasingUpdate: 'quinticInOut'
                    }
                ]
            };

            this.myChart.setOption(this.option);
            this.myChart.hideLoading();
        },
        resizeChart() {
            if (this.myChart) {
                this.myChart.resize();
            }
        }
    },
    beforeDestroy() {
        window.removeEventListener('resize', this.resizeChart);
        if (this.myChart) {
            this.myChart.dispose();
        }
    }
};
</script>



<style></style>
