<template>
    <div class="d3-tree-container" style="margin: 10px;">
        <el-select v-model="selectedSourceAttribute" placeholder="Select Attribute"
            style="width: 250px; margin: 10px; font-size: 20px; z-index: 0;" @change="getDataFlow">
            <el-option v-for="(attr, index) in uniqueSourceAttributes" :key="index" :value="attr"
                style="font-size: 20px;">{{ attr }}</el-option>
        </el-select>
        <svg ref="treeSvg" :width="width" :height="height"></svg>
    </div>
</template>

<script>
import * as d3 from "d3";
import data from "./assets/json/data.json";

export default {
    name: "D3TreeDiagram",
    data() {
        return {
            width: window.innerWidth - 200,
            height: window.innerHeight - 200,
            margin: { top: 20, right: 100, bottom: 20, left: 100 },
            nodes: [],
            links: [],
            selectedSourceAttribute: "",
            uniqueSourceAttributes: [],
            simulation: null,
        };
    },
    mounted() {
        this.extractUniqueAttributes();
    },
    methods: {
        extractUniqueAttributes() {
            const uniqueAttributes = new Set();
            data.forEach((item) => {
                item.sourceAttribute
                    .split(",")
                    .forEach((attr) => uniqueAttributes.add(attr.trim()));
            });
            this.uniqueSourceAttributes = [...uniqueAttributes];
        },
        getDataFlow() {
            this.nodes = [];
            this.links = [];

            const uniqueNodes = new Set();

            data.forEach((obj) => {
                if (obj.sourceAttribute.includes(this.selectedSourceAttribute)) {
                    uniqueNodes.add(obj.sourceComponent);

                    obj.destinationComponent.forEach(dest => uniqueNodes.add(dest));

                    obj.destinationComponent.forEach(dest => {
                        this.links.push({ source: obj.sourceComponent, target: dest });
                    });
                }
            });

            this.nodes = Array.from(uniqueNodes).map(node => ({ id: node }));

            this.updateDiagram();
        },

        updateDiagram() {
            const svg = d3.select(this.$refs.treeSvg);

            svg.selectAll("*").remove();

            this.simulation = d3.forceSimulation(this.nodes)
                .force("link", d3.forceLink(this.links).id(d => d.id).distance(100))
                .force("charge", d3.forceManyBody().strength(-200))
                .force("center", d3.forceCenter(this.width / 2, this.height / 2));

            const defs = svg.append("defs");
            const filter = defs.append("filter")
                .attr("id", "drop-shadow")
                .attr("height", "130%");

            filter.append("feGaussianBlur")
                .attr("in", "SourceAlpha")
                .attr("stdDeviation", 5)
                .attr("result", "blur");

            filter.append("feOffset")
                .attr("in", "blur")
                .attr("dx", 5)
                .attr("dy", 5)
                .attr("result", "offsetBlur");

            const feMerge = filter.append("feMerge");

            feMerge.append("feMergeNode")
                .attr("in", "offsetBlur");
            feMerge.append("feMergeNode")
                .attr("in", "SourceGraphic");

            const gradient = defs.append("radialGradient")
                .attr("id", "glossyGradient")
                .attr("cx", "50%")
                .attr("cy", "50%")
                .attr("r", "50%")
                .attr("fx", "50%")
                .attr("fy", "50%");

            gradient.append("stop")
                .attr("offset", "0%")
                .style("stop-color", "#6baed6")
                .style("stop-opacity", 1);

            gradient.append("stop")
                .attr("offset", "100%")
                .style("stop-color", "#08519c")
                .style("stop-opacity", 1);

            const link = svg.append("g")
                .attr("class", "links")
                .selectAll("line")
                .data(this.links)
                .enter().append("line")
                .attr("stroke", "black")
                .attr("stroke-width", 1);

            const node = svg.append("g")
                .attr("class", "nodes")
                .selectAll(".node")
                .data(this.nodes)
                .enter().append("g")
                .attr("class", "node");

            node.append("circle")
                .attr("r", 40)
                .attr("fill", "url(#glossyGradient)")
                .style("filter", "url(#drop-shadow)");

            node.append("text")
                .attr("dy", "-3em")
                .attr("text-anchor", "middle")
                .text(d => d.id);

            node.call(d3.drag()
                .on("start", this.dragStarted)
                .on("drag", this.dragged)
                .on("end", this.dragEnded));

            this.simulation.on("tick", () => {
                link
                    .attr("x1", d => d.source.x)
                    .attr("y1", d => d.source.y)
                    .attr("x2", d => d.target.x)
                    .attr("y2", d => d.target.y);

                node.attr("transform", d => `translate(${d.x},${d.y})`);
            });
        },

        // dragStarted(event, d) {
        //     if (!event.active) this.simulation.alphaTarget(0.3).restart();
        //     d.fx = d.x;
        //     d.fy = d.y;
        // },
        // dragged(event, d) {
        //     d.fx = event.x;
        //     d.fy = event.y;
        // },
        // dragEnded(event, d) {
        //     if (!event.active) this.simulation.alphaTarget(0);
        //     d.fx = null;
        //     d.fy = null;
        // },
    },
};
</script>

<style scoped>
/* Add your custom styles here */
</style>
