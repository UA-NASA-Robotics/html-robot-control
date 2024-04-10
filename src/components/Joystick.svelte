<script lang="ts">
    export let yAxis: number;
</script>

<p>Button: {yAxis}</p>
<div class="joystick-container">
    <div class="joystick">
        <div class="joystick-thumb" style="transform: translate(-50%, calc({yAxis} * 25px - 50%));"></div>
    </div>
</div>

<style>
    .joystick-container {
        display: flex;
        align-items: center; /* Align items vertically */
        margin-left: 50px; /* Move container to the left */
    }

    .joystick {
        width: 100px;
        height: 100px;
        background-color: #FFD700;
        border-radius: 50%;
        border: 5px solid #ccc;
    }
    
    .joystick-thumb {
        position: relative;
        width: 50px;
        height: 50px;
        background-color: #00008B;
        border-radius: 50%;
        border: 5px solid #ccc;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
    }
</style>
