<script>
	import { browser } from '$app/environment';
	import P5 from 'p5-svelte';

	//Visual variables
	let x;
	let y;
	let angle1 = 180;
	let angle2 = 0.0;
	let segLength = 100;
	let bg;
	let count = 0;
	let reset = true;
	let unterarm;
	let unterarmGrey;
	let bizeps;
	let bizepsGrey;
	let pulse = 120;
	let i = 0;
	let cal = 0;
	let potiValuePrev;

	//Timer variables
	let sec = 0;
	let min = 0;
	let hour = 0;
	let time;
	let prevSec = 0;
	let body;

	//Speed detector
	let counter = 0;
	let tooFast = false;
	let slowdown;
	let set = true;

	const sketch = (p5) => {
		p5.setup = () => {
			bg = p5.loadImage('assets/background_ar.jpg');
			unterarmGrey = p5.loadImage('assets/unterarm_female_grey.png');
			bizepsGrey = p5.loadImage('assets/bizeps_female_grey.png');
			unterarm = p5.loadImage('assets/unterarm_female.png');
			bizeps = p5.loadImage('assets/bizeps_female.png');
			body = p5.loadImage('assets/Body.png');
			slowdown = p5.loadImage('assets/slow-down.png');

			p5.angleMode(p5.DEGREES);
			p5.createCanvas(p5.displayWidth, p5.displayHeight);
			p5.background(bg);
			p5.stroke('#ed225d');
			p5.strokeWeight(30);
			p5.stroke(255, 160);

			x = p5.width / 2;
			y = p5.height / 2;
		};

		p5.draw = () => {
			p5.background(0);

			// initialize timer
			if (set) {
				sec = add0(sec);
				min = add0(min);
				hour = add0(hour);
				set = false;
			}

			// angle calibration
			angle1 = 0;
			angle2 = -potiValue;

			// pulse Display
			if (i % 40 == 0) {
				pulse = Math.floor(120 + Math.random() * 10);
			}

			// cal Display
			if (count > 0) {
				if (i % 1000 == 0 && i != 0) {
					cal++;
				}
			}

			// timer
			if (count > 0) {
				if (prevSec != p5.second()) {
					sec++;
					if (sec % 60 == 0) {
						sec = 0;
						min++;
					}

					sec = add0(sec);
				}

				if (sec % 60 == 0) {
					min = add0(min++);
				}

				prevSec = p5.second();
			}

			time = hour + ':' + min + ':' + sec;

			// add zero in front of numbers < 10
			function add0(i) {
				if (i < 10) {
					i = '0' + i;
				}
				return i;
			}

			// use push and pop to "contain" the transforms. Note that
			// even though we draw the segments using a custom function,
			// the transforms still accumulate
			p5.image(bizepsGrey, 580, 350);

			p5.image(body, 550, 200);

			p5.push()
			p5.tint(255, (potiValue-50)*2.5);
			p5.image(bizeps, 580, 350);
			p5.pop();


			p5.push();
			segment(segLength + 400, 350, angle2, unterarmGrey);
			p5.pop();

			p5.push();
			p5.tint(255, (potiValue-50)*2.5);
			segment(segLength + 400, 350, angle2, unterarm);
			
			p5.pop();

			



			// Display arm
			function segment(x, y, a, ima) {
				p5.translate(x + 107, y + 149); // origin
				p5.rotate(a + 30); // startangle
				p5.imageMode(p5.CORNER_LEFT);
				p5.image(ima, -6, 0, 0, 0); // -6 anchorpoint
			}

			// increment count when height is reached
			if (potiValue > 120 && reset) {
				if (count < 15) count++;
				reset = false;
			}

			// detect when rep is finished (arm is lowered)
			if (potiValue < 25) {
				reset = true;
			}

			p5.fill(255);

			p5.push();
			p5.noStroke();

			//Speed detector
			if (potiValuePrev - potiValue < -10) {
				tooFast = true;
			}

			// Timer for Display
			if (tooFast) {
				counter++;
				if (counter % 150 == 0) {
					tooFast = false;
					counter = 0;
				}

				p5.push();
				p5.imageMode(p5.CENTER);
				p5.image(slowdown, 600, 360, 0, 0);
				p5.pop();

				p5.fill(220, 20, 60);
			} else {
				if (potiValue > 120) {
					p5.fill(50, 205, 50);
				} else {
					p5.fill(255);
				}
			}

			//Bar
			p5.rect(838, 307 - potiValue * 0.8, 11, potiValue * 0.8);
			p5.pop();
			i++;

			//Reset count variable after 1000 iterations
			if (i % 1000 == 0) {
				i = 0;
			}
			potiValuePrev = potiValue;
		};
	};

	let socket;
	let potiValue;

	if (browser) {
		socket = new WebSocket('ws://localhost:8080');

		socket.addEventListener('open', (event) => {
			socket.send('Hello Server!');
		});

		socket.addEventListener('message', (event) => {
			const data = JSON.parse(event.data);
			potiValue = data.message;
			if(potiValue > 150)
			{
				potiValue = 158;
			}

			if(potiValue == undefined)
			{
				potiValue = 0;
			}
		});
	}
