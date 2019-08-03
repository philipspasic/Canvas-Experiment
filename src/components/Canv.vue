<template>
    <div class='canv' ref='canvWrapper'>
        <canvas ref='canvas' @mousemove='updateMouse'></canvas>
    </div>
</template>

<script>
import HelperFunctions from '../mixins/helperFunctions.js'

export default {
    name: 'canv',

    data: () => ({
        width: 0,
        height: 0,
        mouse: {
            x: 0,
            y: 0
        },
        objects: [],
        colors: ['red', 'blue', 'green', 'orange']
    }),

    props: [],

    mixins: [
        HelperFunctions
    ],

    watch: {},

    mounted() {
        this.canvWrapper = this.$refs.canvWrapper
        this.canv = this.$refs.canvas
        this.ctx = this.canv.getContext('2d')

        window.addEventListener('resize', this.resizeHandler)
        window.addEventListener('orientationchange', this.resizeHandler)
        this.resizeHandler()

        this.animate()
    },

    methods: {
        updateSize() {
            if(!this.canvWrapper) {
                return
            }

            this.width = this.canvWrapper.offsetWidth
            this.height = this.canvWrapper.offsetHeight

            this.canv.width = this.width
            this.canv.height = this.height
        },

        resizeHandler() {
            this.updateSize()
            this.initObjects()
        },

        initObject(x, y, velocity, radius, color) {
            let self = this

            function Object(x, y, velocity, radius, color) {
                this.x = x
                this.y = y
                this.velocity = velocity
                this.radius = radius
                this.color = color
                this.mass = 1
            }

            Object.prototype.draw = function() {
                self.ctx.beginPath()
                self.ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2, false)
                self.ctx.strokeStyle = this.color
                self.ctx.stroke()
                self.ctx.closePath()
            }

            Object.prototype.update = function() {
                this.draw()

                // RESOLVE COLLISION
                if (self.objects && self.objects.length > 1) {
                    for (let i = 0; i < self.objects.length; i++) {
                        if (this === self.objects[i]) continue;
                        if (self.distance(this.x, this.y, self.objects[i].x, self.objects[i].y) - (this.radius + self.objects[i].radius) <= 0) {
                            self.resolveCollision(this, self.objects[i])
                        } 
                    }
                }

                // MOVE OBJECT AND BOUNCE IT OFF THE EDGES OF THE CANVAS
                if (this.x + this.radius > self.width || this.x - this.radius < 0) {
                    this.velocity.x = -this.velocity.x
                }

                if (this.y + this.radius > self.height || this.y - this.radius < 0) {
                    this.velocity.y = -this.velocity.y
                }

                this.x += this.velocity.x
                this.y += this.velocity.y
            }

            return new Object(x, y, velocity, radius, color)
        },

        initObjects() {
            this.objects = []

            for (let i = 0; i < Math.ceil(this.width/40); i++) {
                let radius = this.randomIntFromRange(10, this.width/25)
                let x = this.randomIntFromRange(radius, this.width - radius)
                let y = this.randomIntFromRange(radius, this.height - radius)
                let velocity = {
                    x: (Math.random() - 0.5) * 8,
                    y: (Math.random() - 0.5) * 8
                }
                let color = this.colors[i%this.colors.length]

                // SPAWN WITHOUT COLIDING
                if(i != 0) {
                    for (let j = 0; j < this.objects.length; j++) {
                        if (this.distance(x, y, this.objects[j].x, this.objects[j].y) - (radius + this.objects[j].radius) <= 0) {
                            x = this.randomIntFromRange(radius, this.width - radius)
                            y = this.randomIntFromRange(radius, this.height - radius)

                            j = -1
                        }
                    }
                }

                this.objects.push(this.initObject(x, y, velocity, radius, color))
            }
        },

        animate() {
            if(this._isDestroyed){
                return
            }

            requestAnimationFrame(this.animate)
            this.ctx.clearRect(0, 0, this.width, this.height)

            for (let i = 0; i < this.objects.length; i++) {
                this.objects[i].update()
            }
        }
    },

    computed: {
    },

    components: {
    }
}
</script>

<style scoped="">
.canv {
    height: 100vh;
    width: 100vw;
}
</style>