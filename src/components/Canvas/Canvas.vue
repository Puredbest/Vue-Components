<template>
    <div>
         <canvas id="front-animation"></canvas>
    </div>
</template>

<script>
export default {
    name: 'iv-front-page',
    mounted(){
        let canvas = document.querySelector('canvas');
        canvas.width = window.innerWidth*0.65;
        canvas.height = window.innerHeight*0.8;

        let c = canvas.getContext('2d');

        // c.fillStyle = 'rgba(255,0,0,0.5)';
        // c.fillRect(100,100, 100, 100);
        // c.fillStyle = 'rgba(0,0,255,0.5)';
        // c.fillRect(150,150, 100, 100);
        // c.fillStyle = 'rgba(0,255,0,0.5)';
        // c.fillRect(300,0, 100, 100);

        // c.beginPath();
        // c.moveTo(50, 300);
        // c.lineTo(300, 100);
        // c.lineTo(400,300);
        // c.strokeStyle = 'rgba(255,0,255,0.5)';
        // c.stroke();

        // for(let i = 0; i < 10; i++){
        //     let x = Math.random() * window.innerWidth * 0.65;
        //     let y = Math.random() * window.innerHeight * 0.8;
        //     let r = Math.random() * 255;
        //     let g = Math.random() * 255;
        //     let b = Math.random() * 255;

        //     c.beginPath();
        //     c.arc(x,y, 30, 0, Math.PI * 2);
        //     c.strokeStyle = 'rgb(' + r + ',' + g + ',' + b +')';
        //     c.stroke();
        // 

        // let radius = 5;
        // let ballNumRoot = 2;
        // let xPositions = [];
        // let yPositions = [];
        // for (let i = 0; i < ballNumRoot; i++){
        //     for (let j = 0; j < ballNumRoot; j++){
        //         xPositions.push(i*canvas.width/ballNumRoot + radius);
        //         yPositions.push(j*canvas.height/ballNumRoot + radius);
        //     }
        // }

        function rotateShape(angle, centre, points){
            // console.log(angle, centre, points);
            for(let i = 0; i < 3; i++){
                points[i] = [points[i][0]*Math.cos(angle) - points[i][1]*Math.sin(angle), points[i][0]*Math.sin(angle) + points[i][1]*Math.cos(angle)];
                points[i][0] = centre[0] + points[i][0];
                points[i][1] = centre[1] + points[i][1];
            }
            return points;
        }
        
        function Ball(x, y) {
            this.x = x;
            this.y = y;

            this.dx = 0;
            this.dy = 0.5;

            this.draw = function() {
                let points = rotateShape(this.angle, [this.x, this.y], [[0, radius], [-0.5*radius, -radius], [0.5*radius, -radius]]);

                c.beginPath();
                c.moveTo(points[0][0], points[0][1]);
                c.lineTo(points[1][0], points[1][1]);
                c.lineTo(points[2][0], points[2][1]);
                //c.arc(this.x, this.y, radius, 0, Math.PI * 2);
                c.fillStyle = 'blue';
                c.fill(); 
            }

            this.update = function() {
                // if (this.x + radius > canvas.width || this.x - radius < 0){
                //     this.dx = -this.dx;
                // }
                // if (this.y + radius > canvas.height || this.y - radius < 0){
                //     this.dy = -this.dy;
                // }

                this.x += this.dx;
                this.y += this.dy;
                this.angle = -Math.atan2(this.dx,this.dy);
                
                let dist = Math.pow(Math.pow(this.x-massCentre[0] , 2) + Math.pow(this.y-massCentre[1], 2), 0.5);
                //let angle = Math.atan2(this.y-massCentre[1], this.x-massCentre[0]);

                if(this.x - massCentre[0] !== 0){
                    this.dx += 1*Math.abs(Math.pow(dist , -2))*(massCentre[0]-this.x);
                }
                if(this.y - massCentre[1] !== 0){
                    this.dy += 1*Math.abs(Math.pow(dist , -2))*(massCentre[1]-this.y);
                }

                this.draw();
            }
        }
        
        
        let radius = 2;
        let ballArray = [];
        let ballNumRoot = 70;
        let massCentre = [200, 200];

        for (let i = 0; i < ballNumRoot; i++){
            for (let j = 0; j < ballNumRoot; j++){
                let x = i*canvas.width/ballNumRoot + radius;
                let y = j*canvas.height/ballNumRoot + radius;
                
                ballArray.push(new Ball(x,y));
            }
        }

        function animate() {
            requestAnimationFrame(animate);

            c.clearRect(0,0, innerWidth, innerHeight);

            for (let i = 0; i < ballArray.length; i++){
                ballArray[i].update();
            }

            c.beginPath();
            c.arc(massCentre[0], massCentre[1], 5, 0, Math.PI * 2); 
            c.fillStyle = 'black';
            c.fill(); 
        }

        animate();
    },


}
</script>

<style>
#front-animation{
    border: 1px solid black;
}
</style>