</script>

<div class="info">
	<div class="time-elapsed">{time}</div>
	<div class="heartrate-container">
		<div class="heartrate">
			{pulse}
		</div>
		<div class="heart" />
	</div>
	<div class="calories">{cal}</div>
	<div class="cal">kCal</div>
</div>

<div class="optimal-heartrate">/ 140</div>

<div class="exercise">
	<div class="currentRep">{count}</div>
	<div class="of">of</div>
	<div class="maxRep">15</div>
	<div class="bicepCurls">Bicep Curls</div>
	<div class="progressContainer">
		<div class="progressInactive" />
		<div class="progressInactive" />
		<div class="progressInactive" />
		<div class="progressInactive" />
		<div class="progressInactive" />
		<div class="progressInactive" />
		<div class="progressInactive" />
	</div>
	<div class="setContainer">Set 1 of 3</div>
	<div class="setProgressContainer">
		<div class="progressInactive" />
		<div class="progressInactive" />
		<div class="progressInactive" />
	</div>
</div>
<div class="bar-container">
	<div class="bar-rect" />
	<div class="line" />
</div>

<div class="loadingBackground">
	<img src="./assets/armor_logo.png" alt="Loading" class="logo" />
	<div id="loading-content" />
</div>

<P5 {sketch} />

<div class="container">
	<h1>Potentiometer - receiving data</h1>
	<p>
		For wiring you can check the plan at the <a
			href="https://docs.arduino.cc/built-in-examples/basics/AnalogReadSerial"
			>Ardunio Documentation</a
		>
	</p>
	<p class="sidenote">
		Make sure to upload the correct Code to the Arduino (/hardware/poti_photoresistor-example)
	</p>

	<div class="controls">
		<p>potiValue:</p>
		<p class="data-value center">{potiValue}</p>
	</div>
</div>

