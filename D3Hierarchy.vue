<template>
    <div class="d3-tree-container" style="margin: 10px;">
        <el-select v-model="selectedSourceAttribute" placeholder="Select Attribute"
            style="width: 300px; margin: 10px; font-size: 20px; z-index: 0;" @change="updateChart">
            <el-option v-for="(attr, index) in uniqueSourceAttributes" :key="index" :value="attr"
                style="font-size: 20px;"></el-option>
        </el-select>

        <svg :width="width" :height="height">
            <g :transform="'translate(' + margin.left + ',' + margin.top + ')'" ref="treeContainer"></g>
        </svg>
    </div>
</template>


<script>
import * as d3 from 'd3';
import data from './assets/json/data.json';

export default {
    name: 'D3TreeDiagram',
    data() {
        return {
            width: window.innerWidth - 200,
            height: window.innerHeight - 200,
            margin: { top: 50, right: 100, bottom: 20, left: 150 },
            selectedSourceAttribute: '',
            uniqueSourceAttributes: []
        };
    },
    mounted() {
        this.extractUniqueAttributes();
    },
    methods: {
        extractUniqueAttributes() {
            const uniqueAttributes = new Set();
            data.forEach(item => {
                item.sourceAttribute.split(',').forEach(attr => uniqueAttributes.add(attr.trim()));
            });
            this.uniqueSourceAttributes = [...uniqueAttributes];
        },
        getDataFlowComp() {
            const dataFlows = [];
            data.forEach(obj => {
                if (obj.sourceAttribute.split(',').includes(this.selectedSourceAttribute)) {
                    const flowComps = [obj.sourceComponent, ...obj.destinationComponent];
                    dataFlows.push(flowComps);
                }
            });
            return dataFlows;
        },
        updateChart() {
            const dataFlows = this.getDataFlowComp();
            const hierarchyData = this.transformData(dataFlows);
            this.renderTree(hierarchyData);
        },
        transformData(dataFlows) {
            const root = { name: this.selectedSourceAttribute, children: [] };
            const nodesMap = {};
		
            dataFlows.forEach(flow => {
                let currentNode = root;
                flow.forEach(comp => { 
                    const newNode = { name: comp, children: [] };
                    nodesMap[comp] = newNode;
                    currentNode.children.push(newNode);
                    currentNode = nodesMap[comp];
                });
            });

            return root;
        },
        renderTree(data) {
            const svg = d3.select(this.$refs.treeContainer);
            svg.selectAll("*").remove();

            const treeLayout = d3.tree().size([this.height - this.margin.top - this.margin.bottom, this.width - this.margin.right - this.margin.left]);
            const root = d3.hierarchy(data);
            treeLayout(root);

            const nodeSize = this.calculateNodeSize(root.descendants().length);

            const link = svg.selectAll(".link")
                .data(root.links())
                .enter().append("path")
                .attr("class", "link")
                .attr("d", d3.linkHorizontal()
                    .x(d => d.y)
                    .y(d => d.x)
                )
                .attr("fill", "none")
                .attr("stroke", "#ccc")
                .attr("stroke-width", 5)
                .on("mouseover", this.highlightPath)
                .on("mouseout", this.resetPathHighlight);

            const node = svg.selectAll(".node")
                .data(root.descendants())
                .enter().append("g")
                .attr("class", "node")
                .attr("transform", d => `translate(${d.y},${d.x})`);

            node.append("circle")
                .attr("r", nodeSize)
                .attr("fill", "#3182bd");

            node.append("text")
                .attr("dy", "-1em")
                .attr("x", d => d.children ? -12 : 12)
                .style("text-anchor", d => d.children ? "end" : "start")
                .text(d => d.data.name);
                
           console.log(svg.selectAll(".link"));     
        },
        calculateNodeSize(numNodes) {
            const baseSize = 10;
            const maxSize = 30;
            const minSize = 5;
            const size = baseSize + (maxSize - baseSize) / Math.sqrt(numNodes);
            return Math.max(minSize, Math.min(maxSize, size));
        },
        highlightPath(event, d) {
            d3.selectAll(".highlight").classed("highlight", false);

            let current = d.target;
            while (current) {
                d3.select(current).classed("highlight", true);
                current = current.parent;
            }
        },
        resetPathHighlight(event, d) {
            d3.selectAll(".highlight").classed("highlight", false);
        }
    }
};
</script>



<style scoped>
.d3-tree-container {
    margin-bottom: 20px;
}

.link.highlight {
    stroke: orange;
    stroke-width: 4;
}

.node.highlight circle {
    stroke: orange;
    stroke-width: 4;
}
</style>

