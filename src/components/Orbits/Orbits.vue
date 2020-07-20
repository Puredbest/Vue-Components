<template>
    <div id="parent" style="height:70vh;" @mousemove="mouseOver" @mouseleave="mouseLeave" @mousedown="activateSelect" @mouseup="cancelSelect">
         <canvas id="front-animation" style="width:100%; height:100%" ></canvas>
    </div>
</template>

<script>
export default {
    name: 'iv-orbits',
    props: {
        mouse: {
            default: {
                x: undefined,
                y: undefined
            }
        },
        animationSpeed: {
            default: 1
        },
        velocitySelect: {
            default: false
        }
    },
    methods: {
        mouseOver(event){
            this.mouse.x = event.x;
            this.mouse.y = event.y;
        },
        mouseLeave(){
            this.mouse.x = undefined;
            this.mouse.y = undefined;
        },
        activateSelect(){
            this.velocitySelect = true;
            console.log('activated');
        },
        cancelSelect(){
            this.velocitySelect = false;
            console.log('released');
        }
    },
    mounted(){
        let canvas = document.querySelector('canvas');
        let parent = document.getElementById('parent');
        canvas.width = parent.offsetWidth;
        canvas.height = parent.offsetHeight;
        let rect = parent.getBoundingClientRect();


        let vm = this;
        let mouseX = this.mouse.x - rect.left;
        let mouseY = this.mouse.y - rect.top;


        let c = canvas.getContext('2d');


        // function rotateShape(angle, centre, points){
        //     // console.log(angle, centre, points);
        //     for(let i = 0; i < 3; i++){
        //         points[i] = [points[i][0]*Math.cos(angle) - points[i][1]*Math.sin(angle), points[i][0]*Math.sin(angle) + points[i][1]*Math.cos(angle)];
        //         points[i][0] = centre[0] + points[i][0];
        //         points[i][1] = centre[1] + points[i][1];
        //     }
        //     return points;
        // }
        
        function Ball(x, y, defaultRadius) {
            this.x = x;
            this.y = y;
            this.radius = defaultRadius;
            this.pathIndex = 0;

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

            this.path = function()  {
                let currentPos = [this.x, this.y];
                let currentVel = [this.dx, this.dy];
                let iterations = 0;
                this.pathCoords = [];
                let stepSize = 1;
                this.scale = 1;
                this.closedPath = true;
                let reverseMultiplier = 1;
                //c.scale(0.5,0.5);
                console.log('path drawing');

                while(Math.pow((Math.pow(this.x-currentPos[0], 2) + Math.pow(this.y-currentPos[1],2)),0.5) > 1 || iterations < 100){
                    
                    if(currentPos[0] < (canvas.width - canvas.width*(1/this.scale))*(1/this.scale) || currentPos[0] > canvas.width*(1/this.scale)*(1/this.scale)|| currentPos[1] < (canvas.height - canvas.height*(1/this.scale))*(1/this.scale) || currentPos[1] > canvas.height*(1/this.scale)*(1/this.scale)){
                        
                        console.log('scale = ', this.scale);
                        console.log(currentPos[0], (canvas.width - canvas.width*(1/this.scale)), canvas.width*(1/this.scale));
                        console.log(currentPos[1], (canvas.height - canvas.height*(1/this.scale)) , canvas.height*(1/this.scale));
                        
                        c.translate(canvas.width/4, canvas.height/4);
                        this.scale = this.scale/2;
                        c.scale(this.scale, this.scale);
                        
                        //console.log('newScale = ', canvas.width/currentPos[0]);
                        // this.scale = Math.pow(Math.max(currentPos[0]/(canvas.width) , currentPos[1]/(canvas.height)), -1);
                        
                        //console.log('scale =', this.scale);
                        //c.scale(Math.pow(this.scale, 1), Math.pow(this.scale,1));
                        //console.log(Math.pow(this.scale, -1));
                        //console.log('rescale');
                    }


                    currentPos[0] += currentVel[0] * reverseMultiplier * stepSize;
                    currentPos[1] += currentVel[1] * reverseMultiplier * stepSize;
                    
                    //console.log(currentPos, currentVel);

                    for(let i = 0; i < massCentres.length; i++){
                        let dist = Math.pow(Math.pow(currentPos[0]-massCentres[i][0] , 2) + Math.pow(currentPos[1]-massCentres[i][1], 2), 0.5);

                        currentVel[0] = currentVel[0] + reverseMultiplier * massCentres[i][2]*Math.abs(Math.pow(dist , -3))*(massCentres[i][0]-currentPos[0]);
                        currentVel[1] = currentVel[1] + reverseMultiplier * massCentres[i][2]*Math.abs(Math.pow(dist , -3))*(massCentres[i][1]-currentPos[1]);
                    }

                    if (this.closedPath){
                        this.pathCoords.push(currentPos.slice());
                    } else{
                        this.pathCoords.unshift(currentPos.slice());
                    }

                    iterations += 1;
                    
                    //Draw reverse path for open path
                    if(iterations > 10000){
                        this.closedPath = false;
                        currentPos = [this.x, this.y];
                        currentVel = [this.dx, this.dy];
                        iterations = 0;
                        
                        if(reverseMultiplier < 0){
                            break;
                        }

                        reverseMultiplier = -1;
                        console.log('reverse');
                    }
                }

            }

            this.draw = function() {

                c.beginPath();
                c.moveTo(this.pathCoords[0][0], this.pathCoords[0][1]);
                for(let i = 1; i < this.pathCoords.length; i++){
                    c.lineTo(this.pathCoords[i][0], this.pathCoords[i][1]);
                }
                if(this.closedPath){
                    c.lineTo(this.pathCoords[0][0], this.pathCoords[0][1]);
                }
                c.lineWidth = 2;
                c.stroke();

                c.beginPath();
                c.arc(this.x, this.y, this.radius, 0, 2*Math.PI);
                c.fillStyle = 'rgb(0,0,255)';
                c.fill(); 

                
            }

            this.update = function() {

                this.x = this.pathCoords[this.pathIndex][0];
                this.y = this.pathCoords[this.pathIndex][1];

                this.pathIndex += vm.animationSpeed;
                if(this.pathIndex >= this.pathCoords.length){
                    this.pathIndex=0;
                }

                //interactivity with mouse
                if(Math.pow((Math.pow((mouseX-this.x),2) + Math.pow((mouseY-this.y),2)),1/2) < mouseRadius){
                    if(this.radius < maxRadius){
                        this.radius += 0.5;
                    }
                } else if (this.radius > defaultRadius) {
                    this.radius -= 0.5;
                }

                if(vm.velocitySelect && Math.pow((Math.pow((mouseX-this.x),2) + Math.pow((mouseY-this.y),2)),1/2) < mouseRadius){
                    vm.animationSpeed = 0; 
                }

                this.draw();
            }
        }
        
        let maxRadius = 30;
        let mouseRadius = 55;
        let ballArray = [];

        // [[x,y, mass]]
        //let massCentres = [[canvas.width/3, canvas.height/3], [2*canvas.width/3, canvas.height/3], [canvas.width/2, 2*canvas.height/3]];
        let massCentres = [[canvas.width/2, canvas.height/2 , 100]];
        let initialVel = 1.3;

        ballArray.push(new Ball(canvas.width/3, canvas.height/3, 5));
        for(let i = 0; i < ballArray.length; i++){
            ballArray[i].path();
            // let xScale = Math.max(ballArray[i].pathCoords[0])/canvas.width;
            // console.log(xScale);

            // let scale = Math.max([Math.max(ballArray[i].pathCoords[0])/canvas.width, Math.max(ballArray[i].pathCoords[1])/canvas.height]);
            // console.log(scale);
            // c.scale(scale,scale);
        }
        

        function animate() {
            requestAnimationFrame(animate);

            for (let i = 0; i < ballArray.length; i++){    
                c.clearRect((canvas.width-canvas.width*(1/ballArray[i].scale))*(1/ballArray[i].scale),(canvas.height-canvas.height*(1/ballArray[i].scale))*(1/ballArray[i].scale), canvas.width*(1/ballArray[i].scale)*2*(1/ballArray[i].scale), canvas.height*(1/ballArray[i].scale)*2*(1/ballArray[i].scale));
            //     bounding box
            //     c.beginPath();
            //     c.moveTo((canvas.width - canvas.width*(1/ballArray[i].scale))*(1/ballArray[i].scale), (canvas.height - canvas.height*(1/ballArray[i].scale))*(1/ballArray[i].scale));
            //     c.lineTo(canvas.width*(1/ballArray[i].scale)*(1/ballArray[i].scale), (canvas.height - canvas.height*(1/ballArray[i].scale))*(1/ballArray[i].scale));
            //     c.lineTo(canvas.width*(1/ballArray[i].scale)*(1/ballArray[i].scale), canvas.height*(1/ballArray[i].scale)*(1/ballArray[i].scale));
            //     c.lineTo((canvas.width - canvas.width*(1/ballArray[i].scale))*(1/ballArray[i].scale), canvas.height*(1/ballArray[i].scale)*(1/ballArray[i].scale));
            //     c.lineTo((canvas.width - canvas.width*(1/ballArray[i].scale))*(1/ballArray[i].scale), (canvas.height - canvas.height*(1/ballArray[i].scale))*(1/ballArray[i].scale));
            //     c.lineWidth = 100;
            //     c.stroke();
            }

            mouseX = vm.mouse.x - rect.left;
            mouseY = vm.mouse.y - rect.top;

            for (let i = 0; i < ballArray.length; i++){
                ballArray[i].update();
            }

            for (let i = 0; i <massCentres.length; i++){
                c.beginPath();
                c.arc(massCentres[i][0], massCentres[i][1], 20, 0, Math.PI * 2); 
                c.fillStyle = 'Orange';
                c.fill();
            }

            
            // c.beginPath();
            // c.arc(2316, 669, 500, 0, Math.PI * 2); 
            // c.fillStyle = 'Red';
            // c.fill();
            
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