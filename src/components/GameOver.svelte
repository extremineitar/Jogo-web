<script>
</script>

<div class="bin"></div>
<div class="game-end">
    <div class="game-over-card">Game over</div>
</div>

<style>
    .game-end {
        position: absolute;
        width: 100%;
        height: 100%;
        display: flex;
        align-items: center;
        z-index: 3;
        font-size: 80px;
        color: white;
    }

    .bin {
        background-size: 100% 100%;
        opacity: 0.5;
        position: absolute;
        width: 100%;
        height: 100%;
        z-index: 3;
    }

    .game-over-card {
        text-align: center;
        width: 100%;
        background-color: black;
        opacity: 0.5;
    }
</style>
