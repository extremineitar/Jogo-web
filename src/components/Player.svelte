<script>
    let { largura } = $props();
</script>

<div class="player" style="width: {largura}px"></div>

<style>
    .player {
        background-color: green;
        height: 10px;
        margin-top: auto;
        margin-left: auto;
        margin-right: auto;
        z-index: 2;
    }
</style>
