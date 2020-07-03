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
        // }

        let x = 200;
        let dx = 5;
        let radius = 30;

        function animate() {
            c.clearRect(0,0, innerWidth, innerHeight);
            c.beginPath();
            c.arc(x,200, radius, 0, Math.PI * 2);
            c.strokeStyle = 'blue';
            c.stroke();

            if (x + radius > innerWidth*0.65){
                dx = -dx;
            }
            x += dx;

            requestAnimationFrame(animate);
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