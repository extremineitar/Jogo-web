<script>
    import { onMount } from "svelte";
    import Predio from "./components/Predio.svelte";
    import Player from "./components/Player.svelte";

    /// Para quem tiver a ler, o motivo pelo qual todas as divisões por valores fixos são feitas através de multiplicações (ex: em vez de x / 2 faço x * 0.5, x / 4 faço x * 0.25, etc.), é simplesmente porque é melhor a nível de performance. Parece exagerado fazer isto, mas é preciso lembrar que alguns cálculos são feitos n vezes por segundo, por isso, manter este hábito vai acabar por fazer diferença a longo prazo

    //#region player handlers e state
    let keys = {};

    function handleKeyDown(e) {
        keys[e.key.toLowerCase()] = true;
    }

    function handleKeyUp(e) {
        keys[e.key.toLowerCase()] = false;
    }

    let player = $state({
        pos: 0,
        deltaX: 0, // 1 full direita; -1 full esquerda; 0 centrado;
        largura: 300, // gordo
    });
    //#endregion

    const tamanho_mapa = {
        altura: 800,
        largura: 800,
    };

    const maxDeltaX = 3;
    const deltaXAcel = 0.05;
    const deltaXNeutralization = 0.03;

    function renderLoop() {
        if (keys["a"] || keys["d"]) {
            if (keys["a"] && player.deltaX < maxDeltaX) {
                player.deltaX += deltaXAcel;
            }

            if (keys["d"] && player.deltaX > -maxDeltaX) {
                player.deltaX -= deltaXAcel;
            }
        } else if (player.deltaX != 0)
            player.deltaX +=
                player.deltaX > 0
                    ? -deltaXNeutralization
                    : deltaXNeutralization;

        if (Math.abs(player.deltaX) < 0.01) player.deltaX = 0;
        if (player.pos > tamanho_mapa.largura * 2) {
            player.deltaX = 0;
            player.pos = tamanho_mapa.largura * 2;
        }

        if (player.pos < -tamanho_mapa.largura) {
            player.deltaX = 0;
            player.pos = -tamanho_mapa.largura;
        }

        player.deltaX = parseFloat(player.deltaX.toFixed(4));
        player.pos = parseFloat(player.pos.toFixed(4));

        player.pos += player.deltaX;

        requestAnimationFrame(renderLoop);
    }

    onMount(() => {
        window.addEventListener("keydown", handleKeyDown);
        window.addEventListener("keyup", handleKeyUp);

        renderLoop();
        spawnPredio();
        const interval = setInterval(spawnPredio, 2000);

        return () => {
            window.removeEventListener("keydown", handleKeyDown);
            window.removeEventListener("keyup", handleKeyUp);
            clearInterval(interval);
        };
    });

    //#region handler de colisão e states dos prédios
    let predios = $state([]);
    let idCounterPredio = 0;

    function checkCollision(id, posicao, largura) {
        // odeio matematica chatgpt n vale nada
        if (posicao > tamanho_mapa.largura * 0.5) {
            //ao lado direito da caixa de colisão do player
            const ladoEsquerdoPredio = posicao - largura * 0.5;
            const ladoDireitoPlayer =
                tamanho_mapa.largura * 0.5 + player.largura * 0.5;

            if (ladoDireitoPlayer > ladoEsquerdoPredio) console.log("coalisão");
        } else {
            // ao lado esquerdo da caixa de colisão do player
            const ladoDireitoPredio = posicao + largura * 0.5;
            const ladoEsquerdoPlayer =
                tamanho_mapa.largura * 0.5 - player.largura * 0.5;

            if (ladoDireitoPredio > ladoEsquerdoPlayer) console.log("coalisão");
        }

        killPredio(id);
    }

    function spawnPredio() {
        predios = [
            ...predios,
            {
                id: idCounterPredio++,
            },
        ];
    }

    function killPredio(id) {
        predios = predios.filter((p) => p.id != id);
    }
    //#endregion
</script>

<div class="pagina">
    <div
        class="mapa"
        style="width: {tamanho_mapa.largura}px; height: {tamanho_mapa.altura}px"
    >
        <p style="color: white; margin: 0">{player.deltaX}</p>
        <p style="color: white; margin: 0">{player.pos}</p>

        {#each predios as predio (predio.id)}
            <Predio id={predio.id} posPlayer={player.pos} {checkCollision} />
        {/each}

        <Player largura={player.largura} />
    </div>
</div>

<style>
    .mapa {
        background-color: black;
        overflow: hidden;
        display: flex;
        flex-direction: column;
        position: relative
    }

    .pagina{
        display: flex;
        justify-content: center;
        align-items: center;
        width: 100vw;
        height: 100vh;
    }
</style>
