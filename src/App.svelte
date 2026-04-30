<script>
    import { onMount } from "svelte";
    import Missil from "./components/Missil.svelte";
    import Player from "./components/Player.svelte";
    import Vidas from "./components/Vidas.svelte";
    import GameOver from "./components/GameOver.svelte";
    import Mira from "./components/Mira.svelte";

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
        largura: 400, // gordo
        angulo: 0,
    });
    //#endregion

    const tamanho_mapa = {
        altura: 800,
        largura: 800,
    };

    const maxDeltaX = 3;
    const deltaXAcel = 0.05;
    const deltaXNeutralization = 0.03;

    let score = $state(0);

    const larguraFundo = 960;
    let posFundo = $state((larguraFundo - tamanho_mapa.largura) * 0.5);

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

        if (player.pos < -tamanho_mapa.largura * 2) {
            player.deltaX = 0;
            player.pos = -tamanho_mapa.largura * 2;
        }

        //#region efeito parallax
        // const mapRange = (value, oldMin, oldMax, newMin, newMax) => ((value - oldMin) / (oldMax - oldMin)) * (newMax - newMin) + newMin;
        posFundo =
            ((-player.pos + tamanho_mapa.largura * 2) /
                (tamanho_mapa.largura * 2 + tamanho_mapa.largura * 2)) *
            (larguraFundo - tamanho_mapa.largura);
        //#endregion

        player.deltaX = parseFloat(player.deltaX.toFixed(4));
        player.pos = parseFloat(player.pos.toFixed(4));

        player.pos += player.deltaX;

        //#region entortação do jato
        // const mapRange = (value, oldMin, oldMax, newMin, newMax) => ((value - oldMin) / (oldMax - oldMin)) * (newMax - newMin) + newMin;
        player.angulo = ((-player.deltaX + 1) / (1 + 1)) * (10 + 10) - 10;
        //#endregion

        if (!gameOver) requestAnimationFrame(renderLoop);
    }

    let intervalTimer = setInterval(updateTimer, 1000);

    let milisegundosSpawnPredio = 2000;
    let intervalSpawnPredio = setInterval(spawnPredio, milisegundosSpawnPredio);

    onMount(() => {
        window.addEventListener("keydown", handleKeyDown);
        window.addEventListener("keyup", handleKeyUp);

        renderLoop();
        spawnPredio();

        return () => {
            window.removeEventListener("keydown", handleKeyDown);
            window.removeEventListener("keyup", handleKeyUp);
            clearInterval(intervalSpawnPredio);
            clearInterval(intervalTimer);
        };
    });

    //#region handler de colisão e states dos prédios
    let predios = $state([]);
    let idCounterPredio = 0;

    function checkCollision(id, posicao, largura) {
        if (posicao > tamanho_mapa.largura * 0.5) {
            //ao lado direito da caixa de colisão do player
            const ladoEsquerdoPredio = posicao - largura * 0.5;
            const ladoDireitoPlayer =
                tamanho_mapa.largura * 0.5 + player.largura * 0.5;

            if (ladoDireitoPlayer > ladoEsquerdoPredio) numVidas -= 1;
        } else {
            // ao lado esquerdo da caixa de colisão do player
            const ladoDireitoPredio = posicao + largura * 0.5;
            const ladoEsquerdoPlayer =
                tamanho_mapa.largura * 0.5 - player.largura * 0.5;

            if (ladoDireitoPredio > ladoEsquerdoPlayer) numVidas -= 1;
        }

        if (numVidas == 0) {
            gameOver = true;
            clearInterval(intervalSpawnPredio);
            clearInterval(intervalTimer);
            predios = [];
            return;
        }

        killPredio(id);
    }

    function spawnPredio() {
        predios = [
            {
                id: idCounterPredio++,
            },
            ...predios,
        ];
    }

    function killPredio(id) {
        predios = predios.filter((p) => p.id != id);
    }

    function shootPredio(id) {
        killPredio(id);
        score++;
    }

    let displayTimer = $state("00:00");
    let timerSeconds = 0;
    function updateTimer() {
        timerSeconds++;
        displayTimer = new Date(timerSeconds * 1000)
            .toISOString()
            .substring(14, 19);

        if (timerSeconds % 5 == 0) {
            clearInterval(intervalSpawnPredio);
            if (milisegundosSpawnPredio > 500) milisegundosSpawnPredio -= 100;
            else if (milisegundosSpawnPredio > 100)
                milisegundosSpawnPredio -= 50;
            intervalSpawnPredio = setInterval(
                spawnPredio,
                milisegundosSpawnPredio,
            );
        }
    }
    //#endregion

    let numVidas = $state(3);
    let gameOver = $state(false);
</script>

<div class="pagina">
    <div
        class="mapa"
        style="width: {tamanho_mapa.largura}px; height: {tamanho_mapa.altura}px"
    >
        <img alt="" class="fundo" style="left: -{posFundo}px" />

        <div class="hud">
            {#if !gameOver}
                <Vidas {numVidas} />
            {/if}
            <div class="timer">
                <span style="font-size: 67px;">{displayTimer}</span><span
                    style="font-size: 30px;">score: {score}</span
                >
            </div>
        </div>

        {#each predios as predio (predio.id)}
            <Missil
                id={predio.id}
                posPlayer={player.pos}
                {checkCollision}
                {shootPredio}
            />
        {/each}

        <Player
            largura={player.largura}
            isDead={gameOver}
            angulo={player.angulo}
        />

        {#if gameOver}
            <GameOver />
        {/if}

        <Mira />
    </div>
</div>

<style>
    .mapa {
        background-color: black;
        overflow: hidden;
        display: flex;
        flex-direction: column;
        position: relative;
    }

    .pagina {
        display: flex;
        justify-content: center;
        align-items: center;
        width: 100vw;
        height: 100vh;
    }

    .hud {
        z-index: 3;
        height: 200px;
        display: flex;
        pointer-events: none;
        user-select: none;
    }

    .fundo {
        background-image: url("/img/fundo.png");
        background-size: 960px 800px;
        width: 960px;
        height: 100%;
        position: absolute;
        pointer-events: none;
    }

    .timer {
        margin-left: auto;
        align-content: center;
        padding-right: 10px;
        display: flex;
        flex-direction: column;
        justify-content: center;
    }
</style>
