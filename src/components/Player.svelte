<script>
    let { largura, angulo, isDead } = $props();
</script>

<div class="player jato" style="width: {largura}px; display:{isDead ? 'none' : 'block'}; transform: rotate({angulo}deg);"></div>
<div class="player explosao" style="width: {largura}px; display:{isDead ? 'block' : 'none'}"></div>

<style>
    .player {
        height: 400px;
        margin-top: auto;
        margin-left: auto;
        margin-right: auto;
        z-index: 2;
        background-size: 400px 400px;
        transform-origin: center;
    }

    .explosao {
        background-image: url("../../img/explosao.png");
    }

    .jato {
        background-image: url("../../img/jato.gif");
    }
</style>
