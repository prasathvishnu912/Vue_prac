<template>
    <div class="d3-tree-container" style="margin: 10px;">
        <el-select v-model="selectedSourceAttribute" placeholder="Select Attribute"
            style="width: 250px; margin: 10px; font-size: 20px; z-index: 0;" @change="updateChart">
            <el-option v-for="(attr, index) in uniqueSourceAttributes" :key="index" :value="attr"
                style="font-size: 20px;"></el-option>
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
            margin: { top: 20, right: 100, bottom: 20, left: 50 },
            nodes: [],
            links: [],
            selectedSourceAttribute: "",
            uniqueSourceAttributes: [],
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
        getDataFlowComp() {
            const dataFlows = [];
            let i=0;
            data.forEach((obj) => {
                
                if (obj.sourceAttribute.split(",").includes(this.selectedSourceAttribute)) {
                    const flowComps = [obj.sourceComponent, ...obj.destinationComponent];
                    dataFlows.push(flowComps);
                    console.log(dataFlows);
                    console.log(i++);
                }
            });
            return dataFlows;
        },

        updateChart() {
            if (this.selectedSourceAttribute) {
                const dataFlows = this.getDataFlowComp();
              
                const componentCount = {};
                dataFlows.forEach((flow) => {
                    flow.forEach((component) => {
                        if (componentCount[component]) {
                            componentCount[component]++;
                        } else {
                            componentCount[component] = 1;
                        }
                    });
                });

                const sortedComponents = Object.keys(componentCount)
                    .map((name) => ({ name, count: componentCount[name] }))
                    .sort((a, b) => b.count - a.count);

                const hierarchyData = this.createHierarchy(sortedComponents);

                const treeLayout = d3.tree().size([this.height - 100, this.width - 200]);

                const treeData = treeLayout(hierarchyData);

                this.nodes = treeData.descendants().map((node) => ({
                    id: node.data.name,
                    name: node.data.name,
                    x: node.y,
                    y: node.x,
                }));

                this.links = treeData.links();
             
                this.updateSvg();
            } else {
                this.nodes = [];
                this.links = [];
                this.updateSvg();
            }
        },


        createHierarchy(sortedComponents) {
            if (sortedComponents.length === 0) return null;

            const createNode = (components, start, end) => {
                if (start > end) return null;

                const mid = Math.floor((start + end) / 2);
                const node = { name: components[mid].name, children: [] };

                const leftChild = createNode(components, start, mid - 1);
                const rightChild = createNode(components, mid + 1, end);

                if (leftChild) node.children.push(leftChild);
                if (rightChild) node.children.push(rightChild);

                return node;
            };

            const root = createNode(sortedComponents, 0, sortedComponents.length - 1);
            return d3.hierarchy(root);
        }
        ,
        updateSvg() {
            const svg = d3.select(this.$refs.treeSvg);

            svg.selectAll("*").remove();

           
            const nodeRadius = Math.min(this.width, this.height) * 0.05; 
            const fontSize = Math.min(this.width, this.height) * 0.020; 
            const g = svg.append("g")
                .attr("transform", `translate(${this.width / 2},${this.margin.top})`);

            const nodes = g.selectAll(".node")
                .data(this.nodes)
                .enter()
                .append("g")
                .attr("class", "node")
                .attr("transform", (d) => `translate(${d.x - this.width / 2},${d.y})`);

            nodes.append("circle")
                .attr("r", nodeRadius)
                .attr("fill", "#3182bd");

            nodes.append("text")
                .attr("dy", 3)
                .attr("x", (d) => (d.children ? -nodeRadius : nodeRadius))
                .style("text-anchor", (d) => (d.children ? "end" : "start"))
                .style("font-size", `${fontSize}px`)
                .text((d) => d.name);

            const { width, height } = this;
            svg.attr("viewBox", `-50 0 ${width} ${height}`);
        }




    },
};
</script>

<style scoped>
.d3-tree-container {
    margin-bottom: 20px;
}
</style>