<style>
	.center {
		align-self: center;
	}

	.info {
		position: absolute;
		width: 268px;
		height: 223px;
		left: 129px;
		top: 116px;

		background: rgba(70, 70, 70, 0.57);
		border-radius: 21.3333px;
	}

	.time-elapsed {
		position: absolute;
		width: 130px;
		height: 48px;
		left: 24px;
		top: 24px;

		font-family: 'Inter';
		font-style: normal;
		font-weight: 400;
		font-size: 40px;
		line-height: 48px;

		color: #ffffff;
	}

	.heartrate-container {
		position: absolute;
		width: 220px;
		height: 46px;
		left: 24px;
		top: 80px;
	}

	.heartrate {
		font-family: 'Inter';
		font-style: normal;
		font-weight: 400;
		font-size: 38px;
		line-height: 46px;
		/* identical to box height */

		color: #ffffff;
	}

	.calories {
		position: absolute;
		width: 61px;
		height: 44px;
		left: 24px;
		top: 135px;

		font-family: 'Inter';
		font-style: normal;
		font-weight: 400;
		font-size: 36px;
		line-height: 44px;
		/* identical to box height */

		color: #ffffff;
	}

	.cal {
		position: absolute;
		width: 34px;
		height: 27px;
		left: 85px;
		top: 134px;

		font-family: 'Inter';
		font-style: normal;
		font-weight: 400;
		font-size: 22px;
		line-height: 27px;
		/* identical to box height */

		color: #d65858;
	}

	/* Animation */
	.heart {
		animation: 3s linear 0s normal none infinite pulse;
	}
	@keyframes pulse {
		0% {
			transform: scale(0.3);
		}
		10% {
			transform: scale(0.4);
		}
		20% {
			transform: scale(0.3);
		}
		40% {
			transform: scale(0.3);
		}
		47% {
			transform: scale(0.4);
		}
		55% {
			transform: scale(0.3);
		}
		62% {
			transform: scale(0.4);
		}
		69% {
			transform: scale(0.3);
		}
		100% {
			transform: scale(0.3);
		}
	}

	.heart {
		position: absolute;
		left: 140px;
		top: -66px;
		width: 80px;
		height: 70px;
		margin: 20% auto 0;
	}
	.heart:before,
	.heart:after {
		position: absolute;
		content: '';
		left: 50px;
		top: 0;
		width: 50px;
		height: 80px;
		background: #d65858;
		border-radius: 50px 50px 0 0;
		transform: rotate(-45deg);
		transform-origin: 0 100%;
	}
	.heart:after {
		left: 0;
		transform: rotate(45deg);
		transform-origin: 100% 100%;
	}

	.exercise {
		position: absolute;
		width: 264px;
		height: 223.11px;
		left: 807px;
		top: 116px;
		background: rgba(70, 70, 70, 0.57);
		border-radius: 21.3333px;
	}

	.currentRep {
		position: absolute;
		width: 81px;
		height: 63px;
		left: 50px;
		top: 49px;

		font-family: 'Inter';
		font-style: normal;
		font-weight: 700;
		font-size: 64px;
		line-height: 77px;
		text-align: right;

		color: #ffffff;
	}

	.of {
		position: absolute;
		width: 39px;
		height: 48px;
		left: 138px;
		top: 68px;

		font-family: 'Inter';
		font-style: normal;
		font-weight: 400;
		font-size: 40px;
		line-height: 48px;

		color: #ffffff;
	}

	.maxRep {
		position: absolute;
		width: 43px;
		height: 48px;
		left: 193px;
		top: 68px;

		font-family: 'Inter';
		font-style: normal;
		font-weight: 400;
		font-size: 40px;
		line-height: 48px;

		color: #ffffff;
	}

	.bicepCurls {
		/* Bicep Curls */

		position: absolute;
		width: 172px;
		height: 39px;
		left: 74px;
		top: 119px;

		font-family: 'Inter';
		font-style: normal;
		font-weight: 400;
		font-size: 32px;
		line-height: 39px;
		/* identical to box height */

		color: #ffffff;
	}

	.progressContainer {
		/* Frame 57 */

		position: absolute;
		width: 206px;
		height: 2px;
		left: 30px;
		top: 26px;
		display: flex;
	}

	.progressActive {
		/* Rectangle 12 */

		position: relative;
		width: 26px;
		height: 2px;
		left: 0px;
		top: 0px;
		margin: 1%;

		background: #d9d9d9;
		border-radius: 1px;
	}

	.progressInactive {
		/* Rectangle 13 */

		position: relative;
		width: 26px;
		height: 2px;
		left: 0px;
		top: 0px;
		margin: 1%;
		background: #636363;
		border-radius: 1px;
	}

	.setContainer {
		position: absolute;
		width: 91px;
		height: 24px;
		left: 118px;
		top: 159px;

		font-family: 'Inter';
		font-style: normal;
		font-weight: 400;
		font-size: 20px;
		line-height: 24px;

		color: #ffffff;
	}

	.setProgressContainer {
		/* setProgressContainer */

		position: absolute;
		width: 86px;
		height: 2px;
		left: 120px;
		top: 189px;
		display: flex;
	}

	#loading-wrapper {
		position: fixed;
		width: 100%;
		height: 100%;
		left: 0;
		top: 0;
	}

	#loading-content {
		display: block;
		position: relative;
		left: 50%;
		top: 50%;
		width: 170px;
		height: 170px;
		margin: -85px 0 0 -85px;
		border: 3px solid #f00;
	}

	#loading-content:after {
		content: '';
		position: absolute;
		border: 3px solid #0f0;
		left: 15px;
		right: 15px;
		top: 15px;
		bottom: 15px;
	}

	#loading-content:before {
		content: '';
		position: absolute;
		border: 3px solid #00f;
		left: 5px;
		right: 5px;
		top: 5px;
		bottom: 5px;
	}

	#loading-content {
		border: 3px solid transparent;
		border-top-color: #e3d032;
		border-bottom-color: #e3d032;
		border-radius: 50%;
		-webkit-animation: loader 2s linear infinite;
		-moz-animation: loader 2s linear infinite;
		-o-animation: loader 2s linear infinite;
		animation: loader 2s linear infinite;
	}

	#loading-content:before {
		border: 3px solid transparent;
		border-top-color: #3dcce3;
		border-bottom-color: #3dcce3;
		border-radius: 50%;
		-webkit-animation: loader 3s linear infinite;
		-moz-animation: loader 2s linear infinite;
		-o-animation: loader 2s linear infinite;
		animation: loader 3s linear infinite;
	}

	#loading-content:after {
		border: 3px solid transparent;
		border-top-color: #074973;
		border-bottom-color: #074973;
		border-radius: 50%;
		-webkit-animation: loader 1.5s linear infinite;
		animation: loader 1.5s linear infinite;
		-moz-animation: loader 2s linear infinite;
		-o-animation: loader 2s linear infinite;
	}

	@-webkit-keyframes loaders {
		0% {
			-webkit-transform: rotate(0deg);
			-ms-transform: rotate(0deg);
			transform: rotate(0deg);
		}
		100% {
			-webkit-transform: rotate(360deg);
			-ms-transform: rotate(360deg);
			transform: rotate(360deg);
		}
	}

	@keyframes loader {
		0% {
			-webkit-transform: rotate(0deg);
			-ms-transform: rotate(0deg);
			transform: rotate(0deg);
		}
		100% {
			-webkit-transform: rotate(360deg);
			-ms-transform: rotate(360deg);
			transform: rotate(360deg);
		}
	}

	.loadingBackground {
		position: absolute;
		width: 100%;
		height: 100%;
		left: 0px;
		top: 0px;
		background: #000000;

		-moz-animation: cssAnimation 0s ease-in 5s forwards;
		/* Firefox */
		-webkit-animation: cssAnimation 0s ease-in 5s forwards;
		/* Safari and Chrome */
		-o-animation: cssAnimation 0s ease-in 5s forwards;
		/* Opera */
		animation: cssAnimation 0s ease-in 5s forwards;
		-webkit-animation-fill-mode: forwards;
		animation-fill-mode: forwards;
	
	}

	.logo {
		/* armor_logo 1 */

		position: absolute;
		width: 111px;
		height: 23px;
		left: 541px;
		top: 407px;
	}

	@keyframes cssAnimation {
		to {
			width: 0;
			height: 0;
			overflow: hidden;
		}
	}
	@-webkit-keyframes cssAnimation {
		to {
			width: 0;
			height: 0;
			visibility: hidden;
		}
	}

	.bar-container {
		/* Frame 67 */

		position: absolute;
		width: 13px;
		height: 127px;
		left: 837px;
		top: 180px;
	}

	.bar-rect {
		position: absolute;
		width: 11px;
		height: 127px;
		left: 1px;
		top: 0px;
		background: rgba(122, 122, 122, 0.3);
	}

	.line {
		position: absolute;
		width: 13px;
		height: 0px;
		left: 0px;
		top: 32px;

		border: 1px solid #ffffff;
	}

	.optimal-heartrate {
		position: absolute;
		width: 95px;
		height: 46px;
		left: 225px;
		top: 196px;

		font-family: 'Inter';
		font-style: normal;
		font-weight: 400;
		font-size: 38px;
		line-height: 46px;
		/* identical to box height */

		color: #ffffff;
	}
</style>