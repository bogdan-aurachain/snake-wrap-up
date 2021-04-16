<template>
    <div class="hello">
        <canvas ref="canvas" :height="size * zoom" :width="size * zoom"></canvas>
        <button @click="onStopClick">STOP</button>
    </div>
</template>

<script>
import apple from "@/assets/apple.svg";

const image = new Image();
image.src = apple;
let ctx, interval;

export default {
    name: "HelloWorld",
    props: {
        size: { type: Number, default: 31 },
        zoom: { type: Number, default: 15 }
    },
    data() {
        return {
            snake: null,
            snakeBody: null,
            direction: null,
            food: null,
            keyStack: []
        };
    },
    computed: {
        xAxis() {
            const values = {
                right: 1, left: -1
            };
            return values[this.direction] || 0;
        },
        yAxis() {
            const values = {
                down: 1, up: -1
            };
            return values[this.direction] || 0;
        }
    },
    mounted() {
        const { canvas } = this.$refs;
        ctx = canvas.getContext("2d");

        this.initGame();
        window.addEventListener("keydown", this.keyHandler);
        interval = setInterval(this.runGame, 125);
    },
    beforeDestroy() {
        window.removeEventListener("keydown", this.keyHandler);
    },
    methods: {
        keyHandler(event) {
            const { key, keyCode } = event;

            if (keyCode > 36 && keyCode < 41) {
                const ignoredDirectionChanges = {
                    right: "left", left: "right", up: "down", down: "up"
                };
                const newDirection = key.replace("Arrow", "").toLowerCase();
                const previousDirection = this.keyStack.length ? this.keyStack.slice(-1) : this.direction;

                if (newDirection !== ignoredDirectionChanges[previousDirection]) {
                    this.keyStack.push(newDirection);
                }
            }
        },
        onStopClick() {
            clearInterval(interval);
        },
        initGame() {
            const middle = Math.floor(this.size / 2);
            this.snake = [{ x: middle, y: middle }];
            this.direction = "right";
            this.snakeBody = 3;
            this.keyStack = [];
            this.addFood();
        },
        addFood() {
            const randomPoint = () => Math.floor(Math.random() * this.size);
            let xFood, yFood;
            do {
                xFood = randomPoint();
                yFood = randomPoint();
                // eslint-disable-next-line no-loop-func
            } while (this.snake.some(({ x, y }) => xFood === x && yFood === y));

            this.food = { x: xFood, y: yFood };
        },
        runGame() {
            const { x, y } = this.snake.slice(-1)[0];
            this.direction = this.keyStack.length ? this.keyStack.shift() : this.direction;

            this.clearCanvas();
            this.drawFood();
            this.drawSnake();

            if (this.snake.length === this.snakeBody) {
                this.snake.shift();
            }

            this.snake.push({ x: x + this.xAxis, y: y + this.yAxis });
            this.checkCollisions();
        },
        clearCanvas() {
            ctx.fillStyle = "black";
            ctx.fillRect(0, 0, this.size * this.zoom, this.size * this.zoom);
        },
        drawFood() {
            const { x, y } = this.food;
            ctx.fillStyle = "yellow";
            ctx.drawImage(image, x * this.zoom, y * this.zoom, this.zoom, this.zoom);
        },
        drawSnake() {
            ctx.fillStyle = "#00CC00";
            ctx.strokeStyle = "black";
            this.snake.forEach(({ x, y }) => {
                ctx.fillRect(x * this.zoom, y * this.zoom, this.zoom, this.zoom);
                ctx.strokeRect(x * this.zoom, y * this.zoom, this.zoom, this.zoom);
            });
        },
        checkCollisions() {
            const { x: xFood, y: yFood } = this.food;
            const [head, ...tail] = [...this.snake].reverse();

            if (tail.some(({ x, y }) => x === head.x && y === head.y)) {
                this.initGame();
            }
            if (head.x === xFood && head.y === yFood) {
                this.snakeBody += 1;
                this.addFood();
            } else if (head.x < 0) {
                head.x = this.size - 1;
            } else if (head.x === this.size) {
                head.x = 0;
            } else if (head.y < 0) {
                head.y = this.size - 1;
            } else if (head.y === this.size) {
                head.y = 0;
            }
        }
    }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
