<script>
    import { onMount } from "svelte";
    let { id, posPlayer, checkCollision } = $props();

    let escala = $state(1);
    const distancia_inicial = 400;
    let distancia = $state(distancia_inicial);
    let posInicial = $state(0);

    let velocidade = 0.2;

    function renderLoop() {
        if (distancia > 0) {
            const t = 1 - distancia / distancia_inicial;
            const aprox = velocidade * (1 + t * 6);

            escala += 0.02 * aprox;
            distancia -= aprox;
        } else {
            let largura = parseFloat((30 * escala).toFixed(2));
            let posicaoFinal = parseFloat((posPlayer + posInicial).toFixed(2));
            checkCollision(id, posicaoFinal, largura);
            return;
        }

        requestAnimationFrame(renderLoop);
    }

    onMount(() => {
        posInicial = Math.random() * 800 + -posPlayer;
        renderLoop();
    });
</script>

<div
    class="retangulo"
    style="transform: scale({escala}); top: {Math.abs(
        distancia_inicial - distancia,
    )}px; left: {posPlayer + posInicial}px"
></div>

<style>
    .retangulo {
        background-color: red;
        height: 80px;
        width: 30px;
        opacity: 0.3;
        transform-origin: center;
        position: absolute;
    }
</style>
