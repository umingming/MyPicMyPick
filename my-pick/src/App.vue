<template>
    <div id="grid" ref="gridRef">
        <div
            v-for="col of count"
            :key="getCellKey(-1, col)"
            ref="configRefs"
            class="config col-config"
        ></div>
        <div v-for="row of count" :key="row" ref="rowRefs">
            <div
                v-for="col of count"
                :key="getCellKey(row, col)"
                ref="cellRefs"
                class="cell"
                @click="clickCell"
                :data-row="row"
                :data-col="col"
            ></div>
        </div>
    </div>
    <div id="box">
        <input type="file" @change="loadImage" />
        <canvas ref="canvasRef"></canvas>
        <div id="drag-area" @dragover.prevent="dragImage" @drop.prevent="dropImage"></div>
    </div>
</template>

<script>
import { onMounted, reactive, ref, nextTick, readonly } from "vue";

export default {
    setup() {
        //============================= Image Upload
        const canvasRef = ref(null);
        const img = reactive(new Image());

        /**
         * @param {DragEvent} event
         */
        function dragImage({ dataTransfer }) {
            dataTransfer.dropEffect = "copy";
        }

        /**
         * @param {DragEvent} event
         */
        function dropImage({ dataTransfer: { files } }) {
            if (files.length) {
                loadImage({ target: { files } });
            }
        }

        /**
         * @param {Event} event
         * @param {Object} event.target
         */
        function loadImage({ target }) {
            const file = target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = ({ target: { result } }) => {
                    img.onload = () => {
                        const { width, height } = img;
                        canvasRef.value.width = width;
                        canvasRef.value.height = height;
                        drawImage();
                    };
                    img.src = result;
                };
                reader.readAsDataURL(file);
            }
        }

        /**
         * 캔버스 크기에 맞춰 이미지를 그린다.
         */
        function drawImage() {
            const { width, height } = canvasRef.value;
            const ctx = canvasRef.value.getContext("2d");
            ctx.drawImage(img, 0, 0, width, height);
        }

        //============================= Resize
        const isResizing = ref(false);
        const point = reactive({ x: 0, y: 0 });

        /**
         * @param {Number} x
         * @param {Number} y
         */
        function initPoint(x, y) {
            point.x = x;
            point.y = y;
        }

        /**
         * @param {Number} x
         * @param {Number} y
         */
        function resizeImage(x, y) {
            if (isResizing.value) {
                canvasRef.value.width += x - point.x;
                canvasRef.value.height += y - point.y;
                drawImage();
                initPoint(x, y);
            }
        }

        onMounted(() => {
            canvasRef.value.addEventListener("mousedown", ({ offsetX, offsetY, x, y }) => {
                const { width, height } = canvasRef.value;
                if (Math.abs(width - offsetX) < 10 && Math.abs(height - offsetY) < 10) {
                    initPoint(x, y);
                    isResizing.value = true;
                }
            });
            window.addEventListener("mousemove", ({ x, y }) => resizeImage(x, y));
            window.addEventListener("mouseup", () => (isResizing.value = false));
        });

        //============================= Grid Layout
        const gridRef = ref(null);
        const rowRefs = ref([]);
        const cellRefs = ref([]);
        const configRefs = ref([]);
        const count = 10;

        const CELL = readonly({
            ACTIVE: "active",
            INACTIVE: "inactive",
            CHECKED: "checked"
        });

        function initGrid() {
            const size = 1000 / count + "px";

            rowRefs.value.forEach((row) => (row.style.height = size));
            cellRefs.value.forEach((cell) => (cell.style.width = size));
            configRefs.value.forEach((cell) => (cell.style.width = size));
        }

        function getCellKey(row, col) {
            return `${row}-${col}`;
        }

        function clickCell({ target }) {
            Object.keys(CELL)
                .filter((className) => className !== CELL.ACTIVE)
                .forEach((className) => target.classList.remove(className));
            target.classList.toggle(CELL.ACTIVE);
        }

        onMounted(() => nextTick(initGrid));

        //============================= Logic
        const logicMap = [
            [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
            [0, 0, 1, 1, 0, 0, 1, 1, 0, 0],
            [0, 1, 0, 0, 1, 1, 0, 0, 1, 0],
            [1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
            [1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
            [0, 1, 0, 0, 0, 0, 0, 0, 1, 0],
            [0, 0, 1, 0, 0, 0, 0, 1, 0, 0],
            [0, 0, 0, 1, 0, 0, 1, 0, 0, 0],
            [0, 0, 0, 0, 1, 1, 0, 0, 0, 0],
            [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
        ];
        const config = {
            row: Array.from({ length: count }, () => [0]),
            col: Array.from({ length: count }, () => [0])
        };

        for (let i = 0; i < count; i++) {
            for (let j = 0; j < count; j++) {
                updateConfig(config.row, i, logicMap[i][j]);
                updateConfig(config.col, j, logicMap[i][j]);
            }
        }

        function updateConfig(config, index, flag) {
            flag ? config[index][config[index].length - 1]++ : config[index].push(0);
        }

        config.row = config.row.map((config) => config.filter((value) => value));
        config.col = config.col.map((config) => config.filter((value) => value));

        return {
            // Upload
            canvasRef,
            dragImage,
            dropImage,
            loadImage,

            // Grid
            gridRef,
            rowRefs,
            cellRefs,
            configRefs,
            count,
            getCellKey,
            clickCell

            // Grid
        };
    }
};
</script>

<style>
/* Grid */
#grid {
    width: 1202px;
    height: 1202px;
    border: 1px solid black;
}
.cell {
    display: inline-block;
    background-color: #eee; /* 셀 배경색 */
    border: 1px solid #ddd; /* 셀 경계선 */
    height: 100%;
}
.cell.active {
    background-color: black;
}
.config {
    display: inline-block;
    border: 1px solid #ddd;
}
.config.col-config {
    height: 200px;
}
.config.row-config {
    width: 200px;
    height: 100%;
}

/* Box */
#box {
    position: relative;
    width: 100vw;
    height: 100vh;
}
#drag-area {
    width: 500px;
    height: 100px;
    background: gray;
}
canvas {
    position: relative;
    left: 0;
    top: 0;
}
</style>
