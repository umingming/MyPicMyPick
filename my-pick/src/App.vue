<template>
    <div id="box">
        <input type="file" @change="loadImage" />
        <canvas ref="canvasRef"></canvas>
        <button @click="updateSize">사이즈 조정</button>
    </div>
</template>

<script>
import { onMounted, reactive, ref } from "vue";

export default {
    setup() {
        //============================= Image Upload
        const canvasRef = ref(null);
        const img = reactive(new Image());

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

        function drawImage() {
            const { width, height } = canvasRef.value;
            const ctx = canvasRef.value.getContext("2d");
            ctx.drawImage(img, 0, 0, width, height);
        }

        //============================= Resize
        const isResizing = ref(false);
        const point = reactive({ x: 0, y: 0 });

        function initPoint(x, y) {
            point.x = x;
            point.y = y;
        }

        function resizeImage(x, y) {
            canvasRef.value.width += x - point.x;
            canvasRef.value.height += y - point.y;
            drawImage();
            initPoint(x, y);
        }

        onMounted(() => {
            canvasRef.value.addEventListener("mousedown", ({ offsetX, offsetY, x, y }) => {
                const { width, height } = canvasRef.value;
                if (Math.abs(width - offsetX) < 10 && Math.abs(height - offsetY) < 10) {
                    initPoint(x, y);
                    isResizing.value = true;
                }
            });
            window.addEventListener("mousemove", ({ x, y }) => {
                if (isResizing.value) {
                    resizeImage(x, y);
                }
            });
            window.addEventListener("mouseup", () => {
                isResizing.value = false;
            });
        });
        return { canvasRef, loadImage };
    }
};
</script>

<style>
#box {
    position: relative;
    width: 100vw;
    height: 100vh;
}
canvas {
    position: relative;
    left: 0;
    top: 0;
}
</style>
