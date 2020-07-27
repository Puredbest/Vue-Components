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
        },
        cancelSelect(){
            this.velocitySelect = false;
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
        //centre about x=0, y=0
        c.translate(canvas.width/2, canvas.height/2);


        function rotateShape(angle, centre, points){
            //console.log(angle, centre, points);
            for(let i = 0; i < points.length; i++){
                points[i] = [points[i][0]*Math.cos(angle) - points[i][1]*Math.sin(angle), points[i][0]*Math.sin(angle) + points[i][1]*Math.cos(angle)];
                points[i][0] = centre[0] + points[i][0];
                points[i][1] = centre[1] + points[i][1];
            }
            return points;
        }
        
        function Ball(x, y, defaultRadius, dx, dy) {
            this.x = x;
            this.y = y;
            this.radius = defaultRadius;
            this.pathIndex = 0;
            this.velocitySelect = false;
            this.timeSinceClick = 10;
            this.buttonStep = 0.1;

            if(dx == undefined || dy == undefined){
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
            } else {
                this.dx = dx;
                this.dy = dy;
            }

            this.newdx = this.dx;
            this.newdy = this.dy;


            this.path = function(dx, dy)  {
                let currentPos = [this.x, this.y];
                let currentVel = [this.dx, this.dy];
                console.log(this.velocitySelect);
                if(this.velocitySelect){
                    currentVel = [dx, dy];
                    console.log('new Velocity', [dx,dy]);
                } else {      
                    this.pathVels = [];
                    this.pathCoords = [];
                    this.scale = 1;
                }
                let iterations = 0;
                this.newCoords = [];
                this.newVels = [];
                let stepSize = 1;
                this.closedPath = true;
                let reverseMultiplier = 1;
                console.log('path drawing');

                while(Math.pow((Math.pow(this.x-currentPos[0], 2) + Math.pow(this.y-currentPos[1],2)),0.5) > 1 || iterations < 100){
                    
                    if(currentPos[0] < -canvas.width*(1/this.scale)/2 || currentPos[0] > canvas.width*(1/this.scale)/2 || currentPos[1] < -canvas.height*(1/this.scale)/2 || currentPos[1] > canvas.height*(1/this.scale)/2){
                        
                        console.log('scale = ', this.scale);
                        console.log(currentPos[0], (canvas.width - canvas.width*(1/this.scale)), canvas.width*(1/this.scale));
                        console.log(currentPos[1], (canvas.height - canvas.height*(1/this.scale)) , canvas.height*(1/this.scale));
                        
                        c.scale((1/this.scale), (1/this.scale));
                        // c.translate(canvas.width/4, canvas.height/4);
                        this.scale = this.scale/2;
                        c.scale(this.scale, this.scale);
                        console.log("Scale: ", this.scale);
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
                        if(this.velocitySelect){
                            this.newCoords.push(currentPos.slice());
                            this.newVels.push(currentVel.slice());
                        } else{
                            this.pathCoords.push(currentPos.slice());
                            this.pathVels.push(currentVel.slice());
                        }
                    } else{
                        if(this.velocitySelect){
                            this.newCoords.unshift(currentPos.slice());
                            this.newVels.unshift(currentVel.slice());
                        } else{
                            this.pathCoords.unshift(currentPos.slice());
                            this.pathVels.unshift(currentVel.slice());
                        }
                    }

                    iterations += 1;
                    
                    //Draw reverse path for open path
                    if(iterations > 20000){
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
                this.radius = defaultRadius*(1/this.scale);
                let arrowOffset = this.radius*2;

                c.beginPath();
                c.moveTo(this.pathCoords[0][0], this.pathCoords[0][1]);
                for(let i = 1; i < this.pathCoords.length; i++){
                    c.lineTo(this.pathCoords[i][0], this.pathCoords[i][1]);
                }
                if(this.closedPath){
                    c.lineTo(this.pathCoords[0][0], this.pathCoords[0][1]);
                }
                c.lineWidth = 2*(1/this.scale);
                c.strokeStyle = "black";
                c.stroke();

                if(this.velocitySelect && this.newCoords.length > 0){
                    c.beginPath();
                    c.moveTo(this.newCoords[0][0], this.newCoords[0][1]);
                    for(let i = 1; i < this.newCoords.length; i++){
                        c.lineTo(this.newCoords[i][0], this.newCoords[i][1]);
                    }
                    if(this.closedPath){
                        c.lineTo(this.newCoords[0][0], this.newCoords[0][1]);
                    }
                    c.lineWidth = 2*(1/this.scale);
                    c.strokeStyle = "blue";
                    c.stroke();
                }

                c.beginPath();
                c.arc(this.x, this.y, this.radius, 0, 2*Math.PI);
                c.fillStyle = 'rgb(0,0,255)';
                c.fill();
                
                if(this.velocitySelect){
                    c.beginPath();
                    c.arc(this.x, this.y, this.radius*1.5, 0, 2*Math.PI);
                    c.fillStyle = 'rgb(0,255,0)';
                    c.fill();

                    this.tanAngle = Math.atan2(this.pathCoords[this.pathIndex+1][1]- this.pathCoords[this.pathIndex-1][1], this.pathCoords[this.pathIndex+1][0]- this.pathCoords[this.pathIndex-1][0]);

                    this.arrowCentres = [];
                    for(let i = 0; i < 4; i++){
                        this.arrowCentres.push([this.x + this.radius*4*Math.cos(this.tanAngle + i*Math.PI/2), this.y + this.radius*4*Math.sin(this.tanAngle + i*Math.PI/2)]);
                    }

                    this.tanPoints1 = rotateShape(this.tanAngle, [this.x, this.y], [[arrowOffset, this.radius], [this.radius*2+arrowOffset, this.radius], [this.radius*2+arrowOffset, this.radius*2], [this.radius*3+arrowOffset, 0],
                    [this.radius*2+arrowOffset, -this.radius*2], [this.radius*2+arrowOffset, -this.radius], [arrowOffset, -this.radius]]);
                    this.tanPoints2 = rotateShape(this.tanAngle + Math.PI, [this.x, this.y], [[arrowOffset, this.radius], [this.radius*2+arrowOffset, this.radius], [this.radius*2+arrowOffset, this.radius*2], [this.radius*3+arrowOffset, 0],
                    [this.radius*2+arrowOffset, -this.radius*2], [this.radius*2+arrowOffset, -this.radius], [arrowOffset, -this.radius]]);
                    this.rotPoints1 = rotateShape(this.tanAngle + Math.PI/2, [this.x, this.y], [[arrowOffset, this.radius], [this.radius*2+arrowOffset, this.radius], [this.radius*2+arrowOffset, this.radius*2], [this.radius*3+arrowOffset, 0],
                    [this.radius*2+arrowOffset, -this.radius*2], [this.radius*2+arrowOffset, -this.radius], [arrowOffset, -this.radius]]);
                    this.rotPoints2 = rotateShape(this.tanAngle + 3*Math.PI/2, [this.x, this.y], [[arrowOffset, this.radius], [this.radius*2+arrowOffset, this.radius], [this.radius*2+arrowOffset, this.radius*2], [this.radius*3+arrowOffset, 0],
                    [this.radius*2+arrowOffset, -this.radius*2], [this.radius*2+arrowOffset, -this.radius], [arrowOffset, -this.radius]]);

                    c.beginPath();
                    c.moveTo(this.tanPoints1[0][0], this.tanPoints1[0][1]);
                    for(let i = 1; i < this.tanPoints1.length; i++){
                        c.lineTo(this.tanPoints1[i][0], this.tanPoints1[i][1]);
                    }
                    c.fillStyle = 'blue';
                    c.fill();

                    c.beginPath();
                    c.moveTo(this.tanPoints2[0][0], this.tanPoints2[0][1]);
                    for(let i = 1; i < this.tanPoints2.length; i++){
                        c.lineTo(this.tanPoints2[i][0], this.tanPoints2[i][1]);
                    }
                    c.fillStyle = 'blue';
                    c.fill();


                    c.beginPath();
                    c.moveTo(this.rotPoints1[0][0], this.rotPoints1[0][1]);
                    for(let i = 1; i < this.rotPoints1.length; i++){
                        c.lineTo(this.rotPoints1[i][0], this.rotPoints1[i][1]);
                    }
                    c.fillStyle = 'purple';
                    c.fill();

                    c.beginPath();
                    c.moveTo(this.rotPoints2[0][0], this.rotPoints2[0][1]);
                    for(let i = 1; i < this.rotPoints2.length; i++){
                        c.lineTo(this.rotPoints2[i][0], this.rotPoints2[i][1]);
                    }
                    c.fillStyle = 'purple';
                    c.fill();
                }
            }

            this.update = function() {

                this.timeSinceClick += 1; 
                this.draw();
                
                if(!this.velocitySelect){
                    this.x = this.pathCoords[this.pathIndex][0];
                    this.y = this.pathCoords[this.pathIndex][1];
                    this.dx = this.pathVels[this.pathIndex][0];
                    this.dy = this.pathVels[this.pathIndex][1];
                    this.pathIndex += vm.animationSpeed;
                }
                if(this.pathIndex >= this.pathCoords.length){
                    this.pathIndex=0;
                }
                
                if(vm.velocitySelect && Math.pow((Math.pow((mouseX-this.x),2) + Math.pow((mouseY-this.y),2)),1/2) < this.radius*1.5 && this.timeSinceClick > 20){
                    if(this.velocitySelect){
                        this.pathCoords = this.newCoords;
                        this.pathVels = this.newVels;
                        this.newCoords = [];
                        this.newVels = [];
                    }

                    this.timeSinceClick = 0;
                    this.velocitySelect = !this.velocitySelect;
                }
                
                if(this.velocitySelect){
                    // if(ballArray.length = 1){
                    //     ballArray.push(new Ball(canvas.width/3, canvas.height/3, 10, undefined, undefined));
                    //     //console.log(ballArray);
                    // }
                    for(let i = 0; i < this.arrowCentres.length; i++){
                        if(vm.velocitySelect && Math.pow((Math.pow((mouseX-this.arrowCentres[i][0]),2) + Math.pow((mouseY-this.arrowCentres[i][1]),2)),1/2) < this.radius*2 && this.timeSinceClick > 20){
                            this.timeSinceClick = 0;
                            if(i == 0){
                                console.log('tangential out'); 
                                this.newdx += this.buttonStep * Math.cos(this.tanAngle);
                                this.newdy += this.buttonStep * Math.sin(this.tanAngle);
                            }
                            if(i == 1){
                                console.log('radial in');
                                this.newdx += this.buttonStep * Math.sin(this.tanAngle);
                                this.newdy += this.buttonStep * Math.cos(this.tanAngle);
                            }
                            if(i == 2){
                                console.log('tangential in');
                                this.newdx += -this.buttonStep * Math.cos(this.tanAngle);
                                this.newdy += -this.buttonStep * Math.sin(this.tanAngle);
                            }
                            if(i == 3){
                                console.log('radial out');
                                this.newdx += -this.buttonStep * Math.sin(this.tanAngle);
                                this.newdy += -this.buttonStep * Math.cos(this.tanAngle);
                            }
                            console.log('arrow Press');
                            this.path(this.newdx, this.newdy);
                        }
                    }
                }
            }
        }
        
        //let maxRadius = 30;
        //let mouseRadius = 55;
        let ballArray = [];

        // [[x,y, mass]]
        //let massCentres = [[canvas.width/3, canvas.height/3], [2*canvas.width/3, canvas.height/3], [canvas.width/2, 2*canvas.height/3]];
        let massCentres = [[0, 0 , 100]];
        let initialVel = 2;

        ballArray.push(new Ball(canvas.width/3, canvas.height/3, 5, undefined, undefined));
        for(let i = 0; i < ballArray.length; i++){
            ballArray[i].path();
        }

        function animate() {
            requestAnimationFrame(animate);

            for (let i = 0; i < ballArray.length; i++){    
                c.clearRect(-canvas.width*(1/ballArray[i].scale)/2, -canvas.height*(1/ballArray[i].scale)/2, canvas.width*(1/ballArray[i].scale), canvas.height*(1/ballArray[i].scale));
                
                //bounding box
                c.beginPath();
                c.moveTo(-canvas.width*(1/ballArray[i].scale)/2, -canvas.height*(1/ballArray[i].scale)/2);
                c.lineTo(canvas.width*(1/ballArray[i].scale)/2, -canvas.height*(1/ballArray[i].scale)/2);
                c.lineTo(canvas.width*(1/ballArray[i].scale)/2, canvas.height*(1/ballArray[i].scale)/2);
                c.lineTo(-canvas.width*(1/ballArray[i].scale)/2, canvas.height*(1/ballArray[i].scale)/2);
                c.lineTo(-canvas.width*(1/ballArray[i].scale)/2, -canvas.height*(1/ballArray[i].scale)/2);
                c.lineWidth = 1;
                c.stroke();
            }

            mouseX = (vm.mouse.x - rect.left - canvas.width/2)*(1/ballArray[0].scale);
            mouseY = (vm.mouse.y - rect.top - canvas.height/2)*(1/ballArray[0].scale);

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
            // c.arc(mouseX, mouseY, 20, 0, Math.PI * 2);
            // c.fillStyle = "black";
            // c.fill();
            
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