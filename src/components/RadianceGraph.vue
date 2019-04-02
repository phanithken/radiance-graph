<!-- template -->
<template>
    <div ref="wrapper" class="container"></div>
</template>
<!-- end template -->


<!-- script -->
<script>
export default {
    name: 'radiance-graph',
    props: {
        circles: { required: true },
        max: { required: true },
        scale: { required: true },
        firstValue: { required: true },
        secondValue: { required: true },
        thirdValue: { required: true }
    },
    data () {
        return {
            min: 0,
            angleMaxA: (Math.PI * 5) / 4,
            angleMaxB: (Math.PI * 7) / 4,
            angleMaxC: Math.PI / 2,
            ctx: null,
            trictx: null,
            canvas: null,
            tricanvas: null,
            startTime: null
        }
    },
    mounted: function () {
        // create canvas for circle path and triangle
        this.canvas = document.createElement('canvas');
        this.tricanvas = document.createElement('canvas');

        // set canvas width and height
        this.canvas.height = (this.scale * 15) - (this.scale * 0.9);
        this.canvas.width = (this.scale * 15) - (this.scale * 0.9);
        this.tricanvas.height = this.canvas.height;
        this.tricanvas.width = this.canvas.width;

        // get ready for draw and amination
        this.ctx = this.canvas.getContext('2d');
        this.trictx = this.tricanvas.getContext('2d');
        this.ctx.fillRect((this.canvas.width * 0.5), (this.canvas.height * 0.5), 3, 3);

        // draw circle path and start triangle animation
        this.drawCirclePath();
        this.startAnimation();

        // add element to DOM
        this.canvas.style.position = 'absolute';
        this.$refs.wrapper.appendChild(this.canvas);
        this.$refs.wrapper.appendChild(this.tricanvas);
    },
    methods: {
        // convert value to point
        valueToPoint: function (value, angle) {
            var max = this.getMax(angle)
            var x, y
            switch (angle) {
                case this.angleMaxA:
                    var deltax = ((this.canvas.width * 0.5) - max.x) / this.max;
                    var deltay = ((this.canvas.width * 0.5) - max.x) / this.max;
                    var vcx = (this.max - value) * deltax;
                    var vcy = (this.max - value) * deltay;
                    x = max.x + vcx;
                    y = max.y + vcy;
                    break;
                case this.angleMaxB:
                    var deltax = (max.x - (this.canvas.width * 0.5)) / this.max;
                    var deltay = ((this.canvas.width * 0.5) - max.y) / this.max;
                    var vcx = (this.max - value) * deltax;
                    var vcy = (this.max - value) * deltay;
                    x = max.x - vcx;
                    y = max.y + vcy;
                    break;
                case this.angleMaxC:
                    var deltay = (max.y - (this.canvas.width * 0.5)) / this.max;
                    var vcy = (this.max - value) * deltay;
                    x = this.canvas.width * 0.5;
                    y = max.y - vcy;
                    break;
            }
            
            return {
                x: x,
                y: y
            }
        },
        // get max point on the circle of the respective angle
        getMax: function (angle) {
            var r = this.canvas.width * 0.5;
            var originx = this.canvas.width * 0.5;
            var originy = this.canvas.height * 0.5;

            return {
                x: originx + r * Math.cos(angle),
                y: originy + r * Math.sin(angle)
            }
        },
        // start animation frame
        startAnimation: function (time) {
            var firstMaxPoint = this.valueToPoint(this.firstValue, this.angleMaxA);
            var secondMaxPoint = this.valueToPoint(this.secondValue, this.angleMaxB);
            var thirdMaxPoint = this.valueToPoint(this.thirdValue, this.angleMaxC);

            var duration = 300;
            var nextX, nextY;

            var x = this.canvas.width * 0.5;
            var y = this.canvas.height * 0.5;

            if (!this.startTime) {
                this.startTime = time || performance.now();
            }

            // deltatime should be in the range between [0 - 1]
            var deltatime = (time - this.startTime) / duration;

            // current position = previous position + (difference * deltatime)
            var currentFirst = {
                x: x + ((firstMaxPoint.x - x) * deltatime),
                y: y + ((firstMaxPoint.y - y) * deltatime)
            }

            var currentSecond = {
                x: x + ((secondMaxPoint.x - x) * deltatime),
                y: y + ((secondMaxPoint.y - y) * deltatime)
            }

            var currentThird = {
                x: x + ((thirdMaxPoint.x - x) * deltatime),
                y: y + ((thirdMaxPoint.y - y) * deltatime)
            }
            
            if (deltatime >= 1) { // end animation
                this.startTime = null;
                this.drawTriangle(firstMaxPoint, secondMaxPoint, thirdMaxPoint);
            } else {
                this.drawTriangle(currentFirst, currentSecond, currentThird);
                requestAnimationFrame(this.startAnimation);
            }
        },
        // draw triangle
        drawTriangle: function (first, second, third) {
            this.trictx.clearRect(0, 0, this.canvas.width, this.canvas.height);
            this.trictx.beginPath();
            this.trictx.moveTo(first.x, first.y);
            this.trictx.lineTo(second.x, second.y);
            this.trictx.lineTo(third.x, third.y);
            this.trictx.closePath();
            this.trictx.stroke();
        },
        // draw circle path
        drawCirclePath: function () {
            for(var i = 0; i < this.circles; i++){
                this.ctx.beginPath();
                this.ctx.arc((this.canvas.width * 0.5), (this.canvas.height * 0.5), this.scale*(i+1), 0, Math.PI * 2, true);
                (i%2 == 0) ? this.ctx.strokeStyle = '#D3D3D3' : this.ctx.strokeStyle = '#808080';
                this.ctx.stroke();
            };
        }
    }
}
</script>
<!-- end script -->


<!-- style -->
<style lang="scss" scoped>
.container {
    width: 100%;
    display: block;
}
</style>
<!-- end style -->