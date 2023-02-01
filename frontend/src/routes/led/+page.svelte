<script>
    import {connectWebSocket, socket} from '../stores.js'
    import {browser} from '$app/environment';


    if (browser) connectWebSocket();

    let light = "off"
    const switchTheLight = () => {

        socket.send(light);

        //switch the state
        if(light === "off") {
            light = "on";
        } else {
            light ="off";
        }
    }
</script>

<div class="container">
    <h1>Controlling an LED - sending a Value</h1>
    <p>Sending multiple Values with custom tags, so the Arduino knows what type of data it gets.</p>
    <p>For wiring you can check the plan at the <a href="https://create.arduino.cc/projecthub/rowan07/make-a-simple-led-circuit-ce8308">Ardunio ProjectHub</a></p>
    <p class="sidenote">Make sure to upload the correct Code to the Arduino (/hardware/led-example)</p>
    <div class="controls">
        <h3>Controls</h3>
        <button on:click={()=>{switchTheLight()}}>turn {light.toUpperCase()}</button>
    </div>
</div>


<style>

</style>