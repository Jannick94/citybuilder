<template>
    <div class="map" style="overflow: auto; width: 100%; height: 100%;">
        <div class="debug-bar" :key="fps">
            <span class="debug-bar-info">
                {{ fps }} fps &mdash;
                x: {{ coordinates.x }} &mdash;
                y: {{ coordinates.y }}
            </span>
        </div>
        <img src="/img/grass.png" ref="1" hidden>
        <img src="/img/dirt.png" ref="2" hidden>
        <img src="/img/water.png" ref="3" hidden>
        <img src="/img/bridgeNorth.png" ref="4" hidden>
        <img src="/img/roadNorth.png" ref="5" hidden>

        <canvas
            ref="canvas"
            @mousedown="startDragging()"
            @mousemove="dragViewport($event)"
            @mouseleave="stopDragging()"
            @mouseup="stopDragging()"
        ></canvas>
    </div>
</template>

<script>
    export default {
        name: 'map-view',
        data() {
            return {
                canvas: null,
                context: null,

                tilemap: [
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                ],

                secondsPassed: null,
                oldTimeStamp: null,
                fps: null,

                coordinates: {
                    x: 0,
                    y: 0,
                },

                isDragging: false,
            }
        },
        methods: {
            init() {
                this.canvas = this.$refs.canvas;
                this.context = this.canvas.getContext('2d');
            },
            gameLoop(timestamp) {
                this.secondsPassed = (timestamp - this.oldTimeStamp) / 1000;
                this.oldTimeStamp = timestamp;

                // Calculate fps
                this.fps = Math.round(1 / this.secondsPassed);

                this.draw();

                window.requestAnimationFrame(this.gameLoop);
            },
            draw() {
                this.canvas.width = window.innerWidth;
                this.canvas.height = window.innerHeight;

                const amountOfColumns = this.tilemap[0].length;
                const amountOfRows = this.tilemap.length;

                this.context.translate(this.coordinates.x, this.coordinates.y);

                for (let x = 0; x < amountOfRows; x++) {
                    for (let y = 0; y < amountOfColumns; y++) {
                        this.drawImageTile(this.$refs[this.tilemap[x][y]], x, y);
                    }
                }
            },
            drawImageTile(tile, x, y) {
                if (!tile) {
                    tile = this.$refs[1];
                }

                const gridWidth = 100;
                const gridHeight = 50;
                const spriteWidth = gridWidth;
                const spriteHeight = tile.height / tile.width * gridWidth;
                const ox = this.canvas.width / 2 - spriteWidth / 2;
                const oy = spriteHeight;

                this.context.drawImage(
                    tile,
                    ox + (y - x) * spriteWidth / 2,
                    oy + (x + y) * gridHeight / 2 - (spriteHeight - gridHeight),
                    spriteWidth,
                    spriteHeight,
                );
            },
            startDragging() {
                this.isDragging = true;
            },
            stopDragging() {
                this.isDragging = false;
            },
            dragViewport(event) {
                if (!this.isDragging) {
                    return;
                }

                // @todo: calculate movementX/Y because it lacks support in IE
                this.coordinates = {
                    x: this.coordinates.x + event.movementX,
                    y: this.coordinates.y + event.movementY,
                };
            },
        },
        mounted() {
            this.init();

            window.requestAnimationFrame(this.gameLoop);
        },
    }
</script>
