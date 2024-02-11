<template>
    <div>
        <input type="file" @change="convertToPixel" />
        <canvas ref="canvasRef"></canvas>
    </div>
</template>

<script>
import { ref } from "vue";

export default {
    setup() {
        const canvasRef = ref(null);

        const convertToPixel = ({ target }) => {
            const file = target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = (e) => {
                const img = new Image();
                img.onload = () => {
                    const offscreenCanvas = document.createElement("canvas");
                    const offscreenCtx = offscreenCanvas.getContext("2d");

                    offscreenCanvas.width = 100;
                    offscreenCanvas.height = 150;
                    offscreenCtx.drawImage(img, 0, 0, 100, 150);

                    const smallImageData = offscreenCtx.getImageData(0, 0, 100, 150);
                    const data = smallImageData.data;
                    for (let i = 0; i < data.length; i += 4) {
                        const grayscale = data[i] * 0.3 + data[i + 1] * 0.59 + data[i + 2] * 0.11;
                        const threshold = grayscale > 128 ? 255 : 0;
                        data[i] = threshold; // red
                        data[i + 1] = threshold; // green
                        data[i + 2] = threshold; // blue
                    }
                    offscreenCtx.putImageData(smallImageData, 0, 0);

                    const canvas = canvasRef.value;
                    canvas.width = 1000; // 실제 크기
                    canvas.height = 1500; // 실제 크기
                    const ctx = canvas.getContext("2d");

                    for (let y = 0; y < 150; y++) {
                        for (let x = 0; x < 100; x++) {
                            const pixel = offscreenCtx.getImageData(x, y, 1, 1).data;
                            ctx.fillStyle = `rgb(${pixel[0]}, ${pixel[1]}, ${pixel[2]})`;
                            ctx.fillRect(x * 10, y * 10, 10, 10);
                        }
                    }
                };
                img.src = e.target.result;
            };
            reader.readAsDataURL(file);
        };
        return {
            canvasRef,
            convertToPixel
        };
    }
};
</script>

<style scoped>
header {
    line-height: 1.5;
}

.logo {
    display: block;
    margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
    header {
        display: flex;
        place-items: center;
        padding-right: calc(var(--section-gap) / 2);
    }

    .logo {
        margin: 0 2rem 0 0;
    }

    header .wrapper {
        display: flex;
        place-items: flex-start;
        flex-wrap: wrap;
    }
}
</style>
