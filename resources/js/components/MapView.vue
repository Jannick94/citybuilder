<template>
    <div class="map" style="overflow: auto; width: 100%; height: 100%;">
        <div class="debug-bar" :key="fps">
            <span class="debug-bar-info">
                {{ fps }} fps &mdash;
                x: {{ coordinates.x }} &mdash;
                y: {{ coordinates.y }}
            </span>
            <button type="button" @click="increaseScale()">&plus;</button>
            <button type="button" @click="decreaseScale()">&minus;</button>

            <button type="button" @click="moveViewport()">Move to 100x100</button>
        </div>
        <img src="/img/grass.png" ref="1" hidden>
        <img src="/img/dirt.png" ref="2" hidden>
        <img src="/img/water.png" ref="3" hidden>
        <img src="/img/bridgeNorth.png" ref="4" hidden>
        <img src="/img/roadNorth.png" ref="5" hidden>
        <img src="/img/building.png" ref="6" hidden>
        <img src="/img/building2.png" ref="7" hidden>

        <canvas
            ref="canvas"
            @mousedown="startDragging()"
            @mousemove="dragViewport($event); hoverTile($event)"
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
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
                ],

                secondsPassed: null,
                oldTimeStamp: null,
                fps: null,

                coordinates: {
                    x: 0,
                    y: 0,
                },

                hoverCoordinates: {
                    x: 0,
                    y: 0,
                },

                scale: 1,

                isDragging: false,

                test: 0,
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
                this.context.scale(this.scale, this.scale);

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

                const spriteX = ox + (y - x) * spriteWidth / 2;
                const spriteY = oy + (x + y) * gridHeight / 2 - (spriteHeight - gridHeight);

                this.context.drawImage(tile, spriteX, spriteY, spriteWidth, spriteHeight);

                // if (x === 0 && y === 0) {
                    // this.context.strokeRect(spriteX + spriteWidth / 2, spriteY, 1, 1);
                    // this.context.strokeRect(spriteX, spriteY + spriteHeight / 2, 1, 1);
                    // this.context.strokeRect(spriteX + spriteWidth, spriteY + spriteHeight / 2, 1, 1);
                    // this.context.strokeRect(spriteX + spriteWidth / 2, spriteY + spriteHeight, 1, 1);
                    const path = new Path2D();

                    path.lineTo(spriteX + spriteWidth / 2, spriteY);
                    path.lineTo(spriteX, spriteY + spriteHeight / 2 - 7.5);
                    path.lineTo(spriteX + spriteWidth / 2, spriteY + spriteHeight - 15);
                    path.lineTo(spriteX + spriteWidth, spriteY + spriteHeight / 2 - 7.5);
                    path.lineTo(spriteX + spriteWidth / 2, spriteY);
                    // this.context.strokeStyle = 'rgba(0,0,0,0)';
                    // this.context.stroke(path);

                    if (this.context.isPointInPath(path, this.hoverCoordinates.x, this.hoverCoordinates.y)) {
                        this.context.fillStyle = 'rgba(0,0,0,0.3)';
                        this.context.fill(path);
                    }
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
            increaseScale() {
                if (this.scale >= 1.4) {
                    return;
                }

                this.scale += 0.1;
            },
            decreaseScale() {
                if (this.scale <= 0.4) {
                    return;
                }

                this.scale -= 0.1;
            },
            moveViewport() {
                const interval = setInterval(() => {
                    if (this.coordinates.x >= 100 && this.coordinates.y >= 100) {
                        clearInterval(interval);

                        return;
                    }

                    this.coordinates = {
                        x: this.coordinates.x + 10,
                        y: this.coordinates.y + 10,
                    };
                }, 20);
            },
            hoverTile(event) {
                const rect = this.canvas.getBoundingClientRect();

                this.hoverCoordinates.x = event.clientX - rect.left - this.coordinates.x;
                this.hoverCoordinates.y = event.clientY - rect.top - this.coordinates.y;
            },
        },
        mounted() {
            this.init();

            window.requestAnimationFrame(this.gameLoop);
        },
    }
</script>
