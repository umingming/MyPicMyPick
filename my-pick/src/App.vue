<template>
    <div id="box">
        <input type="file" @change="loadImage" />
        <canvas ref="canvasRef"></canvas>
        <div id="drag-area" @dragover.prevent="dragImage" @drop.prevent="dropImage"></div>
    </div>
</template>

<script>
import { onMounted, reactive, ref } from "vue";

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
        return {
            // Upload
            canvasRef,
            dragImage,
            dropImage,
            loadImage
        };
    }
};
</script>

<style>
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
