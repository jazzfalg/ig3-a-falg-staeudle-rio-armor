<script>
	import {browser} from '$app/environment';
	import P5 from "p5-svelte";

	let x;
	let y;
	let angle1 = 180;
	let angle2 = 0.0;
	let segLength = 100;
	let bg;
	let count = 0;
	let reset = true;
	let scan = 0;
	let potiValuePrev = 0;
	let unterarm;
	let bizeps;

	const sketch = (p5) => {
		p5.setup = () => {
			
			bg = p5.loadImage('assets/background_ar.jpg');
			unterarm = p5.loadImage('assets/unterarm.png');
			bizeps = p5.loadImage('assets/bizeps.png');
			p5.angleMode(p5.DEGREES);
			p5.createCanvas(p5.displayWidth, p5.displayHeight);
			p5.background(bg);
			p5.stroke('#ed225d');
			p5.strokeWeight(30);
			p5.stroke(255, 160);

			x = p5.width/2;
			y = p5.height/2;
			
		};
  
		p5.draw = () => {
			p5.background(bg);

			

//Change the angle of the segments according to the mouse positions
			angle1 = 0;
			angle2 = -potiValue;

			// if(angle2 > -130)
			// {
			// 	angle2 = -130;
			// }

//use push and pop to "contain" the transforms. Note that
// even though we draw the segments using a custom function,
// the transforms still accumulate
           	p5.image(bizeps,60,-30);

			p5.push();
			segment(segLength, 0, angle2, unterarm);
			p5.pop();


			
			

			function segment(x, y, a, ima) {
			  p5.translate(x, y);
			  p5.rotate(a - 30);
			  p5.imageMode(p5.CORNER);
			  p5.image(ima, 0, 0, 0, 0);


			}

			if(potiValue > 75 && reset)
			{
				count++;
				reset = false;
			}

			if(potiValue < 25)
			{
				reset = true;
			}

			p5.push();
			p5.noStroke();
			p5.text(count, 10, 30);
			p5.pop();
			
			p5.fill(255);

			p5.push();
			p5.noStroke();
  			p5.rect(252, 200 - potiValue * 2, 15, potiValue * 2);
			p5.pop();
  			
		};

		potiValuePrev = potiValue;


	};



	let socket;
	let potiValue;

	if (browser) {
        socket = new WebSocket('ws://172.20.76.153:8080');

        socket.addEventListener('open', (event) => {
            socket.send('Hello Server!');
        });

        socket.addEventListener('message', (event) => {
            const data = JSON.parse(event.data);
            potiValue = data.message;
	    });
    }
    
</script>
<P5 {sketch} />

<div class="container">
	
	<h1>Potentiometer - receiving data</h1>
	<p>For wiring you can check the plan at the <a href="https://docs.arduino.cc/built-in-examples/basics/AnalogReadSerial">Ardunio Documentation</a></p>
	<p class="sidenote">Make sure to upload the correct Code to the Arduino (/hardware/poti_photoresistor-example)</p>

	<div class="controls">
		<p>potiValue:</p>
		<p class="data-value center">{potiValue}</p>
	</div>
</div>


<style>
	.center {
		align-self: center;
	}
</style>