<script>
    import { onMount } from "svelte";
    let { id, posPlayer, checkCollision, shootPredio } = $props();

    let escala = $state(1);
    const distancia_inicial = 500;
    let distancia = $state(distancia_inicial);
    let posInicial = $state(0);

    let velocidade = 0.2;

    let dead = $state(false);

    function renderLoop() {
        if (dead) return;

        if (distancia > 0) {
            const t = 1 - distancia / distancia_inicial;
            const aprox = velocidade * (1 + t * 6);

            escala += 0.02 * aprox;
            distancia -= aprox;
        } else {
            let largura = parseFloat((3 * escala).toFixed(2));
            let posicaoFinal = parseFloat((posPlayer + posInicial).toFixed(2));
            checkCollision(id, posicaoFinal, largura);
            return;
        }

        requestAnimationFrame(renderLoop);
    }

    function onClickHandler() {
        dead = true;
        shootPredio(id);
    }

    onMount(() => {
        posInicial = Math.random() * 800 + -posPlayer;
        renderLoop();
    });
</script>

<button
    title=""
    class="retangulo"
    style="
        transform: scale({escala}); 
        top: {Math.abs(distancia_inicial - distancia)}px; 
        left: {posPlayer + posInicial}px;
        "
    onclick={onClickHandler}
></button>

<style>
    .retangulo {
        /* border: solid 1px green; */
        height: 15px;
        width: 3px;
        transform-origin: center;
        position: absolute;
        background-size: 8px 15px;
        border: none;
        background-color: transparent;
        background-image: url('../../img/rocket.png');
        background-repeat: no-repeat;
    }
</style>
