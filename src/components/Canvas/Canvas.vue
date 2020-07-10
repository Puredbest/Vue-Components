<template>
    <div id="parent" style="height:70vh;">
         <canvas id="front-animation" style="width:100%; height:100%"></canvas>
    </div>
</template>

<script>
export default {
    name: 'iv-front-page',
    mounted(){
        let canvas = document.querySelector('canvas');
        // canvas.width = window.innerWidth*0.65;
        // canvas.height = window.innerHeight*0.8;
        let parent = document.getElementById('parent');
        canvas.width = parent.offsetWidth;
        canvas.height = parent.offsetHeight;

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
        let mouse = {
            x: undefined,
            y: undefined
        }
        
        let rect = parent.getBoundingClientRect();
        let timer;

        parent.addEventListener('mousemove', function(event){
            mouse.x = event.x - rect.left;
            mouse.y = event.y - rect.top;

            clearInterval(timer);
            timer = setInterval(function(){mouse.x = undefined; mouse.y = undefined}, 10000);
        })

        // parent.addEventListener('onmouseleave', function(){
        //     console.log('mouse out');
        //     mouse.x = undefined;
        //     mouse.y = undefined;
        // })


        function rotateShape(angle, centre, points){
            // console.log(angle, centre, points);
            for(let i = 0; i < 3; i++){
                points[i] = [points[i][0]*Math.cos(angle) - points[i][1]*Math.sin(angle), points[i][0]*Math.sin(angle) + points[i][1]*Math.cos(angle)];
                points[i][0] = centre[0] + points[i][0];
                points[i][1] = centre[1] + points[i][1];
            }
            return points;
        }
        
        function Ball(x, y, defaultRadius) {
            this.x = x;
            this.y = y;
            this.radius = defaultRadius;

            //make all orbit in circular orbits in same direction initially 
            this.dx = 0;
            this.dy = 0;

            for(let i = 0; i < massCentres.length; i++){  
                this.dist = Math.pow(Math.pow(this.x-massCentres[i][0] , 2) + Math.pow(this.y-massCentres[i][1], 2), 0.5);
                this.dy += -initialVel*massCentres[i][2]*Math.abs(Math.pow(this.dist , -2))*(massCentres[i][0]-this.x);
            }
            for(let i = 0; i < massCentres.length; i++){
                this.dist = Math.pow(Math.pow(this.x-massCentres[i][0] , 2) + Math.pow(this.y-massCentres[i][1], 2), 0.5);
                this.dx += initialVel*massCentres[i][2]*Math.abs(Math.pow(this.dist , -2))*(massCentres[i][1]-this.y);
            }

            this.draw = function() {
                let points = rotateShape(this.angle, [this.x, this.y], [[0, this.radius], [-0.5*this.radius, -this.radius], [0.5*this.radius, -this.radius]]);

                c.beginPath();
                c.moveTo(points[0][0], points[0][1]);
                c.lineTo(points[1][0], points[1][1]);
                c.lineTo(points[2][0], points[2][1]);
                //c.fillStyle = 'blue';
                let b = 255*Math.pow(Math.pow(this.dx,2) + Math.pow(this.dy, 2), 0.25);
                //console.log(b);
                c.fillStyle = 'rgb('+ b + ',' + b + ',255)';
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
                
                for(let i = 0; i < massCentres.length; i++){
                    let dist = Math.pow(Math.pow(this.x-massCentres[i][0] , 2) + Math.pow(this.y-massCentres[i][1], 2), 0.5);

                    if(this.x - massCentres[i][0] !== 0){
                        this.dx += massCentres[i][2]*Math.abs(Math.pow(dist , -2))*(massCentres[i][0]-this.x);
                    }
                    if(this.y - massCentres[i][1] !== 0){
                        this.dy += massCentres[i][2]*Math.abs(Math.pow(dist , -2))*(massCentres[i][1]-this.y);
                    }
                }

                //interactivity with mouse
                if(Math.pow((Math.pow((mouse.x-this.x),2) + Math.pow((mouse.y-this.y),2)),1/2) < mouseRadius){
                    if(this.radius < maxRadius){
                        this.radius += 0.5;
                    }
                } else if (this.radius > defaultRadius) {
                    this.radius -= 0.5;
                }

                

                this.draw();
            }
        }
        
        
        let radius = canvas.height/150;
        let maxRadius = 30;
        let mouseRadius = 55;
        let ballArray = [];
        let ballNumRoot = 40;

        // [[x,y, mass]]
        //let massCentres = [[canvas.width/3, canvas.height/3], [2*canvas.width/3, canvas.height/3], [canvas.width/2, 2*canvas.height/3]];
        let massCentres = [[canvas.width/2, canvas.height/2 , 0.4]];
        let initialVel = 50;

        for (let i = 0; i < ballNumRoot; i++){
            for (let j = 0; j < ballNumRoot; j++){
                let x = i*canvas.width/ballNumRoot + radius;
                let y = j*canvas.height/ballNumRoot + radius;
                
                ballArray.push(new Ball(x,y, radius));
            }
        }

        function animate() {
            requestAnimationFrame(animate);

            c.clearRect(0,0, innerWidth, innerHeight);

            for (let i = 0; i < ballArray.length; i++){
                ballArray[i].update();
            }

            // for (let i = 0; i <massCentres.length; i++){
            //     c.beginPath();
            //     c.arc(massCentres[i][0], massCentres[i][1], 5, 0, Math.PI * 2); 
            //     c.fillStyle = 'black';
            //     c.fill();
            // }
            
        }

        animate();
    }

}
</script>

<style>
#front-animation{
    /* border: 1px solid black; */
}
</style>